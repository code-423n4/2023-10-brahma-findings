## Summary
[I-1] Use named parameters in custom errors.

### [I-1] Use named parameters in custom errors.
Using named parameters in custom errors improves code readability. In some errors the parameters are named, but there are some that aren't.

```solidity
error NotGovernance(address);
error NotPendingGovernance(address);
```

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/AddressProvider.sol#L17-L18

```solidity
error NotGovernance(address);
```

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/AddressProviderService.sol#L20

```solidity
error NotGovernance(address);
```
