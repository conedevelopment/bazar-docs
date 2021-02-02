---
posttype: "doc"
title: "Tax"
description: "Registering taxes, disabling the tax calculation."
icon: "/icons/tax.svg"
order: 11
---

## Generic Overview

Bazar comes with a flexible tax support by default. You can easily manage tax definitions by using the `Bazar\Support\Facades\Tax` facade.

Taxes are stored on the `Item` and the `Shipping` models. Both of them implement the `Taxable` interface, which enforces to use a unified method signature for calculating taxes.

## Registering Taxes

You may register taxes using the `Tax` facade. You can pass a number, a `Closure`, or a class (that implements the `Bazar\Contracts\Tax` interface) along the name of the tax.

```php
use Bazar\Support\Facades\Tax;

// Fix tax
Tax::register('fix-20', 20);
```

```php
// Custom closure tax
use Bazar\Models\Shipping;
use Bazar\Support\Facades\Tax;

Tax::register('custom-percent', function (Taxable $model) {
    return $model->price * ($model instanceof Shipping ? 0.3 : 0.27);
});
```

```php
// Class tax
use Bazar\Contracts\Tax as Contract;
use Bazar\Contracts\Taxable;
use Bazar\Models\Shipping;
use Bazar\Support\Facades\Tax;

class CustomTax implements Contract
{
    public function calculate(Taxable $model): float
    {
        return $model->price * ($model instanceof Shipping ? 0.3 : 0.27);
    }
}

Tax::register('complex-tax', CustomTax::class);
// or
Tax::register('complex-tax', new CustomTax);
```

## Removing Taxes

You may remove registered taxes using the `Tax` facade.

```php
use Bazar\Support\Facades\Tax;

Tax::remove('complex-tax');
```

## Disabling Taxes

You may disable tax calculation globally in some screnarios. To do so, call the `disable` method on the `Tax` facade.

```php
use Bazar\Support\Facades\Tax;

Tax::disable();
```

> Note, when disabling taxes, the previously set taxes won't be updated or recalculated, they stay untouched.
