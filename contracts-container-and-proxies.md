---
posttype: "doc"
title: "Contracts, Container & Proxies"
description: "Swap any container binding easily and bring your own customized classes and logic."
icon: "/icons/containers.svg"
github: "https://github.com/conedevelopment/bazar-docs/blob/master/contracts-container-and-proxies.md"
order: 5
---

## Overriding Container Bindings

Bazar tries to be as flexible as possible. This means it provides an easy way to override default configuration without touching anything else but replacing a container binding.

For example, you might want to use your own product model, which has the `App\Models\Product` namespace. It's not enough to extend the `Bazar\Models\Product` model because the relations and the contract-based bindings – like route model bindings – still use the original model.

To solve this without adding more complexity than necessary, Bazar uses proxies to resolve the currently bound values of the given contract. It means you can easily swap container bindings without touching anything else in the codebase.

```php
namespace App\Models;

use Bazar\Models\Product as BaseProduct;

class Product extends BaseProduct
{
    //
}
```

After you created your customizable model, you can swap the container binding for the product contract. You might do that in one of your service provider's `register` method:

```php
use App\Models\Product;
use Bazar\Contracts\Models\Product as Contract;

public function register()
{
    $this->app->bind(Contract::class, Product::class);
}
```

> Note, you should extend the **base model** then bind it to the **interface**.

## Proxies

Proxies are representing the classes that are currently bound to the container. You usually won't use these a lot, but in some cases – like when you develop a package or extension – proxies can be convenient.

```php
use Bazar\Models\Product;

// Available proxy methods
Product::proxy();
Product::getProxiedClass();
Product::getProxiedContract();

// Proxied calls to the bound instance
Product::proxy()->newQuery()->where(...);
Product::proxy()->variation(...);

// Dynamic usage of bound classes
public function product()
{
    $this->belongsTo(Product::getProxiedClass());
}
```

Bazar provides the `InteractsWithProxy` trait, that makes the desired class proxyable. The trait comes with an abstract methods to be able to resolve the proxies class from the container automatically:

```php
namespace App\Models;

use App\Contracts\Models\Addon as Contract;
use Bazar\Concerns\InteractsWithProxy;
use Illuminate\Databse\Eloquent\Model;

class Addon extends Model implements Contract
{
    use InteractsWithProxy;

    /**
     * Get the proxied contract.
     *
     * @return string
     */
    public static function getProxiedContract(): string
    {
        return Contract::class;
    }
}
```

Also, we need to bind the contract to the container in a service provider:

```php
public function register(): void
{
    $this->app->bind(\App\Contracts\Models\Addon::class, \App\Models\Addon::class);
}
```

From this point, our `Addon` model is proxyable, which means if we swap the binding in the container, the proxied class will be the currently bound value and not the original one. This adds more flexibility when extending our application or using packages.

```php
// Use the Addon proxy

Addon::proxy()->newQuery();
```
