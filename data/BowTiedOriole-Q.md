### Low Issues
|Title|
|-|
|[L-1] SafeModerator Is Not Compatible With Function setGuard in Safe v1.4 |

Total: 1 issue


### Non-Critical Issues
|Title|
|-|
|[QA-1] ConsoleFallbackHandler Incompatibility with Safe v1.5.0

Total: 1 issue

## [L-1]: SafeModerator Is Not Compatible With Function setGuard in Safe v1.4

**Severity**: Low

**Effected Contract** `SafeModerator.sol` and `SafeModeratorOverrideable.sol`

## Description

In Gnosis Safe version 1.4, the following lines of code were added to the setGuard function in GuardManager. 

https://github.com/safe-global/safe-contracts/blob/main/contracts/base/GuardManager.sol#L89-L91

For this reason, they created the BaseGuard abstract contract which implements the `supportsInterface` function.

https://github.com/safe-global/safe-contracts/blob/main/contracts/base/GuardManager.sol#L61-L67

SafeModerator and SafeModeratorOverrideable do not implement the `supportsInterface` function and cannot be added via the default setGuard method on the safe.

## Vulnerability Details
If a user creates a console account with a v1.4 Safe, they will not be able to use the default setGuard function with SafeModerator and SafeModeratorOverrideable. 

The setGuard function will revert on L90 because SafeModerator and SafeModeratorOverrideable do not implement `supportsInterface`.

## Tools Used
Manual Review

## Recommendations
Adjust SafeModerator and SafeModeratorOverrideable to extend from the BaseGuard abstract contract.

## [QA-1] - ConsoleFallbackHandler Incompatibility with Safe v1.5.0

**Severity**: QA

**Effected Contract:** `ConsoleFallbackHandler.sol`

## Description
Safe Global is planning to release safe-contracts v1.5.0. In this version, the function signature `checkNSignatures` for the Safe singleton was changed. To ensure compatibility with usage of this function, the old, overloaded function was added to the compatibility fallback handler in [this PR](https://github.com/safe-global/safe-contracts/pull/589/). To ensure that all compatibility usecases are also handled by the consoleFallbackHandler, the overloaded function should be added to the fallback handler when the protocol updates the proxy factory to use safe-contracts deployment v1.5.0.

## Tools Used
Manual Review

## Recommendations
When Brahma Console's AddressProvider is updated to use the soon-to-be-released v1.5.0 contracts, update and re-deploy a new version of the ConsoleFallbackHandler to include the [overloaded](https://github.com/safe-global/safe-contracts/blob/main/contracts/handler/CompatibilityFallbackHandler.sol#L171-L172) `checkNSignatures` function.