# Installation

[[toc]]

## Requirements

- PHP ^8.0
- Laravel 8+
- MySQL 5.7+ / PostgreSQL 9.2+
- exif PHP extension (on most systems it will be installed by default)
- GD PHP extension (used for image manipulation)

## Install GetCandy

### Composer Require Package

```sh
composer require getcandy/getcandy
```

### Publish the Config Files

```sh
php artisan vendor:publish --tag=getcandy
```

## Search Configuration

GetCandy uses Laravel Scout for search. We have had good success using Meilisearch, although it's entirely up to you which driver you use, as long as it's compatible.

If you just want to give the wheels a spin, we also ship with a MySQL driver. Just bear in mind this is highly restrictive and we do not recommend using this in any production capacity.

Publish the Scout config.

```sh
php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"
```

### Going with Meilisearch

::: tip Recommended
Meilisearch is the recommended search driver for GetCandy.
:::

If you're on OSX then you can make use of [Takeout](https://github.com/tighten/takeout) which makes getting Meilisearch set up on Docker a breeze.

Meilisearch also provide great documentation on how to get set up.

[Install Meilisearch](https://docs.meilisearch.com/learn/getting_started/installation.html)

Once you have Meilisearch up and running, simply require the composer packages.

```sh
composer require meilisearch/meilisearch-php http-interop/http-factory-guzzle
```

Add/update the entry in your `.env` file as follows.

```
SCOUT_DRIVER=meilisearch
MEILISEARCH_HOST=http://127.0.0.1:7700
MEILISEARCH_KEY=masterKey
```

See the [Laravel Scout documentation](https://laravel.com/docs/8.x/scout#meilisearch) for more information.

### Going with MySQL

::: warning Development Only
We suggest the MySQL driver is only used for development purposes.
:::

Add/update the entry in your `.env` file as follows.

```
SCOUT_DRIVER=mysql
```

Then finally, add this to your `scout.php` config file.

```php
/*
|--------------------------------------------------------------------------
| MySQL Configuration
|--------------------------------------------------------------------------
*/
'mysql' => [
    'mode' => 'LIKE_EXPANDED',
    'model_directories' => [app_path()],
    'min_search_length' => 0,
    'min_fulltext_search_length' => 4,
    'min_fulltext_search_fallback' => 'LIKE',
    'query_expansion' => false
],
```

## Admin Hub

### Authentication

We use our own authentication guard for the hub, nothing crazy, it just piggy backs off Laravel's own but allows us to use the `staff` table instead of users.

Add this to your `auth.php` config file:

```php
'guards' => [
    // ...
    'staff' => [
        'driver' => 'getcandyhub',
    ],
]
```

### Publish Assets

The admin hub requires some assets to work. Run the following command to publish them to your public directory.

```sh
php artisan getcandy:hub:install
```

## Run Migrations

::: tip
GetCandy uses table prefixes to avoid conflicts with your app's tables. You can change this in the [configuration](/configuration.html).
:::

As you'd expect, there's quite a few tables GetCandy needs to function, so run the migrations now.

```sh
php artisan migrate
```

## Run the Artisan Installer

```sh
php artisan getcandy:install
```

This will take you through a set of questions to configure your GetCandy install. The process includes...

- Creating a default admin user (if required)
- Seeding initial data
- Inviting you to star our repo on GitHub ⭐


::: tip Success 🎉
You are now installed! You can access the admin hub at `http://<yoursite>/hub`
:::

## Spread the Word

If you enjoy our project, please share it with others. The more developers using GetCandy the more we can put back into the project.

Get sharing on Twitter, Reddit, Medium, Dev.to, Laravel News, Slack, Discord, etc.

Go Team GetCandy! 🤟
