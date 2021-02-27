---
posttype: "doc"
title: "Checkout"
description: "The checkout service."
icon: "/icons/checkout.svg"
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
        // If you wish to update the cart items on the checkout page
        Cart::update($request->input('items'));

        return Cart::checkout()->shipping(
            $request->input('shipping-driver'),
            $request->input('shipping-address')
        )->billing(
            $request->input('billing-address')
        )->gateway(
            $request->input('gateway-driver')
        )->onSuccess(function (Order $order) {
            return Response::redirect()->with([
                'message' => 'Thank you for your order!',
            ]);
        })->onFailure(function (Throwable $e, Order $order) {
            return Response::redirect()->back()->withErrors([
                'checkout' => 'Checkout failed!';
            ]);
        })->process();
    }
}
```

## Events

The following events may be triggered when using the checkout service:
- The `Bazar\Events\CheckoutProcessing`: when the checkout processed and **before** the `onSuccess` callback was called.
- The `Bazar\Events\CheckoutProcessed`: when the checkout failed and **after** the `onSuccess` callback was called.
- The `Bazar\Events\CheckoutFailing`: when the checkout failed and **before** the `onFailure` callback was called.
- The `Bazar\Events\CheckoutFailed`: when the checkout failed and **after** the `onFailure` callback was called.
