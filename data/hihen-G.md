# Gas Report

## Summary

Total **27 instances** over **7 issues**with **1013** gas saved:

|                                              ID                                               | Issue                                                                     | Instances | Gas |
| :-------------------------------------------------------------------------------------------: | :------------------------------------------------------------------------ | :-------: | :-: |
|                [[G&#x2011;01]](#g01-use-unchecked-block-for-safe-subtractions)                | Use `unchecked` block for safe subtractions                               |     3     | 255 |
|          [[G&#x2011;02]](#g02-do-not-cache-state-variables-that-are-used-only-once)           | Do not cache state variables that are used only once                      |     1     |  3  |
| [[G&#x2011;03]](#g03-functions-that-revert-when-called-by-normal-users-can-be-marked-payable) | Functions that revert when called by normal users can be marked `payable` |     3     | 63  |
|        [[G&#x2011;04]](#g04-usage-of-intsuints-smaller-than-32-bytes-incurs-overhead)         | Usage of `int`s/`uint`s smaller than 32 bytes incurs overhead             |     8     | 440 |
|   [[G&#x2011;05]](#g05-unused-internal-functions-should-be-removed-to-save-deployment-gas)    | Unused `internal` functions should be removed to save deployment gas      |     1     |  -  |
|  [[G&#x2011;06]](#g06-multiple-accesses-of-the-same-mappingarray-keyindex-should-be-cached)   | Multiple accesses of the same mapping/array key/index should be cached    |     6     | 252 |
|               [[G&#x2011;07]](#g07-consider-using-bytes32-rather-than-a-string)               | Consider using `bytes32` rather than a `string`                           |     5     |  -  |

## Gas Optimizations

### [G&#x2011;01] Use `unchecked` block for safe subtractions

If it can be confirmed that the subtraction operation will not overflow, using an unchecked block can save gas.
For example, `require(x <= y); z = y - x;` can be optimized to `require(x <= y); unchecked { z = y - x; }`.

There are 3 instances:

- _PolicyValidator.sol_ ( [#L164](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/PolicyValidator.sol#L164), [#L166](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/PolicyValidator.sol#L166), [#L166](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/PolicyValidator.sol#L166) ):

```solidity
/// @audit checked on line 162
164:         uint32 sigLength = uint32(bytes4(_signatures[length - 8:length - 4]));

/// @audit checked on line 162
166:         validatorSignature = _signatures[length - 8 - sigLength:length - 8];

/// @audit checked on line 162
166:         validatorSignature = _signatures[length - 8 - sigLength:length - 8];
```

### [G&#x2011;02] Do not cache state variables that are used only once

It's cheaper to access the state variable directly if it is accessed only once. This can save the 3 gas cost of the extra stack allocation.

There is 1 instance:

- _PolicyRegistry.sol_ ( [#L42](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/PolicyRegistry.sol#L42) ):

```solidity
42:         bytes32 currentCommit = commitments[account];
```

### [G&#x2011;03] Functions that revert when called by normal users can be marked `payable`

If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.
The extra opcodes avoided are:
`CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2)`
which cost an average of about 21 gas per call to the function, in addition to the extra deployment cost.

There are 3 instances:

- _AddressProvider.sol_ ( [#L52](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/AddressProvider.sol#L52), [#L77](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/AddressProvider.sol#L77), [#L97](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/AddressProvider.sol#L97) ):

```solidity
52:     function setGovernance(address _newGovernance) external {

77:     function setAuthorizedAddress(bytes32 _key, address _authorizedAddress, bool _overrideCheck) external {

97:     function setRegistry(bytes32 _key, address _registry) external {
```

### [G&#x2011;04] Usage of `int`s/`uint`s smaller than 32 bytes incurs overhead

Using `ints`/`uints` smaller than 32 bytes may cost more gas. This is because the EVM operates on 32 bytes at a time, so if an element is smaller than 32 bytes, the EVM must perform more operations to reduce the size of the element from 32 bytes to the desired size.

There are 8 instances:

- _PolicyValidator.sol_ ( [#L22](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/PolicyValidator.sol#L22), [#L113](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/PolicyValidator.sol#L113), [#L159](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/PolicyValidator.sol#L159), [#L164](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/PolicyValidator.sol#L164) ):

```solidity
/// @audit uint32
22:     error TxnExpired(uint32 expiryEpoch);

/// @audit uint32
113:         (uint32 expiryEpoch, bytes memory validatorSignature) = _decompileSignatures(signatures);

/// @audit uint32
159:         returns (uint32 expiryEpoch, bytes memory validatorSignature)

/// @audit uint32
164:         uint32 sigLength = uint32(bytes4(_signatures[length - 8:length - 4]));
```

- _SafeModeratorOverridable.sol_ ( [#L21](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeModeratorOverridable.sol#L21) ):

```solidity
/// @audit uint8
21:     uint8 public constant DIFFER_SAFE_MOD = 0;
```

- _SafeHelper.sol_ ( [#L110](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/libraries/SafeHelper.sol#L110) ):

```solidity
/// @audit uint8
110:             uint8 call = uint8(Enum.Operation.Call);
```

- _TypeHashHelper.sol_ ( [#L24](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/libraries/TypeHashHelper.sol#L24), [#L40](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/libraries/TypeHashHelper.sol#L40) ):

```solidity
/// @audit uint8
24:         uint8 operation;

/// @audit uint32
40:         uint32 expiryEpoch;
```

### [G&#x2011;05] Unused `internal` functions should be removed to save deployment gas

All `internal` functions that are never used can be safely removed or commented out to save gas. If the functions are required by an interface, the contract should inherit from that interface and use the override keyword.

There is 1 instance:

- _AddressProviderService.sol_ ( [#L62](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/AddressProviderService.sol#L62) ):

```solidity
62:     function _onlyGov() internal view {
```

### [G&#x2011;06] Multiple accesses of the same mapping/array key/index should be cached

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local `storage` or `calldata` variable when the value is accessed [multiple times](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0), saves **~42 gas per access** due to not having to recalculate the key's keccak256 hash (Gkeccak256 - **30 gas**) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into memory/calldata

There are 6 instances:

- _AddressProvider.sol_ ( [#L101](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/AddressProvider.sol#L101) ):

```solidity
/// @audit `registries[_key]` is also accessed on line 102
101:         if (registries[_key] != address(0)) revert RegistryAlreadyExists();
```

- _SafeEnabler.sol_ ( [#L53](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeEnabler.sol#L53), [#L53](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeEnabler.sol#L53) ):

```solidity
/// @audit `modules[module]` is also accessed on line 52
53:         modules[module] = modules[_SENTINEL_MODULES];

/// @audit `modules[_SENTINEL_MODULES]` is also accessed on line 54
53:         modules[module] = modules[_SENTINEL_MODULES];
```

- _PolicyRegistry.sol_ ( [#L67](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/PolicyRegistry.sol#L67) ):

```solidity
/// @audit `commitments[account]` is also accessed on line 68
67:         emit UpdatedPolicyCommit(account, policyCommit, commitments[account]);
```

- _WalletRegistry.sol_ ( [#L36](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/WalletRegistry.sol#L36), [#L51](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/WalletRegistry.sol#L51) ):

```solidity
/// @audit `isWallet[msg.sender]` is also accessed on line 38
36:         if (isWallet[msg.sender]) revert AlreadyRegistered();

/// @audit `subAccountToWallet[_subAccount]` is also accessed on line 52
51:         if (subAccountToWallet[_subAccount] != address(0)) revert AlreadyRegistered();
```

### [G&#x2011;07] Consider using `bytes32` rather than a `string`

Using the `bytes` types for fixed-length strings is more efficient than having the EVM have to incur the overhead of string processing. Consider whether the value _needs_ to be a `string`. A good reason to keep it as a `string` would be if the variable is defined in an interface that this project does not own.

There are 5 instances:

- _ExecutorPlugin.sol_ ( [#L53](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ExecutorPlugin.sol#L53), [#L55](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ExecutorPlugin.sol#L55) ):

```solidity
53:     string private constant _NAME = "ExecutorPlugin";

55:     string private constant _VERSION = "1.0";
```

- _PolicyValidator.sol_ ( [#L26](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/PolicyValidator.sol#L26), [#L28](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/PolicyValidator.sol#L28) ):

```solidity
26:     string private constant _NAME = "PolicyValidator";

28:     string private constant _VERSION = "1.0";
```

- _SafeDeployer.sol_ ( [#L23](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeDeployer.sol#L23) ):

```solidity
23:     string public constant VERSION = "1";
```
