---
posttype: "doc"
title: "Discount"
description: "Registering discounts, disabling the discount calculation."
icon: "/icons/discount.svg"
order: 8
---

## Generic Overview

Bazar comes with a flexible discount support by default. You can easily manage discount definitions by using the `Bazar\Support\Facades\Discount` facade.

Discounts are stored on the `Item` and the `Shipping` models. Both of them implement the `Discountable` interface, which enforces to use a unified method signature for calculating discounts.

## Registering Discounts

You may register discounts using the `Discount` facade. You can pass a number, a `Closure`, or a class (that implements the `Bazar\Contracts\Discount` interface) along the name of the discount.

```php
use Bazar\Support\Facades\Discount;

// Fix discount
Discount::register('fix-20', 20);
```

```php
// Custom closure discount
use Bazar\Support\Facades\Discount;

Discount::register('custom-percent', function (Discountable $model) {
    return $model->total * 0.3;
});
```

```php
// Class discount
use Bazar\Contracts\Discount as Contract;
use Bazar\Contracts\Discountable;
use Bazar\Support\Facades\Discount;

class CustomDiscount implements Contract
{
    public function calculate(Discountable $model): float
    {
        return $model->total * 0.3;
    }
}

Discount::register('complex-discount', CustomDiscount::class);
// or
Discount::register('complex-discount', new CustomDiscount);
```

## Removing Discounts

You may remove registered discounts using the `Discount` facade.

```php
use Bazar\Support\Facades\Discount;

Discount::remove('complex-discount');
```

## Disabling Discounts

You may disable discount calculation globally in some screnarios. To do so, call the `disable` method on the `Discount` facade.

```php
use Bazar\Support\Facades\Discount;

Discount::disable();
```

> Note, when disabling discounts, the previously set discounts won't be updated or recalculated, they stay untouched.
