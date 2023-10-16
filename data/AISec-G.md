# 1) Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol?plain=1#L117
```
        if (execRequest.executor.code.length == 0 && execRequest.executorSignature.length == 0) {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol?plain=1#L135
```
        if (trustedValidator.code.length == 0 && validatorSignature.length == 0) {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol?plain=1#L165
```
        if (_from == _to && _value == 0 && _operation == Enum.Operation.Call) {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol?plain=1#L45-L54
```
        if (
            currentCommit == bytes32(0)
                && msg.sender == AddressProviderService._getAuthorizedAddress(_SAFE_DEPLOYER_HASH)
        ) {
            // In case invoker is safe  deployer
        } else if (walletRegistry.isOwner(msg.sender, account)) {
            //In case invoker is updating on behalf of sub account
        } else if (msg.sender == account && walletRegistry.isWallet(account)) {
            // In case invoker is a registered wallet
        }
```

# Recommended Mitigation Steps

```
   if (execRequest.executor.code.length == 0) {
     if (execRequest.executorSignature.length == 0) {
         ...
     }
   }
```

In the same way for other contracts