---
posttype: "doc"
title: "Gateway"
description: "Registering payment gateways, payment and refund handlers."
icon: "/icons/gateway.svg"
order: 7
---

## Generic Overview

Gateways are responsible to handle the payment or the refund process of an order.

## Gateway Drivers

Bazar provides some default drivers out of the box: `Cash`, `Transfer` and `Manual` drivers. They all hold a very simple logic and transaction handling, however in many cases they can be enough.

> Note, `Cash` and `Manual` drivers are makring the transactions completed instantly, while the `Transfer` driver saves it as a pending transaction.

Also, you may add your own driver easily that implements the required logic or pull in a package that provides a gateway.

### Creating Custom Drivers

Registering gateways works almost the same as registering shipping methods. All custom drivers should extend the `Bazar\Gateway\Driver` class, which holds two abstract methods: `pay()` and `refund()`.

> Note, the name is guessed automatically from the classname by default. If the guessed name does not match the desired one, you may specify your custom driver name by using the `name()` method.

Let's create a simple driver as an example:

```php
use Bazar\Gateway\Driver;
use Bazar\Models\Order;
use Bazar\Models\Transaction;

class CreditCardDriver extends Driver
{
    public function pay(Order $order, float $amount = null): Transaction
    {
        //
    }

    public function refund(Order $refund, float $amount = null): Transaction
    {
        //
    }
}
```

Now, let's register the driver using the `Bazar\Support\Facades\Gateway` facade:

```php
use Bazar\Support\Facades\Gateway;

Gateway::extend('credit-card', function ($app) {
    return new CreditCardDriver(
        $app['config']->get('services.creditcard')
    );
});
```
