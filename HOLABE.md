# Holabe fork of Akeneo PIM Community Edition

Akeneo no longer maintains the Community Edition. This fork keeps the version used by
[holabe/pim](https://github.com/holabe/pim) alive with security and dependency updates.

## Branches and versions

| Branch       | Purpose                                                                      |
|--------------|------------------------------------------------------------------------------|
| `holabe/7.0` | Our maintained line. Base: upstream `master` @ `fe8f8c6` (2023-07-03).        |
| `main`       | Untouched mirror of upstream `main`. Do not build on it (see below).          |
| `master`     | Untouched mirror of upstream `master`.                                       |

Releases are tagged on `holabe/7.0` as `v7.1.x` (first release: `v7.1.0`).

- `7.1.x` patch releases: security and dependency updates.
- `7.2.0` and up: bigger changes (e.g. base image / PHP upgrades). `holabe/pim` must opt in on purpose.
- `holabe/pim` requires `akeneo/pim-community-dev: ~7.1.0` (any `7.1.x`, never `7.2`).

Do not use other version schemes. Upstream `v7.0.x` tags and upstream's calendar tags (`v2026.x`,
used on upstream `main`) also exist in this repository. A calendar version on our side could match
an upstream `main` tag, and Composer would silently install upstream `main`.

## Gotchas

- `composer install` / `update` in `holabe/pim` runs `std-build/install-required-files.sh`. Up to
  `v7.1.0` it copied this repository's `.env` over the project's `.env` (losing `REDIS_URL` etc.).
  Since `v7.1.1` it only creates `.env` when the project has none.
- A fresh core in `vendor/` loses the Webkul icon links. Run `docker/install-plugin-icons.sh` inside
  the node container (the links are absolute paths), then `make front-packages`.

## Do not merge upstream `main`

Upstream `main` is the 7.0 maintenance line. It split from `master` at `v7.0.20` (2023-06-29)
and does **not** contain about 2,900 later `master` commits, including:

- enriched categories: category templates and category attributes
  (`holabe/pim` `CategoryMapping` depends on them)
- most of the identifier generator
- newer catalogs and connectivity work

Merging or rebasing onto `main` would remove these features. If we need something from `main`
(for example the PHP 8.3 / MySQL 8.0.34 / Elasticsearch 8.17 upgrades), cherry-pick it onto
`holabe/7.0` and test it.

## Changes compared to upstream `fe8f8c6`

- Dependency pins relaxed for security fixes: `aws/aws-sdk-php` `^3.371.4`, `dompdf/dompdf` `^3.1.6`
  (SH-3009).
- `composer.lock` and `yarn.lock` refreshed to close the OSV.dev advisories (SH-3001, SH-3009).
- Removed unused devDependencies `dependency-cruiser` (only source of `handlebars`), `babel-minify`
  and `webpack-cleanup-plugin`. `loader-utils` `^1.4.2`, `underscore` `^1.13.8`.
- `resolutions` force patched versions inside old dev tooling (react-scripts 4, cypress 6).
  `holabe/pim` has the same `resolutions` block in its root `package.json`, because yarn only reads
  resolutions from the workspace root. Keep both lists in sync.
- `v7.1.1` (SH-3010): removed lockfiles that were never installed but showed up in the nightly
  scan: `src/Oro/Bundle/ConfigBundle/front/yarn.lock` (that folder is a workspace of the root, yarn
  ignores its own lock) and the design-system demo app `front-packages/akeneo-design-system/example`
  (plus its steps in `.github/workflows/dsm-test.yml`). `install-required-files.sh` no longer
  overwrites an existing `.env`.

## Known leftovers

The remaining advisories are in dev-only tooling that is not part of the production build:
`react-scripts` 4 (only used by the `app:start` standalone dev servers) and `cypress` 6 (upstream
CircleCI e2e). Fixing them needs a react-scripts / cypress upgrade or removal, tracked separately.
