## [L-01] `_decompileSignatures` can underflow.

### Lines of code
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/PolicyValidator.sol#L144-L167

### Description
`_decompileSignatures` function in `PolicyValidator.sol` contract can underflow due to the condition:

```
if (length < 8) revert InvalidSignatures();
```

`length` of the signature is used when `sigLength`, `expiryEpoch` and `validatorSignature` values are calculated.

If the length of signature will be equal to 8 it can underflow in:

```
validatorSignature = _signatures[length - 8 - sigLength:length - 8];
```
If length = 8 and `sigLength` is equal to 4 it will always underflow due to the fact that `8 - 8 - sigLength` < 0.
### Tools Used
Manual review

### Recommended Mitigation Steps
Consider to change condition to:
```
++  if (length < 12) revert InvalidSignatures();
```
Since length of `validatorSignature` is 4 bytes.

## [NC-01] Error not used.

### Lines of code
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeDeployer.sol#L39

### Description

In `SafeDeployer.sol` contract there is an error `PreComputedAccount(address addr)` and it is not used.

### Tools Used
Manual review

### Recommended Mitigation Steps
Consider to use it or to remove.


## [NC-02] Incorrect return name in natSpec comments.

### Lines of code
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/TransactionValidator.sol#L147

### Description
Due to previous audit report `isGuardBeingRemoved` function in `TransactionValidator.sol` was renamed to `_isConsoleBeingOverriden`, but in `@return` on natSpec comments on L147 it called `isGuardBeingRemoved`.

### Tools Used
Manual review

### Recommended Mitigation Steps
Consider to correct it to `@return _isConsoleBeingOverriden`.


## [NC-03] `registerWallet` function can be called by anyone.

### Lines of code
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/WalletRegistry.sol#L31-L40

### Description
In the natSpec comments says `Can only be called by safe deployer or the wallet itself`, but it any user can call it.

### Recommended Mitigation Steps
Consider to change natSpec comments, since there is no restriction to call it by anyone, `test` files confirm this