## important events should be emitted 
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ExecutorPlugin.sol#L68-L77

the function `executeTransaction()` should emit an event with the returned data . 

## use the functions of `addressProviderServices` directly because the contract inherit it 
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ExecutorPlugin.sol#L73-L74
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ExecutorPlugin.sol#L24
 
the `executorPlugin` contract can use the internal functions directly because it inherits the contract `AddressProviderService` 

## external calls to an empty functions can be removed  
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeModeratorOverridable.sol#L77-L78

the function `validatePostTransactionOverridable` has no implementation so the call can be removed 

## consider adding a logic to remove a subAccount from the main console in the walletRegistry 
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/WalletRegistry.sol#L49-L55

add a function `deregisterWallet()` to remove a subaccount , to allow the main console to remove the bad sub accounts from the wallet 
## there is no need to initialize the variable `call` to the default value 
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/libraries/SafeHelper.sol#L110
```
            uint8 call = uint8(Enum.Operation.Call);
```
because that the `Enum.Operation.Call` is equal to 0 

