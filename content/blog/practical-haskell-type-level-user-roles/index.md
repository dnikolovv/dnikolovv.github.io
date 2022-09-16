---
title: "Moving user role constraints to the type level"
date: "2022-09-13T09:52:55.684205549Z"
description: "Why check if a user has a particular role if the compiler can do it for you."
---

> This short post is part of the [Practical Haskell Bits initiative](/practical-haskell-bits-initiative/). Visit the [repository](https://github.com/dnikolovv/practical-haskell) to find out more real-world examples like this.

Implementing user roles is a common scenario. Code like this is not uncommon in any language:

```haskell
adminsOnly :: User -> m ()
adminsOnly user = do
  when (not . isAdmin $ user) $
    error "You must be an admin."
  ...
```

As Haskellers, we often boast about how great Haskell's type system is. Surely we could use it to make this situation better.

A thing that may come to mind is [GADTs](/practical-haskell-ddd-gadt). We could have something like:

```haskell
adminsOnly :: User 'Admin -> m ()
adminsOnly _ = do
  -- We don't need the user, but the simple fact
  -- that we got a `User 'Admin` means we're most likely
  -- in an admin context.
  ...
```

> If you're not familiar with GADTs, check out the [Domain Driven Design using GADTs]((/practical-haskell-ddd-gadt)) article.

The above is a fine approach, but today we'll look at how we can take it further. Also, having to pass in the `User` when we don't really need it is kind of annoying.

We'll look at the following scenario - a web API that has 2 user types and 3 endpoints:

1. An endpoint that can be accessed anonymously
2. An endpoint that can be accessed by `Regular` or `Admin` users
3. An endpoint that can be accessed by `Admin` users only

# Defining the API

The following section uses [`servant`](https://docs.servant.dev/en/stable/cookbook/structuring-apis/StructuringApis.html). If you're not familiar with `servant`, don't worry, you should still be able to follow.

The main thing to know is that`servant` provides a `Handler` monad that your API handlers use by default (you can have a custom one as well, but we'll look at that later).

Here is how our API and server definitions look:

```haskell
-- The "whole" API is a mix of the PublicAPI
-- and the ProtectedAPI that also requires JWT authentication
type API =
  PublicAPI
    :<|> (Auth '[JWT] AnyAPIUser :> ProtectedAPI)

type PublicAPI =
  "public" :> Get '[JSON] Text

anonymousUserHandler :: Handler Text
anonymousUserHandler =
  pure "you hit the anonymous handler!"

type ProtectedAPI =
  RegularAPI :<|> AdminAPI

type RegularAPI =
  "regular" :> Get '[JSON] Text

regularHandler :: Handler Text
regularHandler = do
  pure "you hit the regular handler!"

type AdminAPI =
  "admin" :> Get '[JSON] Text

adminOnlyHandler :: Handler Text
adminOnlyHandler = do
  pure "you hit the admin only handler!"

-- The server implementation
server :: Server API
server =
  publicServer
    :<|> protectedServer
  where
    publicServer :: Handler Text
    publicServer = anonymousUserHandler

    protectedServer :: AuthResult AnyAPIUser -> Server ProtectedAPI
    protectedServer (Authenticated user) = do
      regularHandler
        :<|> adminOnlyHandler
    protectedServer _ =
      -- For non Authenticated users, always throw HTTP 401
      throwAll err401
```

Not great, not terrible, but our admin only and regular user only handlers can still be mistakenly substituted! We want to prevent that.

Ideally, we'd be able to say something like:

```haskell
adminOnlyHandler :: IsAdmin user => Handler Text
adminOnlyHandler = do
  ...
```

That's not really valid Haskell code, but it visualizes the constraint we want to put.


> This short post is part of the [Practical Haskell Bits initiative](/practical-haskell-bits-initiative/). Visit the [repository](https://github.com/dnikolovv/practical-haskell) to find out more real-world examples like this.