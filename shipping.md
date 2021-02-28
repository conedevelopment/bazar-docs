---
posttype: "doc"
title: "Shipping"
description: "Registering shipping methods, calculating shipping costs."
icon: "/icons/shipping.svg"
order: 10
---

## Generic Overview

Shippings are responsible for calculate the cost of a model that implements the `Bazar\Contracts\Shippable` contract.

### The Shippable Contract

The class that implements the interface must implement the `shipping` method, where the relationship is defined between the model and the `Bazar\Models\Shipping` model:

> By default, the `Bazar\Models\Order` and the `Bazar\Models\Cart` models are implementing the contract.

```php
use Bazar\Models\Shipping;
use Illuminate\Database\Eloquent\Relations\MorphOne;

public function shipping(): MorphOne
{
    return $this->morphOne(Shipping::class, 'shippable')->withDefault();
}
```

### Shipping Address

The `Shipping` model does not hold the address values directly on the model, but in the related `Bazar\Models\Address` model.

## Shipping Drivers

Bazar provides some default drivers out of the box: `Local Pickup` and `Weight Based Shipping` drivers.

### Creating Custom Drivers

Registering shipping methods works almost the same as registering gateways methods. All custom drivers should extend the `Bazar\Shipping\Driver` class, which holds one abstract method: `calculate()`.

> Note, the name is guessed automatically from the class name by default. If the guessed name does not match the desired one, you may specify your custom driver name using the `name()` method.

Let's create a simple driver as an example:

```php
use Bazar\Contracts\Shippable;
use Bazar\Shipping\Driver;

class FedexDriver extends Driver
{
    public function calculate(Shippable $model): float
    {
        //
    }
}
```

Now, let's register the driver using the `Bazar\Support\Facades\Shipping` facade:

```php
use Bazar\Support\Facades\Shipping;

Shipping::extend('fedex', function ($app) {
    return new FedexDriver(
        $app['config']->get('services.fedex')
    );
});
```
