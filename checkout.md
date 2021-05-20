---
posttype: "doc"
title: "Checkout"
description: "The checkout service."
icon: "/icons/checkout.svg"
github: "https://github.com/conedevelopment/bazar-docs/blob/master/checkout.md"
order: 4
---

## Generic Overview

The checkout service is no more than a helper class that manages and chains the various steps like updating addresses, creating the order, calculating shipping cost, taxes and discounts.

```php
use Bazar\Modesl\Order;
use Bazar\Support\Facades\Cart;
use Illuminate\Support\Facades\Response;
use Throwable;

class CheckoutController extends Controller
{
    public function process(Request $request)
    {
        Cart::updateItems($request->input('items'));
        Cart::updateBilling($request->input('billing'));
        Cart::updateShipping($request->input('shipping.address'), $request->input('shipping.diver'));

        $order = Cart::checkout($request->input('gateway-driver'));

        // Handle the freshly created order
    }
}
```

## Events

The following events may be triggered when using the checkout service:
- The `Bazar\Events\CheckoutProcessing`: before the checkout processed.
- The `Bazar\Events\CheckoutProcessed`: after the checkout processed.
- The `Bazar\Events\CheckoutFailed`: after the checkout failed.

> When writing a custom gateway driver, make sure, you dispatch the `CheckoutProcessed` event manually.

