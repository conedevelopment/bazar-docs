---
posttype: "doc"
title: "Cart"
description: "Conceptual overview of the cart facade and the custom cart drivers."
icon: "/icons/cart.svg"
github: "https://github.com/conedevelopment/bazar-docs/blob/master/cart.md"
order: 3
---

## Generic Overview

Bazar comes with a cart service by default, which manages cart models and their functionality.y. The cart service is one of the most crucial concepts in Bazar. It handles products, variations, quantities, taxes, discounts, shipping and prepares the checkout process.

You can access the current cart by using the `Bazar\Support\Facades\Cart` facade.

> When using the `Cart` facade, keep in mind, the **facade** forwards calls to the `Cart` **model**; this provides a more flexible approach.

### The User Model

Cart models can be associated with the `Bazar\Models\User` models by using the `HasOne` relationship. Retrieving and associating the models is the job of the current cart driver. Bazar ships the [`CookieDriver`](#cookie-driver) by default.

### Items

Products can be attached to the cart by using the `Bazar\Models\Item` pivot model. This model holds all the critical information of the relationship between the cart and the product, such as `price`, `quantity`, `tax` and custom `properties`.

One product can be attached to the cart as different items. In fact, if a product will be attached to the cart with `properties` that are already present, the `quantity` will be increased automatically. However, if there is no product linked with the given properties, a new item will be created.

This approach makes it simple to handle quantities, prices, and variations in a cart – even if the same product was attached multiple times but with different `properties`.

```php
$item = Cart::addItem($product, 1, ['Size' => 'L']);

Cart::updateItem($item->id, ['Size' => 'S']);

Cart::removeItem($item->id);
```

### Taxes

> Before moving on, you may check the [tax documentation](/docs/tax) about managing taxes.

The `Cart` model uses the `Itemable` trait, which allows the model to interact with its `Taxable` models. Taxes are stored on the `Item` and `Shipping` models.

There are several methods to retrieve the aggregated tax for the model:

```php
use Bazar\Support\Facades\Cart;

// Aggregate the calculated taxes
$tax = Cart::getModel()->getTax();

// Recalculate the taxes and update the items
$tax = Cart::getModel()->calculateTax();

// Recalculate the taxes without updating the items
$tax = Cart::getModel()->calculateTax(false);
```

### Discounts

> Before moving on, you may check the [discount documentation](/docs/discount) about managing discounts.

Unlike TAXes, discounts are stored directly on the model as an aggregated value.

```php
use Bazar\Support\Facades\Cart;

// Get the discount attribute
$tax = Cart::getModel()->getDiscount();

// Recalculate and update the model
$tax = Cart::getModel()->calculateDiscount();

// Recalculate without updating the model
$tax = Cart::getModel()->calculateDiscount(false);
```

### Shipping

> Before moving on, you may check the [shipping documentation](/docs/shipping) about managing shipping methods.

```php
use Bazar\Support\Facades\Cart;

// Get the calculated shipping cost
Cart::getModel()->shipping->getCost();

// Recalculate the shipping cost
Cart::getModel()->shipping->calculateCost();

// Recalculate the shipping cost without updating the shipping model
Cart::getModel()->shipping->calculateCost(false);
```

### Lock/Unlock Mechanism

Bazar supports multiple currencies by default. Imagine a scenario when your customer starts to put an item in the cart with USD prices, but suddenly, the customer changes the currency. In this case, Bazar recalculates the fees, taxes, discounts and will update the cart's currency.

However, this behavior can be controlled by the lock/unlock mechanism. When the cart is *unlocked*, Bazar will update the items on currency change. But when the cart is *locked*, it will keep the original currency and its values.

> Note, you may retrieve the `Cart` model using the `Cart` facade for this feature.

```php
use Bazar\Models\Cart;

$cart = Cart::first();

// Lock the cart
$cart->lock();

// Unlock the cart
$cart->unlock();

// Get locked carts
Cart::query()->locked();

// Get unlocked carts
Cart::query()->unlocked();
```

This can be extremely useful in the checkout process. Locking the cart can make sure that the values are not changing during the process.

### Expiration

To keep the database clean, carts without owners **expire in 3 days**. To retrieve the expired carts, you may use the `expired()` query scope on the `Cart` model:

```php
use Bazar\Models\Cart;

$expired = Cart::expired()->get();
```

Also, Bazar provides a command to clean all the expired carts easily. You may run the `php artisan bazar:clear-carts` command to delete the database's expired cart records.

> In some cases, you may delete all the existing carts. To do so, pass the `--all` flag to the command.

You may call this command from the scheduler to automatize the cleanup process:

```php
// app/Console/Kernel.php

protected function schedule(Schedule $schedule)
{
    $schedule->command('bazar:clear-carts')->daily();
}
```

## Cart Drivers

Like shipping and payment gateway, Bazar manages multiple cart drivers as well. You can set the default driver easily in the configuration file:

```php
// config/bazar.php

'cart' => [
    'default' => 'token',
    'drivers' => [
        'cookie' => [],
        'token' => [
            // ...
        ],
    ],
]
```

### Cookie Driver

Bazar comes with a cookie driver by default. It stores the current cart's token as a cookie and retrieves the cart model based on the stored token. Because of its straightforward implementation, the cookie driver is not configurable.

### Creating Custom Drivers

Registering cart drivers works almost the same as registering shipping methods or payment gateways. All custom drivers should extend the `Bazar\Cart\Driver` class, which holds one abstract method: `resolve()`.

```php
use Bazar\Cart\Driver;
use Bazar\Models\Cart;

class TokenDriver extends Driver
{
    protected function resolve(Request $request): Cart
    {
        return Cart::query()->firstOrNew([
            //
        ]);
    }

    protected function resolved(Request $request, Cart $cart): void
    {
        parent::resolved($request, $cart);

        // Callback after cart is resolved
    }
}
```

Now, let's register the driver using the `Bazar\Support\Facades\Cart` facade:

```php
use Bazar\Support\Facades\Cart;

Cart::extend('custom', function ($app) {
    return new CustomDriver(
        $app['config']->get('cart.drivers.custom')
    );
});
```
