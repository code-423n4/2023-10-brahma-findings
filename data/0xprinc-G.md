## 1. The term `override` can be removed while declaring the function `AddressProviderService.sol/addressProviderTarget()` to save deployment gas cost
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol#L35

This can be done as interface functions don't need an `override` term in their declaration after inheritance.

## 2. Better to store the constant variable `SafeDeployer.sol/VERSION` in form of `bytes memory` to save gas
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L23

## 3. Better to use `abi.encode` instead of `abi.encodePacked` to save some gas while doing a `keccak256` hash
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L253-L255

In function `SafeDeployer.sol/_genNonce(...)` we are using `abi.encodePacked` which can be replaced with `abi.encode` which will not affect the keccak hash output.