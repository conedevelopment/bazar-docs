---
posttype: "doc"
title: "Media"
description: "Media manager, conversions, file uploads."
icon: "/icons/media.svg"
order: 9
---

## Generic Overview

Bazar comes with a very simple yet flexible and powerful media manager component both on back-end and front-end. We can easily upload multiple files in the same time – even bigger ones thanks to the chunked uploads – or search in existing files. Also, we can easily assing any medium to any model that uses the `HasMedia` trait.

You may also configure in the `bazar.php` config file, which disk should be used for storing the uploaded files. By default it's `public`. You may also set the file chunk expiration as well.

> If you decide to stick with the `public` disk, don't forget to run the `php artisan storage:link` command.

> Bazar supports all the configured filesystem drivers.

## The HasMedia trait

This trait does nothing more, but defining a polymorphic relation between the `Medium` and the target model.

```php
use Bazar\Concerns\HasMedia;
use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    use HasMedia;

    //
}
```

After we setup our model, we can easily attach media models to our model, in this case to the `Post` model:

```php
$post = Post::first();

// Attach
$post->media()->attach([1, 2, 3]);

// Retrieve
$medium = $post->media->first();
```

## The Media Manager

The manager is a globally registered Vue component. You can also set two attributes on the component, the `v-model` and the `multiple` attributes:

```html
<button type="button" class="btn btn-primary" @click.prevent="$refs.media.open()">
    Select media
</button>
<media-manager ref="media" v-model="media" multiple></media-manager>
```

> Note, if you omit the `multiple` attribute, the manager allows you to select only one file.

Also, you may want to use the manager as a form field. To do so, just use the `form-media` component which is a wrapper around the manager. However it provides error handling, displays the selected media items and so on.

```html
<data-form action="/posts/1" :model="post">
    <template #default="form">
        <form-input name="title" label="Title" v-model="form.fields.title"></form-input>
    </template>

    <template #aside="form">
        <form-media name="media" multiple v-model="form.fields.media"></form-media>
    </template>
</data-form>
```

## Conversions

You can define conversions to the uploaded **images**. This means, after an image is uploaded, behind the scenes Bazar generates resized and cropped versions that you can easily access using the medium model.

### Managing Conversions

You may register or remove conversions in one of your service provider, or you may create a dedicated `ConversionsProvider` class:

```php
use Bazar\Services\Image;
use Bazar\Support\Facades\Conversion;

public function boot(): void
{
    Conversion::register('custom-resize', function (Image $image) {
        $image->resize(1000, 600);
    });

    Conversion::register('custom-crop', function (Image $image) {
        $image->crop(300, 300);
    });

    Conversion::remove('custom-resize');
}
```

### Using Conversions

You can easily access to the conversions of a medium model by using their URL and path functions:

```php
use Bazar\Models\Medium;

$medium = Medium::find(1);

// Get the URL of the original file or the given conversion
$medium->url();
$medium->url('thumb');

// Get the relative path of the original file or the given conversion
$medium->path();
$medium->path('thumb');

// Get the full path of the original file or the given conversion
$medium->fullPath();
$medium->fullPath('thumb');
```

> Note, in case of using an external driver like `S3`, the `fullPath()` method will return the URL of the file. If the `local` driver is used, the file path will be used.

## Commands

To clear up the expired file chunks, you may call the `php artisan bazar:clear-chunks` command. You may call this command from the scheduler to automatize the cleanup process:

```php
// app/Console/Kernel.php

protected function schedule(Schedule $schedule)
{
    $schedule->command('bazar:clear-chunks')->daily();
}
```
