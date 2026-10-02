# Commerce Skeleton

Online shop built on **Pine Commerce** (`pine/commerce`, Laravel 13 + Blade + Alpine, no Node build).

| Where | What |
|---|---|
| `vendor/pine/commerce` | the platform (never edit - upgrade with composer) |
| `themes/commerce-skeleton` | this shop's storefront theme (views, CSS/JS, theme config) |
| `config/commerce.php`, `config/commerce-import.php` | client overrides (feature switches, importer mappings) |
| `app/Providers/ClientServiceProvider.php` | client code hooks (events, view composers, extension API) |
| `app/Import/` | client adapters for the WordPress importer |
| `.env` | environment (see `.env.example` - every key is documented) |

## Where pine/commerce comes from

`composer.json` → `repositories` decides (written by `commerce:new-client --repo=…` or `--path=…`):

| Mode | composer.json | Use |
|---|---|---|
| **VCS** (production) | `{"type": "vcs", "url": "https://github.com/SetWebUK/ecom-core.git"}` + `"pine/commerce": "^1.2"` | staging + live servers; `composer.lock` pins the exact release |
| **path** (development) | `{"type": "path", "url": "../ecom-core", "options": {"symlink": true}}` + `"pine/commerce": "*@dev"` | working on the platform and this client at the same time – never committed |

To switch modes, edit the two places in `composer.json` by hand (the `repositories` entry and the `pine/commerce`
constraint – `composer config` mangles array-style repositories), then run `composer update pine/commerce` and
commit `composer.json` + `composer.lock`. Commit only the VCS mode.

`SetWebUK/ecom-core` is public: the HTTPS URL needs no credentials (an optional read-only token in
`~/.composer/auth.json` / `COMPOSER_AUTH` lifts GitHub's API rate limit – never commit `auth.json`). With the SSH URL
(`git@github.com:SetWebUK/ecom-core.git`) the server needs a deploy key; if you install from a **private** fork over SSH
only, also add `"preferred-install": {"pine/commerce": "source", "*": "dist"}` to `config` so composer clones over git
instead of downloading a zip through the GitHub API (which needs a token). Steps: `docs/PLAYBOOK.md` part 1.6 in the
pine/commerce repository.

## Everyday commands

    composer install
    php artisan commerce:doctor                 # health check with fixes
    php artisan commerce:publish                # admin assets → public/vendor/commerce/admin (copies)
    php artisan commerce:theme:publish          # theme assets → public/ (copies)
    composer deploy                             # migrate + publish + theme:publish + optimize
    composer test

Updating the platform: `composer update pine/commerce` then `composer deploy` (take a database backup first).

The step-by-step playbook for setting up / migrating a client is `docs/PLAYBOOK.md` in the pine/commerce repository.

## About this repository – the base system

This is the **base system** of Pine Commerce: a plain Laravel 13 application that pulls in the platform,
[`pine/commerce`](https://github.com/SetWebUK/ecom-core), through composer (`composer.json` → `repositories`:
`{"type": "vcs", "url": "https://github.com/SetWebUK/ecom-core.git"}`, `"pine/commerce": "^1.3"`). The shop itself – storefront, back office,
default theme, WordPress importer – lives in `vendor/pine/commerce` and is upgraded with
`composer update pine/commerce`. Everything a client changes lives in this app: `config/commerce*.php`,
`themes/`, `app/Providers/ClientServiceProvider.php`, `app/Import/`.

Try it locally (SQLite, PHP 8.3+):

    git clone https://github.com/SetWebUK/ecom-skeleton.git shop && cd shop
    composer install
    cp .env.example .env && php artisan key:generate
    # .env: DB_CONNECTION=sqlite, DB_DATABASE=/absolute/path/to/database/database.sqlite, APP_URL=http://127.0.0.1:8000,
    #       SESSION_SECURE_COOKIE=false
    touch database/database.sqlite
    php artisan commerce:install --admin-email=you@example.test --store-name="Demo Shop"
    php artisan serve

### Installing pine/commerce: public HTTPS (default) or SSH

- **HTTPS** (`https://github.com/SetWebUK/ecom-core.git`, the default): the repository is public, so no credentials
  are needed and composer downloads release zips (`preferred-install: dist`). Add a read-only GitHub token to
  `~/.composer/auth.json` only if you hit GitHub's anonymous API rate limit.
- **SSH** (`git@github.com:SetWebUK/ecom-core.git`): needs a key GitHub accepts (deploy key or your own). For a
  **private** copy of the core reached over SSH only, also set
  `"config": {"preferred-install": {"pine/commerce": "source", "*": "dist"}}` so composer clones over git instead
  of downloading a zip through the GitHub API (which needs a token for private repositories).

## Starting a new client from this repository

Recommended (names everything for you):

    git clone https://github.com/SetWebUK/ecom-skeleton.git /tmp/commerce-skeleton && cd /tmp/commerce-skeleton
    composer install
    php artisan commerce:new-client /var/www/acme/app --name="Acme Tools" --slug=acme --repo=https://github.com/SetWebUK/ecom-core.git --constraint=^1.3

Or copy it by hand: clone into the new project directory, `rm -rf .git && git init`, then replace
"Commerce Skeleton" / "commerce-skeleton" in `composer.json`, `.env.example` and this README. A client's own
repository is normally private: change `"license"` in `composer.json` and the LICENSE file as you need.

This repository is exported from pine/commerce (`bin/export-client-skeleton.sh` in the ecom-core repository) –
change the skeleton in pine/commerce `stubs/client-skeleton`, not here.
