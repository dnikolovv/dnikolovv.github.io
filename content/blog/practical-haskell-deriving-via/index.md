---
title: "Using `DerivingVia` to magically derive instances"
date: "2022-09-10T09:52:55.684205549Z"
description: "Reduce boilerplate and add a touch of elegance using DerivingVia."
---

> This short post is part of the [Practical Haskell Bits initiative](/practical-haskell-bits-initiative/). Visit the [repository](https://github.com/dnikolovv/practical-haskell) to find out more real-world examples like this.

Imagine we have defined a typeclass for communicating with some external API.

```haskell
class ExternalAPI m where
  getExternal :: ExternalThingId -> m ExternalThing
```

We define some abstract implementation:

```haskell
getExternalImpl ::
  MonadIO m =>
  ExternalThingId ->
  m ExternalThing
getExternalImpl = ...
```

And add an instance to our application monad:

```haskell
instance ExternalAPI AppM where
  getExternal = getExternalImpl
```

But what if we need an instance of `ExternalAPI` for some other monad as well?

```haskell
instance ExternalAPI AnotherAppM where
  getExternal = getExternalImpl
```

This is not too terrible, but now every time we change the `ExternalAPI` interface, we need to change the instance definitions as well. Also, we need to export and depend on implementation details such as `getExternalImpl`, which is error prone and can quickly get tedious.

Using `DerivingVia`, we can make this a lot more elegant.

Let's define 2 abstract implementations for ExternalAPI: one mocked for testing, and one real one for production

```haskell
-- Mocked implementation
newtype MockedExternalAPI m a = MockedExternalAPI (m a)

instance (MonadIO m) => ExternalAPI (MockedExternalAPI m) where
  getExternal externalId = MockedExternalAPI $ do
    liftIO $ putStrLn $ "Called mocked getExternal with id " <> show externalId <> "..."
    pure ExternalThing
  postExternal _ = MockedExternalAPI $ do
    liftIO $ putStrLn "Called mocked postExternal..."
    pure 1

-- "Real" implementation
newtype RealExternalAPIClient m a = RealExternalAPIClient (m a)

instance (MonadIO m) => ExternalAPI (RealExternalAPIClient m) where
  getExternal externalId = RealExternalAPIClient $
    liftIO $ do
      putStrLn $ "Calling real API with id " <> show externalId <> "..."
      threadDelay $ 1000 * 500 -- Half a second
      pure ExternalThing

  postExternal _ = RealExternalAPIClient $
    liftIO $ do
      putStrLn "Posting to the real API..."
      threadDelay $ 1000 * 500
      pure 123
```

Things have become a tad more abstract.

We have defined two wrappers: `MockedExternalAPI` and `RealExternalAPIClient` and said that if the inner `m` has a `MonadIO` instance, we can give back an `ExternalAPI` implementation.

Now for our application monads we can do

```haskell
-- The real application monad that handles your business logic
newtype AppM a = AppM {runAppM :: IO a}
  ...
  deriving (ExternalAPI) via (RealExternalAPIClient AppM)

-- A monad that you use for testing
newtype TestAppM a = TestAppM {runTestAppM :: IO a}
  ...
  deriving (ExternalAPI) via (MockedExternalAPI TestAppM)
```

This clears up a few things:

1. We don't need to depend on implementation details such as `getExternalImpl`.
2. We have reduced boilerplate.
3. We don't need to change every instance site when we change the `ExternalAPI` interface.


```haskell
demo :: IO ()
demo = do
  runAppM externalAPIAction
  runTestAppM externalAPIAction
  where
    externalAPIAction ::
      Monad m =>
      ExternalAPI m =>
      m ()
    externalAPIAction = do
      void $ getExternal 123
      void $ postExternal ExternalThing
```

You can find the complete example [here](https://github.com/dnikolovv/practical-haskell/tree/main/deriving-via/src).

> This short post is part of the [Practical Haskell Bits initiative](/practical-haskell-bits-initiative/). Visit the [repository](https://github.com/dnikolovv/practical-haskell) to find out more real-world examples like this.