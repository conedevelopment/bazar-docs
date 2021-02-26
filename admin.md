---
posttype: "doc"
title: "Admin"
description: "Documentation of Admin UI and Authorization."
icon: "/icons/admin.svg"
order: 2
---

## Generic Overview

Bazar provides a simple and extendable admin UI that comes with lots of built-in functionality. The UI is built on `Bootstrap`, `Vue`, and `Inertia`.

## Customizing the UI

After running the `php artisan bazar:install` and the `php artisan bazar:publish` commands.

To publish only the assets (compiled as well), run the `php artisan bazar:publish --tag=bazar-assets` command. This will place all the views, JS, CSS, and other assets that the project uses. At this point, all you need to do is to customize what you want and recompile the assets.

> Note, Bazar automatically publishes the assets registered with the `Asset::script()` and `Asset::style()` methods. These assets are compiled, so you can use them instantly. To customize them, make sure the package publishes the pre-compiled scripts and styles to the application's `resources` directory.

Also, you may use the `--packages` option to update your `packages.json` file with the required `devDependencies` automatically and the `--mix` option to update your `webpack.mix.js` file with the proper compiling tasks.

> By default, the `app.blade.php` uses the `/vendor/bazar/` prefix in the asset URLs.

To ensure Bazar's assets are always up-to-date, you may add a Composer hook inside your project's `composer.json` file to automatically publish the latest assets:

```json
"scripts": {
    "post-update-cmd": [
        "@php artisan bazar:publish"
    ]
}
```

## Translations

Bazar is fully translatable. It uses [Laravel's JSON translations](https://laravel.com/docs/master/localization#using-translation-strings-as-keys) on both back- and front-end. To translate strings, you have to create a new JSON language file in the `resources/lang` directory with the code of the language. For example, the Hungarian translation would be named `hu.json`.

```json
{
    "Dashboard": "Vezérlőpult",
    "Logout": "Kijelentkezés"
}
```

> Note: the front-end will automatically pick up the translated strings and apply them where possible.

## Defining Admins

By default, Bazar looks for an array of emails in the `bazar.admins` config. Of course, you can use a completely different approach to define admin users; it's up to you. But let's take a look at the default approach:

```php
// config/bazar.php

'admins' => [
    'admin@example.com',
    'admin_2@example.com',
]
```

> Note, different roles are not supported in Bazar (to keep things simple no plans to change this for now). However once you decide to bring it in, you can easily integrate and customize the different layers for the given admins by using policies and overriding view templates.

After the admins are defined, all users with their email addresses in the list will have access to the admin UI. This is controlled by the `can:manage-bazar` middleware, which uses the `manage-bazar` gate definition.

If you want to use an admin list based on the `.env` file, you may do the following:

```
// .env
BAZAR_ADMINS=admin@example.com,admin_2@example.com
```

```php
// config/bazar.php

'admins' => explode(',', env('BAZAR_ADMINS', ''))
```

> Please note that the defined admins will receive admin notifications as well; the credentials are not used only for authorization.

## Customizing The Authorization

As we mentioned before, the authorization process can be customized easily. By default, it checks for the authenticated user's email address; however, you can easily change its logic:

```php
use Illuminate\Support\Facades\Gate;

Gate::define('manage-bazar', function ($user) {
    return in_array($user->role, ['editor', 'cashier']);
});
```

> Note, this gate definition is being used to allow/deny global access to the admin UI. To customize the authorization in detail, you may use policies.

### Policies

[Policies](https://laravel.com/docs/master/authorization#creating-policies) provide a centralized place for defining authorization for different actions. By default, Bazar does not offer policies for the models, but you may specify your custom ones. To do so, follow the original documentation and register to the Bazar models.

When a policy is registered to the model, Bazar will automatically apply them in the controller actions.

## Routes

Bazar comes with a bunch of routes out of the box. The routes can be registered easily, using the `Bazar::routes()` method.

```php
use Bazar\Bazar;

Bazar::routes(function ($router) {
    //
});
```

All Bazar routes are web routes. They have the `/bazar` URL and the `bazar.` name prefixes. Also, the `web`, `auth`, `can:manage-bazar` middleware are automatically added to each Bazar routes.

## Validation

You may customize the validation rules that Bazar provides out of the box. To do so, you might listen to Bazar's validation events:

```php
use Bazar\Http\Requests\ProductStoreRequest;
use Illuminate\Support\Facades\Event;

Event::listen('bazar.validating:'.ProductStoreRequest::class, function ($validator) {
    $validator->addRules([
        'slug' => ['required'],
    ]);
});
```
