---
posttype: "doc"
title: "Contracts, Container & Proxies"
description: "Swap any container binding easily and bring your own customized classes and logic."
icon: "/icons/containers.svg"
github: "https://github.com/conedevelopment/bazar-docs/blob/main/contracts-container-and-proxies.md"
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

After you created your custimizable model, you can swap the container binding for the product contract. You might do that in one of your service provider's `register` method:

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
use Bazar\Proxies\Product as ProductProxy;

// Available proxy methods
ProductProxy::getProxiedClass();
ProductProxy::getProxiedInstace();
ProductProxy::getProxiedContract();

// Proxied calls to the bound instance
ProductProxy::query()->where(...);
ProductProxy::getProxiedInstace()->variation(...);

// Dynamic usage of bound classes
public function product()
{
    $this->belongsTo(ProductProxy::getProxiedClass());
}
```

> Note, proxies are facades. You can use the container directly by type-hinting the contract itself, like a traditional route-model-binding in your controller.
