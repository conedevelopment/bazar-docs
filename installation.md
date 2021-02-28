
---
posttype: "doc"
title: "Installation"
description: "Installing and configuring Bazar."
icon: "/icons/installation.svg"
order: 1
---

## Requirements

Before moving on, please check out the Laravel [documentation](https://laravel.com/docs/master/installation) about its installation, requirements, and configuration.

Requirements:
- Laravel `^8.22.1`
- PHP `7.3+`
- PHP GD and EXIF extensions

## Installation

You can install Bazar using composer:

```sh
laravel new app

composer require conedevelopment/bazar
```

Also, make sure the verification routes are included. This is required because the user model implements the `MustVerifyEmail` interface and all Bazar routes use the `verified` middleware.

> Please note: **Bazar does not bring any auth logic or functionality out of the box**. The Laravel UI package provides a perfect solution for that; also, it's an easy task to customize it as you wish, but it's optional.

```sh
composer require laravel/ui
```

```php
use Illuminate\Support\Facades\Auth;

Auth::routes(['verify' => true]);
```

### Running the Install Command

After installing the package, run the `bazar:install` command. It will run the migrations and create a symlink for the compiled assets that you may override later.

> Please note: if you are using Windows, you might open your terminal or development environment as a system administrator.

Also, you may populate your local database with *fake* data. To do so, pass the `--seed` flag to the command.

```sh
php artisan ui bootstrap --auth

php artisan bazar:install --seed
```

### Publishing Assets

You may want to customize the admin UI. To do so, run the `bazar:publish --tag=bazar-assets` command.

```sh
php artisan bazar:publish --tag=bazar-assets

npm install && npm run dev
```

### Preparing Storage

Bazar comes with a media manager out of the box. If you are using the `public` driver, you may link your storage directory to make your media files visible for users.

```sh
php artisan storage:link
```

### Extending the User Model

Bazar comes with a `Bazar\Models\User` model by default. Since users can behave as customers, it provides some functionality by default, like managing order, cart, address relationships, and more. Also, it brings the default authentication services; you may keep clean your `App\Models\User` model and extend the one that comes with Bazar.

```php
namespace App\Models;

use Bazar\Models\User as BazarUser;

class User extends BazarUser
{
    //
}
```

> After extending the application's `User` model, no need to extend the `Illuminate\Foundation\Auth\User` class, also, the `Illuminate\Contracts\Auth\MustVerifyEmail` interface will be automatically implemented by the base model.

### Configuration

After install Bazar, you can find the `config/bazar.php` file, where you can configure the package. We'll talk about the specific configurations in their related documentation sections.
