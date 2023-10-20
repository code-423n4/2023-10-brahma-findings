# Low [1]

## Title
The contract `SafeDeployer.sol` can be stuck in an infinite loop

## Links

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeDeployer.sol#L229-L245

## Summary

The `_createSafe()` function in `SafeDeployer.sol` contract could potentially enter an infinite loop if the `createProxyWithNonce()` function always fails with the `_SAFE_CREATION_FAILURE_REASON` error or if it fails for any reason other than a `Safe` already being deployed with the same `salt`. The first one could happen, for example, if an attacker continuously pre-deploys `Safes` with the same `salt` and incremented `nonces`.
To prevent this, you could add a maximum retry limit to the loop.

## Recommendations
Add a retries variable that is incremented each time when the `createProxyWithNonce()` function fails with the `_SAFE_CREATION_FAILURE_REASON` error. If the number of retries reaches for example, the `maxRetries` limit, the function will revert with the `SafeProxyCreationFailed` error, preventing an infinite loop:

```
uint256 maxRetries = 10;  // Set a maximum retry limit
uint256 retries = 0;

do {
    try IGnosisProxyFactory(gnosisProxyFactory).createProxyWithNonce(gnosisSafeSingleton, _initializer, nonce)
    returns (address _deployedSafe) {
        _safe = _deployedSafe;
    } catch Error(string memory reason) {
        if (keccak256(bytes(reason)) != _SAFE_CREATION_FAILURE_REASON || retries >= maxRetries) {
            // If a safe is already deployed with the same salt, or we've reached the maximum retry limit, revert
            revert SafeProxyCreationFailed();
        }
        emit SafeProxyCreationFailure(gnosisSafeSingleton, nonce, _initializer);
        nonce = _genNonce(ownersHash, _salt);
        retries++;
    } catch {
        revert SafeProxyCreationFailed();
    }
} while (_safe == address(0));
```

# Low [2]

## Title
Missing validation of the `_wallet` address

## Links

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/WalletRegistry.sol#L49-L55

## Summary

The contract `WalletRegistry.sol` does not check if the `_wallet` address in `registerSubAccount()` function is a valid wallet address. This could potentially allow the registration of sub-accounts to non-wallet addresses.

## Recommendations
Add check to ensure that the `_wallet` address is a valid wallet address.

# Low [3]

## Title

Missing validation of the `_subAccount` address

## Links

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/WalletRegistry.sol#L49-L55

## Summary

The contract `WalletRegistry.sol` does not check if the `_subAccount` address in `registerSubAccount()` function is a valid address. This could potentially allow the registration of zero-address as a sub-account.

## Recommendations

Add a check to ensure that the `_subAccount` address is a valid address.

# Low [4]

## Title
Missing validation of the `_subAccount` address

## Links

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/WalletRegistry.sol#L35-L40
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/WalletRegistry.sol#L49-L55


## Summary

The contract `WalletRegistry.sol` does not prevent the `Safe` deployer from registering a `wallet` as a sub-account.
Also, The contract does not prevent a `wallet` from registering itself as its own sub-account. This could potentially lead to confusion and misuse.

# Low [5]

## Title
Missing check for zero address

## Links
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/ExecutorRegistry.sol#L38-L44
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/ExecutorRegistry.sol#L53-L59
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/ExecutorRegistry.sol#L67-L68

## Summary
There are several input arguments, that are not checked if they are zero address.
The contract `ExecutorRegistry.sol` does not check if the `_subAccount` or `_executor` addresses are non-zero addresses. This could lead to potential misuse or errors.
It is a good practice to check if the input addresses are zero addresses. This is because the zero address is often used as a default value in Solidity.

## Recommendations
Add a check to ensure that `_subAccount` and `_executor` addresses are not the zero addresses.

# Low [6]

## Title
Lack of access control and input validation in contract `PolicyRegistry.sol`

## Links
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/PolicyRegistry.sol#L35

## Summary
The contract `PolicyRegistry.sol` does not have any access control for the `updatePolicy()` function and the function is external. This could potentially allow any account to update the policy commit of any other account if they meet the conditions in the `updatePolicy()` function. 
Also, the input parameter `account` address is not checked if it is a valid address. This could potentially allow setting policy commits for non-existent or incorrect addresses. 

## Recommendations
Add access control modifier to the function `updatePolicy()` and validate the input argument `account` address.

# Low [7]

## Title
Lack of access control and input validation in contract `SafeEnabler.sol`

## Links
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeEnabler.sol#L43-L56
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeEnabler.sol#L66-L75

## Summary
In contract `SafeEnabler.sol` the input parameters address `module` and address `guard` in functions `enableModule()` and `setGuard()` are not checked if they are valid contract addresses.
Also, the parameter address `guard` should be checked if it is not the zero address. This could lead to potential misuse or errors.
Also, there is a lack of access control. Anyone can call functions `enableModule()` and `setGuard()`. 

## Recommendations
Add access control modifiers and add check of the input arguments.

# Low [8]

## Title
Missing check for zero address in contract `SafeModerator.sol`

## Links
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeModerator.sol#L33-L63

## Summary
In contract `SafeModerator.sol` the input parameter address `to` in function `checkTransaction()` is not checked if is not a zero address.
It is a good practice to check if the input addresses are zero addresses. This is because the zero address is often used as a default value in Solidity and can lead to loss of funds.

## Recommendations
Add a check in function `checkTransaction()` to ensure that the parameter `to` is not a zaero address.

# Low [9]

## Title
The provided gas is not checked if it is sufficient in `SafeModerator.sol` contract

## Links

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeModerator.sol#L33-L63

## Summary

The contract `SafeModerator.sol` does not check if the gas provided in the input parameter `safeTxGas` is sufficient for the transaction. This could potentially lead to out-of-gas errors.

Actually, in Solidity, it's not possible to directly check if the provided gas is sufficient for a transaction before it's executed because the actual gas cost can only be determined during the execution of the transaction.

However, you can estimate the gas cost of a transaction using the `estimateGas()` function provided by the Ethereum web3 API. This function simulates the execution of the transaction and returns an estimate of the gas that would be used.

Keep in mind that this is only an estimate and the actual gas cost may be higher due to factors like changes in the state of the contract or the Ethereum network between the time the estimate is made and when the transaction is actually executed.

## Recommendations
You could add a function that calls estimateGas and checks if the estimated gas cost is less than or equal to the safeTxGas parameter. If it's not, the function could revert the transaction or return an error.

## Tools Used for the QA report
Manual Review, VS Code

