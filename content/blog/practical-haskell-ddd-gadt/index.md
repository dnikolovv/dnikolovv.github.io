---
title: "Domain Driven Design using GADTs"
date: "2022-09-12T09:52:55.684205549Z"
description: "One way of making illegal states unrepresentable."
---

> This short post is part of the [Practical Haskell Bits initiative](/practical-haskell-bits-initiative/). Visit the [repository](https://github.com/dnikolovv/practical-haskell) to find out more real-world examples like this.

Let's imagine we're in the context of an online store.

We have an `Order` type that has two possible states - `Outstanding` and `PaidFor`.

```haskell
data OrderStatus = Outstanding | PaidFor

data ShipmentInfo =
  AwaitingShipment |
  Shipped TrackingNumber ShippedAt

data Order = Order
  { id :: OrderId,
    created :: CreatedAt,
    items :: [OrderItem],
    status :: OrderStatus,
    shipmentInfo :: Maybe ShipmentInfo
  }
```

While writing our business logic, we realize that we're often doing error prone validations.

```haskell
refundOrder :: Order -> m ()
refundOrder order = do
  when (status order == PaidFor) $
    error "This order has already been paid for."
  ...

markAsShipped :: Order -> m ()
markAsShipped order = do
  unless (status order == PaidFor) $
    error "Order must be paid for."

  unless (shipmentInfo order == Just AwaitingShipment) $
    error "Order must be awaiting shipment."
```

This would pass the bar in most languages, but it feels like we may be underutilizing Haskell's type system, so we want to refactor. There's multiple approaches that we could take to make this situation better, but today we'll take a look at [GADTs (Generalized Algebraic Data Types)](https://en.wikibooks.org/wiki/Haskell/GADT).

Using GADTs, first we can move the `OrderStatus` to the type level.

```haskell
{-# LANGUAGE DataKinds #-}
{-# LANGUAGE GADTs #-}
{-# LANGUAGE KindSignatures #-}

-- Rename Order to OrderData and remove the status and shipmentInfo
data OrderData = OrderData
  {
    -- status :: OrderStatus,
    -- shipmentInfo :: Maybe ShipmentInfo
  }

-- Define a new `Order` type that's a lot more type safe
data Order (status :: OrderStatus) where
  OutstandingOrder :: OrderData -> Order 'Outstanding
  PaidOrder :: OrderData -> ShipmentInfo -> Order 'PaidFor
```

This allows us to explictly mark the order type we want to work with

```haskell
markAsPaid :: Order 'Outstanding -> m ()
markAsPaid = ...

markAsShipped :: Order 'PaidFor -> m ()
markAsShipped = ...

refundOrder :: Order 'PaidFor -> m ()
refundOrder = ...

-- Or we can just ignore the type if we don't care about it
data SomeOrder = forall status. SomeOrder (Order status)

getAllOrders :: m [SomeOrder]
getAllOrders = ...
```

It also allows us to drastically reduce the validations needed. In fact, the previously "illegal states" are now unrepresentable.

```haskell
refundOrder :: Order 'PaidFor -> m ()
refundOrder order = do
  -- We don't need to validate the status as
  -- it cannot be anything different than `PaidFor`
  ...

markAsShipped :: Order 'PaidFor -> m ()
markAsShipped (PaidOrder orderData shipmentInfo) = do
  -- We've stated in the type signature that
  -- `markAsShipped` works with orders that are PaidFor.
  -- Since `ShipmentInfo` on `PaidOrder` is no longer a `Maybe`,
  -- we don't need to validate anything.
  ...
```

We used this approach in [`aws-lambda-haskell-runtime`](https://github.com/theam/aws-lambda-haskell-runtime). Since Lambda results and errors must have a differently formatted body depending on the proxy (API Gateway, ALB, etc.), we used GADTs to make illegal states unrepresentable.

Find the complete code example [here](https://github.com/dnikolovv/practical-haskell/blob/main/gadt-ddd/src/Lib.hs).


> This short post is part of the [Practical Haskell Bits initiative](/practical-haskell-bits-initiative/). Visit the [repository](https://github.com/dnikolovv/practical-haskell) to find out more real-world examples like this.