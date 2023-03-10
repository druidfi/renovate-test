# Renovate test

Codebase created with PHP 8.1.13 and Composer 2.5.1

`php` constraint for the project allows PHP 8.1 or newer e.g. PHP 8.2 (`^8.1`)

But one package `laminas/laminas-diactoros` has constraint `^7.3 || ~8.0.0 || ~8.1.0` so PHP 8.2 will cause error
with `composer install`

Note: `.github/renovate.json` has shared profiles from https://github.com/druidfi/renovate-config

- E.g. php is ignored - we don't get updates to php require

## Running composer install with PHP 8.2 

``` shell
Installing dependencies from lock file (including require-dev)
Verifying lock file contents can be installed on current platform.
Warning: The lock file is not up to date with the latest changes in composer.json. You may be getting outdated dependencies. It is recommended that you run `composer update` or `composer update <package name>`.
Your lock file does not contain a compatible set of packages. Please run composer update.

  Problem 1
    - laminas/laminas-diactoros is locked to version 2.11.3 and an update of this package was not requested.
    - laminas/laminas-diactoros 2.11.3 requires php ^7.3 || ~8.0.0 || ~8.1.0 -> your php version (8.2.1) does not satisfy that requirement.
```

## Running composer install with PHP 8.0

``` shell
Installing dependencies from lock file (including require-dev)
Verifying lock file contents can be installed on current platform.
Warning: The lock file is not up to date with the latest changes in composer.json. You may be getting outdated dependencies. It is recommended that you run `composer update` or `composer update <package name>`.
Your lock file does not contain a compatible set of packages. Please run composer update.

  Problem 1
    - Root composer.json requires php ^8.1 but your php version (8.0.27) does not satisfy that requirement.
```

## Renovate PR

As `php` constraint is `^8.1`, Renovate will run Composer operations with PHP 8.2 (default, latest available) and
resulting with this error on the PR, see https://github.com/druidfi/renovate-test/pull/2

``` shell
Command failed: docker run --rm --name=renovate_sidecar --label=renovate_child -v "/mnt/renovate/gh/druidfi/renovate-test":"/mnt/renovate/gh/druidfi/renovate-test" -v "/tmp/renovate-cache":"/tmp/renovate-cache" -v "/tmp/containerbase":"/tmp/containerbase" -e COMPOSER_CACHE_DIR -e COMPOSER_AUTH -e BUILDPACK_CACHE_DIR -e CONTAINERBASE_CACHE_DIR -w "/mnt/renovate/gh/druidfi/renovate-test" docker.io/renovate/sidecar bash -l -c "install-tool php 8.2.1 && install-tool composer 2.5.1 && composer update drupal/core-composer-scaffold drupal/core-dev-pinned drupal/core-recommended --with-dependencies --ignore-platform-req='ext-*' --ignore-platform-req='lib-*' --no-ansi --no-interaction --no-scripts --no-autoloader --no-plugins"
```

## Possible solutions

### Setting platform.php value in composer.json

You can use platform configuration to "fake" PHP version and extensions. See https://getcomposer.org/doc/06-config.md#platform

Composer will always think that operations are run with set version.

DANGER: you might end up with code NOT compatible with your actual environment having different PHP version.

### Using fixed value for php in composer.json

Instead of `"php": "^8.1",` you could have `"php": "8.1",` and Renovate would use PHP 8.1.

See https://github.com/renovatebot/renovate/issues/2355#issuecomment-1386555211

CON: `composer validate` would give a general warning:

```
- require.php : exact version constraints (8.1) should be avoided if the package follows semantic versioning
```

### Renovate config and constraints.php

Using `constraints.php` in Renovate config: https://docs.renovatebot.com/configuration-options/#constraints

``` json
{
  "constraints": {
    "php": "8.1.14"
  }
}
```

CON: currently you cannot use minot version e.g. `8.1` but you need to use patch version `8.1.14`.
