# Plugin CI workflows

## Continuous integration workflow

This workflow will execute the following actions as long as they are available on the plugin repository.

| Action | Required package | Required config file |
| -------- | -------- | -------- |
| PHP Parallel Lint | [php-parallel-lint/php-parallel-lint](https://packagist.org/packages/php-parallel-lint/php-parallel-lint) | |
| PHP CodeSniffer | [squizlabs/php_codesniffer](https://packagist.org/packages/squizlabs/php_codesniffer) | `.phpcs.xml` |
| PHP-CS-Fixer | [friendsofphp/php-cs-fixer](https://packagist.org/packages/friendsofphp/php-cs-fixer) | `.php-cs-fixer.php` |
| PHPStan | [phpstan/phpstan](https://packagist.org/packages/phpstan/phpstan) | `phpstan.neon` |
| ESLint | [eslint](https://www.npmjs.com/package/eslint) | `eslint.config.js` or `eslint.config.mjs` or `eslint.config.cjs` or `.eslintrc.js` |
| Stylelint | [stylelint](https://www.npmjs.com/package/stylelint) | `.stylelintrc.js` |
| Licence headers check | [glpi-project/tools](https://packagist.org/packages/glpi-project/tools) | `tools/HEADER` |
| PHPUnit | [phpunit/phpunit](https://packagist.org/packages/phpunit/phpunit) | `phpunit.xml` |
| Jest | [jest](https://www.npmjs.com/package/jest) | `jest.config.js` |

During the `PHPUnit` tests execution, GLPI will be accessible over HTTP (`http://localhost/`).

```yaml
name: "Continuous integration"

on:
  pull_request:

jobs:
  ci:
    uses: "glpi-project/plugin-ci-workflows/.github/workflows/continuous-integration.yml@v1"
    with:
      # The plugin key (system name).
      plugin-key: "myplugin"

      # The version of GLPI on which to run the tests.
      glpi-version: "10.0.x"

      # The version of PHP on which to run the tests.
      php-version: "8.1"

      # The database docker image on which to run the tests.
      db-image: "mariadb:11.4"
```

The available `glpi-version`/`php-version` combinations corresponds to the `ghcr.io/glpi-project/githubactions-glpi-apache` images tags
that can be found [here](https://github.com/orgs/glpi-project/packages/container/githubactions-glpi-apache/versions?filters%5Bversion_type%5D=tagged).

The `db-image` parameter is a combination of the DB server engine (`mysql`, `mariadb` or `percona`) and the server version.
- MariaDB available versions are listed [here](https://github.com/orgs/glpi-project/packages/container/githubactions-mariadb/versions?filters%5Bversion_type%5D=tagged)
- MySQL available versions are listed [here](https://github.com/orgs/glpi-project/packages/container/githubactions-mysql/versions?filters%5Bversion_type%5D=tagged).
- Percona available versions are listed [here](https://github.com/orgs/glpi-project/packages/container/githubactions-percona/versions?filters%5Bversion_type%5D=tagged).

## Generate CI matrix

This workflow can be used to generate a matrix that contains the default PHP/SQL versions that are supported by the target GLPI version.
You can use it in combination with the `Continuous Integration` workflow, as shown in the example below.

```yaml
name: "Continuous integration"

on:
  push:
    branches:
      - "main"
    tags:
       - "*"
  pull_request:
  schedule:
    - cron: "0 0 * * *"
  workflow_dispatch:

concurrency:
  group: "${{ github.workflow }}-${{ github.ref }}"
  cancel-in-progress: true

jobs:
  generate-ci-matrix:
    name: "Generate CI matrix"
    uses: "glpi-project/plugin-ci-workflows/.github/workflows/generate-ci-matrix.yml@v1"
    with:
      glpi-version: "10.0.x"

      # Whether the complete compatibility matrix should be generated.
      # Default: false
      complete-matrix: true
  ci:
    name: "GLPI ${{ matrix.glpi-version }} - php:${{ matrix.php-version }} - ${{ matrix.db-image }}"
    needs: "generate-ci-matrix"
    strategy:
      fail-fast: false
      matrix: ${{ fromJson(needs.generate-ci-matrix.outputs.matrix) }}
    uses: "glpi-project/plugin-ci-workflows/.github/workflows/continuous-integration.yml@v1"
    with:
      plugin-key: "myplugin"
      glpi-version: "${{ matrix.glpi-version }}"
      php-version: "${{ matrix.php-version }}"
      db-image: "${{ matrix.db-image }}"
```
