# Carts

[[toc]]

## Overview

Carts are a collection of products (or other custom purchasable types) that you would like to order.
Carts belong to Users (which relate to Customers).

::: tip
Cart prices are dynamically calculated and are not stored (unlike Orders).
:::

## Carts

```php
GetCandy\Models\Cart
```

|Field|Description|
|:-|:-|
|id|Unique ID for the cart.|
|user_id|Can be `null` for guest users.|
|merged_id|If a cart was merged with another cart, it defines the cart it was merged into.|
|currency_id|Carts can only be for a single currency.|
|channel_id||
|coupon_code|Can be `null`, stores a promotional coupon code, e.g. `SALE20`.|
|created_at||
|updated_at|When an order was created from the basket, via a checkout.|
|meta|JSON data for saving any custom information.|

### Creating a cart

```php
$cart = Cart::create([
    'currency_id' => 1,
    'channel_id' => 2,
]);
```

## Cart Lines

```php
GetCandy\Models\CartLine
```

|Field|Description|
|:-|:-|
|id||
|cart_id||
|purchasable_type|e.g. `GetCandy\Models\ProductVariant`.|
|purchasable_id||
|quantity||
|created_at||
|updated_at||
|meta|JSON data for saving any custom information.|

```php
$cartLine = new CartLine([
    'cart_id' => 1,
    'purchasable_type' => ProductVariant::class,
    'purchasable_id' => 123,
    'quantity' => 2,
    'meta' => [
        'personalization' => 'Love you mum xxx',
    ]
]);

// Or you can use the relationship on the cart.
$cart->lines()->create([/* .. */]);
```

Now you have a basic Cart up and running, it's time to show you how you would use the cart manager to get all the calculated totals and tax.

We've also tried to make Carts extendable as much as possible so, depending on what your stores requirements are, you are free to chop and change things as much as you need to.

## The cart manager

```php
$cart = $cart->getManager()->getCart();
```

This will return a "hydrated" version of your cart will the following:

::: tip
All values will return a `GetCandy\Datatypes\Price` object. So you have access to the following: `value`, `formatted`, `decimal`
:::

```php
$cart->total; // The total price value for the cart
$cart->subTotal; // The cart sub total, excluding tax
$cart->taxAmount; // The monetary value for the amount of tax applied.
$cart->discountTotal; // The monetary value for the discount total.
$cart->taxBreakdown; // This is a collection of all taxes applied across all lines.

foreach ($cart->taxBreakdown as $taxRate) {
    $taxRate->name
    $taxRate->total->value
}
```

Each `CartLine` has access to the same properties as a Cart does.


## Modifying Carts

There may instances where you need to make changes to a cart or cart line, before and/or after calculations have taken place. For this GetCandy usees `Pipelines`. The cart/cart lines are pumped through these pipelines and you are free to make any changes you need either before or after calculation:

```php
<?php

namespace App\Modifiers;

use GetCandy\Base\CartModifier;
use GetCandy\Models\Cart;

class CustomCartModifier extends CartModifier
{
    public function calculating(Cart $cart)
    {
        //
    }

    public function calculated(Cart $cart)
    {
        //
    }
}
```

```php
<?php

namespace App\Modifiers;

use GetCandy\Base\CartLineModifier;
use GetCandy\Models\CartLine;

class CustomCartLineModifier extends CartLineModifier
{
    public function calculating(CartLine $cartLine)
    {
        //
    }

    public function calculated(CartLine $cartLine)
    {
        //
    }
}
```

Then register your modifier in your service provider.

```php
public function boot(
    \GetCandy\Base\CartModifiers $cartModifiers,
    \GetCandy\Base\CartLineModifiers $cartLineModifiers
) {
    $cartModifiers->add(
        CustomCartModifier::class
    );

    $cartLineModifiers->add(
        CustomCartLineModifier::class
    );
}
```

## Calculating Tax

During the cart's lifetime, it's unlikely you will have access to any address information, which can be a pain when you want to accurately display the amount of tax applied to the basket. Moreover, some countries don't even show tax until they reach the checkout. We've tried to make this as easy and extendable as possible for you as the developer to build your store.

When you calculate the cart totals, you will be able to set the billing and/or shipping address on the cart, which will then be used when we calculate which tax breakdowns should be applied.

```php

$shippingAddress = [
    'country_id' => null,
    'title' => null,
    'first_name' => null,
    'last_name' => null,
    'company_name' => null,
    'line_one' => null,
    'line_two' => null,
    'line_three' => null,
    'city' => null,
    'state' => null,
    'postcode' => 'H0H 0H0',
    'delivery_instructions' => null,
    'contact_email' => null,
    'contact_phone' => null,
    'meta' => null,
];

$billingAddress = /** .. */;

$cart->getManager()->setShippingAddress($shippingAddress);
$cart->getManager()->setBillingAddress($billingAddress);
```

You can also pass through a `\GetCandy\Models\Address` model, or even another `\GetCandy\Models\CartAddress`

```php
$shippingAddress = \GetCandy\Models\Address::first();

$cart->getManager()->setShippingAddress($shippingAddress);

$cart->getManager()->setBillingAddress(
    $cart->shippingAddress
);
```

## Cart Session Manager

::: tip
The cart session manager is useful if you're building a traditional Laravel storefront which makes use of sessions.
:::

When building a store, you're going to want an easy way to fetch the cart for the current user (or guest user) by retrieving it from their current session. GetCandy provides an easy to use class to make this easier for you, so you don't have to keep reinventing the wheel.

### Available config

Configuration for your cart is handled in `getcandy/cart.php`

|Field|Description|Default
|:-|:-|:-|
|`session_key`|What key to use when storing the cart id in the session|`gc_cart`
|`auto_create`|If no current basket exists, should we create one in the database?|`false`
|`auth_policy`|When a user logs in, how should we handle merging of the basket?|`merge`
|`eager_load`|Which relationships should be eager loaded by default when calculating the cart totals|


### Getting the cart session instance

You can either use the facade or inject the `CartSession` into your code.

```php
$cart = \GetCandy\Facades\CartSession::current();

public function __construct(
    protected \GetCandy\Base\CartSessionInterface $cartSession
) {
    // ...
}
```

### Fetching the current cart

```php
$cart = \GetCandy\Facades\CartSession::current();
```

When you call current, you have two options, you either return `null` if they don't have a cart, or you want to create one straight away. By default, we do not create them initially as this could lead to a ton of cart models being created for no good reason. If you want to enable this functionality, you can adjust the config in `getcandy/cart.php`

### Forgetting the cart

```php
CartSession::forget();
```

### Using a specific cart
You may want to manually specify which cart should be used for the session.

```php
$cart = \GetCandy\Models\Cart::first();
CartSessionManager::use($cart);
```

The other available methods are as follows:

### Get the current `CartManager` for the cart.
```php
CartSession::manager();
```

### Add a cart line

```php
CartSession::add($purchasable, $quantity);
```

### Update a single line

```php
CartSession::updateLine($cartLineId, $quantity, $meta);
```

### Update multiplate lines
```php
CartSession::updateLines(collect([
    [
        'id' => 1,
        'quantity' => 25,
        'meta' => ['foo' => 'bar'],
    ],
    // ...
]));
```

### Remove a line
```php
CartSession::removeLine($cartLineId);
```

### Associating a cart to a user
You can easily associate a cart to a user.

```php
$cart = \GetCandy\Models\Cart::first();
CartSession::associate($cart, $user, 'merge');
```

### Adding shipping/billing address
As outlined above, you can add shipping / billing addresses to the cart using the following methods:

```php
$cart->getManager()->setShippingAddress([
    'line_one' => null,
    'line_two' => null,
    'line_three' => null,
    'city' => null,
    'state' => null,
    'postcode' => null,
    'country' => null,
]);
$cart->getManager()->setBillingAddress([
    'line_one' => null,
    'line_two' => null,
    'line_three' => null,
    'city' => null,
    'state' => null,
    'postcode' => null,
    'country' => null,
]);
```

You can easily retrieve these addresses by accessing the appropriate property:

```php
$cart->shippingAddress;
$cart->billingAddress;
```

## Handling user login in
When a user logs in, you will likely want to check if they have a cart associated to their account and use that, or if they have started a cart as a guest and logged in, you will likely want to be able to handle this. GetCandy takes the pain out of this by listening to the authentication events and responding automatically by associating any previous guest cart they may have had and, depending on your `auth_policy` merge or override the basket on their account.

## Saved Carts

```php
GetCandy\Models\SavedCart
```

|Field|Description|
|:-|:-|
|id||
|cart_id||
|name|Refereance name for the saved cart.|
|created_at||
|updated_at||

```php
$savedCart = new SavedCart([
    'cart_id' => 4567,
    'name' => 'Shed Project',
]);
```
