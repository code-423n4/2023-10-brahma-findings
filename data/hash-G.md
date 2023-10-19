## Summary

### Gas Optimizations


|ID|Title|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [GAS-01](#gas-01-avoid-caching-variables-if-only-used-once)| Avoid caching variables if only used once | 1 | 20 |
| [GAS-02](#gas-02-use-assembly-to-check-for-non-zero-values)| Use assembly to check for non-zero values | 2 | 18 |
| [GAS-03](#gas-03-use-modifier-instead-of-internal-functions)| Use modifier instead of internal functions | 5 | 20 |


Total: 8 instances over 3 issues with 156 gas saved

## Gas Optimizations

### [GAS-01] Avoid caching variables if only used once
Since the value is used only once, caching the value in stack is unnecessary.
*There are 1 instances of this issue*

```solidity
File: src/core/registries/PolicyRegistry.sol

42:                bytes32 currentCommit = commitments[account];
```

*GitHub*: [42](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L42-#L42)


### [GAS-02] Use assembly to check for non-zero values
Instead of typecasting 0 to address and checking if it is non-zero, an assemlby `gt` operation can be perfromed to check the values. This can save around 18gas.

*There are 2 instances of this issue*

```solidity
File: src/core/registries/WalletRegistry.sol

37:                if (subAccountToWallet[msg.sender] != address(0)) revert IsSubAccount();


51:                if (subAccountToWallet[_subAccount] != address(0)) revert AlreadyRegistered();

```

*GitHub*: [37](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L37-#L37), [51](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L51-#L51)


### [GAS-03] Use modifier instead of internal functions
Modifier's inline the code which saves `jump` instructions associated with internal function calls. Saves around 20gas

*There are 5 instances of this issue*

```solidity
File: src/core/SafeEnabler.sol

81:                 function _onlyDelegateCall() private view {

```

*GitHub*: [81](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L81-#L81)
```solidity
File: src/core/AddressProvider.sol

139:                function _onlyGov() internal view {


147:                function _notNull(address addr) internal pure {

```

*GitHub*: [139](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L139-#L139), [147](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L147-#L147)
```solidity
File: src/core/AddressProviderService.sol

62:               function _onlyGov() internal view {


72:               function _notNull(address _addr) internal pure {

```

*GitHub*: [62](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol#L62-#L62), [72](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol#L72-#L72)