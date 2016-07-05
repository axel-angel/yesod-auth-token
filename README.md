# Installation

As this package is not published in Hackage, you should install it locally as a git submodule for your project.

1. In the directory of your project where you put external dependancies (usually `deps` or `lib`):

```bash
# Add the repo as a submodule to your project
git submodule add https://github.com/axel-angel/yesod-auth-token.git 
```

2. In your cabal file, under `build-depends` add `yesod-auth-token`
3. Let `stack` know about the location of this package by adding it in `stack.yml` under:

```yaml
# ...
packages:
- '.'
- 'lib/yesod-auth-token' # Change the dir name if needed
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
