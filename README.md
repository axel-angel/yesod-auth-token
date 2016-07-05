# Installation

As this package is not published in Hackage, you should install it locally.

1. From the root of your project:

```bash
# Clone the repository
git clone https://github.com/axel-angel/yesod-auth-token.git

# Delete the package's own .git folder
rm -rf yesod-auth-token/.git
```

2. In your cabal file, under `build-depends` add `yesod-auth-token`
3. Let `stack` know about the location of this package by adding it in `stack.yml` under:

```yaml
# ...
packages:
- '.'
- 'yesod-auth-token'
```

4. Execute `stack build` to finish the installation


# Example

To put in your `Foundation.hs`

```haskell
import Yesod.Auth.Token

instance YesodAuthToken App where
    type AuthTokenId App = UserId

    setUserToken uid t = do
        runDB $ updateWhere [UserId ==. uid] [UserToken =. t]

    getTokenCreds t = do
        mUser <- runDB . getBy $ UniqueToken t
        return $ (\uid -> TokenCreds (entityKey uid) t) <$> mUser

-- stuff in between

instance YesodAuth App where
    -- stuff
    getAuthId (Creds "token" t _) = do
        mTokenCreds <- getTokenCreds t
        return $ tokenCredsAuthId <$> mTokenCreds

    -- You can add other plugins like BrowserID, email or OAuth here
    authPlugins _ = [authToken]
```

In your model, you can have this

```
User
    token Text
    nickname Text Maybe
      UniqueToken token
      deriving Typeable
```
