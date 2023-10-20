## 1. The function `AddressProviderService.sol/_onlyGov()` is redundant and never used.
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol#L62-L66

The function `AddressProvider.sol/_onlyGov()` is used for validating the `msg.sender` but this same function present in `AddressProviderService.sol` is never used and also redundant as this contract is not changing or updating and storage variable and hence will not need any function to validate the `msg.sender` to only view the state.

## 2. Better to make the function `AddressProviderService.sol/_notNull(address)` return the address taken as argument. 
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol#L72

This will reduce the code size and memory usage when calling the functions `_getRegistry(bytes32 key)` and `_getAuthorizedAddress(bytes32 key)` 

The better version of `_notNull(address)` can be 
```Solidity
    function _notNull(address _addr) internal pure returns(address){      
        if (_addr == address(0)) revert InvalidAddress();
        return _addr;
    }
```

which optimize the implementation of functions `_getRegistry(bytes32 key)` and `_getAuthorizedAddress(bytes32 key)`

```Solidity
    function _getRegistry(bytes32 _key) internal view returns (address) {
        return _notNull(addressProvider.getRegistry(_key));
    }

    function _getAuthorizedAddress(bytes32 _key) internal view returns (address) {
        return _notNull(addressProvider.getAuthorizedAddress(_key));
    }
```

## 3. The event and error related to precomputation of address in `SafeDeployer.sol` are never used in any function
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L35
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L39

``` Solidity
    event PreComputeAccount(address[] indexed owners, uint256 indexed threshold);
    error PreComputedAccount(address addr);
```

## 4. Misleading comment of the function `WalletRegistry.sol/registerWallet()`.
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L33-L35

The comment 
``` Solidity 
    @dev Can only be called by safe deployer or the wallet itself
```
is misleading as the wallet can also be registered/called by any address which is not a wallet and subaccount.