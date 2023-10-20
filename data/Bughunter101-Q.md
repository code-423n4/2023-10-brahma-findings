## (non-critical) `registerSubAccount()` does not check the subAccount is `registerWallet`

`registerSubAccount()` does not check the subAccount is `registerWallet`, it will cause the console account = subaacount.I suggest check if `isWallet[_subAccount] == true`

## (non-critical) `_executeOnSafe()` did not be call

The SafeHelper._executeOnSafe() did not be call. I'm not sure if this is a bug or if it's something the author missed. If this is useless abandoned code, I recommend deleting it.
