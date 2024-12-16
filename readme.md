<p align="center">
    <img width="400" height="160" src="https://raw.githubusercontent.com/neoighodaro/laravel-early-access/master/art/logo.png" alt="Laravel Early Access logo" />
</p>

<p align="center">This package makes it easy to add early access mode to your existing application. This is useful for when you want to launch a product and need to gather the email addresses of people who want early access to the application.</p>

<p align="center">
    <a href="license.md"><img src="https://poser.pugx.org/neo/laravel-early-access/license?format=flat-square"/></a>
    <a href="https://packagist.org/packages/neo/laravel-early-access"><img src="https://poser.pugx.org/neo/laravel-early-access/v/stable?format=flat-square"/></a>
    <a href="https://packagist.org/packages/neo/laravel-early-access"><img src="https://poser.pugx.org/neo/laravel-early-access/downloads?format=flat-square"></a>
    <a href="https://travis-ci.org/neoighodaro/laravel-early-access"><img src="https://img.shields.io/travis/neoighodaro/laravel-early-access/master.svg?style=flat-square"></a>
    <a href="https://styleci.io/repos/164292196"><img src="https://styleci.io/repos/164292196/shield"></a>
    <a href="https://scrutinizer-ci.com/g/neoighodaro/laravel-early-access/"><img src="https://img.shields.io/scrutinizer/g/neoighodaro/laravel-early-access.svg?style=flat-square"></a>
</a>

<p align="center"><img src="https://user-images.githubusercontent.com/807318/50734150-a442fc80-119a-11e9-9dfa-57904bb001f7.png"></p>

> Take a look at [contributing.md](contributing.md) to see a to do list.

>⚠️ This version supports Laravel 6 and above. Use version 1.x if you require Laravel 5 support.

## Installation

#### Via Composer

To install via composer, run the following command in the root of your Laravel application:

```bash
$ composer require neo/laravel-early-access
```

Register the middleware `Neo\EarlyAccess\Http\Middleware\CheckForEarlyAccessMode` at the bottom of your `web` group
middleware in `app/Http/Middleware/Kernel.php`.

```php
<?php
// [...]

'web' => [
    \App\Http\Middleware\EncryptCookies::class,

    // [...]

    \Neo\EarlyAccess\Http\Middleware\CheckForEarlyAccessMode::class,
],

// [...]
```

Next, add/update the `MAIL_*` keys in your `.env` file. Make sure to include `MAIL_FROM_*` keys as it is required when
sending welcome or goodbye emails to subscribers.

Also, you can optionally add the following environment variables to your `.env` file:

```
EARLY_ACCESS_ENABLED=true
EARLY_ACCESS_URL="/early-access"
EARLY_ACCESS_LOGIN_URL="/login"
EARLY_ACCESS_TWITTER_HANDLE=NeoIghodaro
EARLY_ACCESS_VIEW="early-access::index"
EARLY_ACCESS_SERVICE_DRIVER=database
EARLY_ACCESS_SERVICE_DB_TABLE=subscribers
EARLY_ACCESS_ENABLED_URIS="/login"
EARLY_ACCESS_THEME_COLOR=indigo
EARLY_ACCESS_THEME_TINT_CSS="filter: sepia(1) saturate(10) hue-rotate(-50deg);"
```

Now migrate the required tables:

```shell
$ php artisan migrate
```

And publish the required assets:

```shell
$ php artisan vendor:publish --provider="Neo\EarlyAccess\EarlyAccessServiceProvider"
```

This will make the config, migrations, views, and assets available inside your applications directory so you can customise them.

> **TIP:** You can append the `--tag=assets` flag to publish only the asset files which is required. Other available tag
> values are: `config`, `translations`, `migrations`, `views` and `assets`.

To activate early access, you can do either of the following:

- Run the command `$ php artisan early-access --activate`
- Set the `EARLY_ACCESS_ENABLED` to true in your `.env` file

> **TIP:** Using the artisan command allows you to add IP addresses that are allowed to bypass the early access screen altogether.
>
> `$ php artisan early-access --allow=127.0.0.1 --allow=0.0.0.0`
>
> Note that logged in users will also bypass the early access screen.

## Configuration

```shell
$ php artisan vendor:publish --provider="Neo\EarlyAccess\EarlyAccessServiceProvider" --tag=config
```

#### Configuration options

- `enabled` - Sets whether the mode is enabled or not. In terms of priority, this is the last thing that is checked to
  see if the early access screen should be shown. Login status is checked, then artisan command status is checked, then
  this value is checked. `default: false`

- `url` - The URL the early access screen will be shown at. The client will be redirected to this URL if they do not have
  access and the mode is enabled. You can set the value to `/` or any other existing routes. `default: /early-access`

- `login_url` - The URL to your application's login page. This URL will automatically be bypassed even if early access
  mode is turned on. `default: /login`

- `twitter_handle` - This is used when sending subscription confirmation via email. The user will have the option to tweet
 with the handle you specify tagged.

- `view` - The early access screen view to be loaded. You can publish the views and customise it, or leave the default.
  `default: early-access::index`.

- `service` - This is the subscription driver. See below for how to create your own driver. `default: database`.

- `services.database.table_name` - The database table name. This is useful is you want to change the name of the database
  table. You need to do this before you run the migration though. `default: subscribers`

- `notifications` - The default notification classes. You can use your own notification classes if you would like to
  change how users will be notified when they subscribe or unsubscribe.

## Using `/` or an existing route as the early access URL

To use `/` or an existing route in your application as the early access URL, you need to do the following:

First, register the service provider manually below the `App\Providers\RouteServiceProvider::class` in `config/app.php`.

```php
<?php

return [

    'providers' => [

        // [...]

        App\Providers\RouteServiceProvider::class,
        Neo\EarlyAccess\EarlyAccessServiceProvider::class,

        // [...]

    ],

    // [...]
];
```

Next, open your `composer.json` file and add the package in the `dont-discover` array:

```
// [...]

"laravel": {
    "dont-discover": [
        "neo/laravel-early-access"
    ]
},

// [...]
```

Now run `composer dump-autoload -o` and it should work.

## Creating your own subscription service driver

By default, there is a database driver that manages all the users. You can decide to create your own driver though for other
services like Mailchimp etc. (If you do, please consider submitting a PR with the driver).

To get started, you need to create a new class that implements the service provider class:

```php
<?php

namespace App\Services\SubscriptionServices;

use Neo\EarlyAccess\Contracts\Subscription\SubscriptionProvider;

class MailchimpService implements SubscriptionProvider
{
    public function add(string $email, string $name = null): bool
    {
        // Implement adding a new subscriber...
    }

    public function remove(string $email): bool
    {
        // Implement removing a subscriber...
    }

    public function verify(string $email): bool
    {
        // Implement verifying a subscriber
    }

    /**
     * @return \Neo\EarlyAccess\Subscriber|false
     */
    public function findByEmail(string $email)
    {
        // Implement returning a subscriber from email
    }
}
```

Next, register your service in the `register` method of your `app/Providers/AppServiceProvider` class:

```php
<?php

// [...]

$this->app->bind('early-access.mailchimp', function () {
    return new \App\Services\SubscriptionServices\MailchimpService;
});

// [...]
```

> **NOTE:** Leave the `early-access.` namespace. It is required. Just append the name of your service to the namespace
> as seen above.

Next, go to your published configuration and change the service driver from `database` to `mailchimp`. That's all.

## Change log

Please see the [changelog](changelog.md) for more information on what has changed recently.

## Testing

```bash
$ composer test
```

## Contributing

Please see [contributing.md](contributing.md) for details and a todolist.

## Security

If you discover any security related issues, please email author email instead of using the issue tracker.

## Credits

- [Neo Ighodaro][link-author]
- [Caneco](https://twitter.com/caneco) (for the logo)
- [All Contributors][link-contributors]

## License

Please see the [license file](license.md) for more information.

[link-author]: https://github.com/neoighodaro
[link-contributors]: ../../contributors
