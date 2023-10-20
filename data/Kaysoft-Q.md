## [L-1] No Address zero check on the setRegistry._registry parameter
File: https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/AddressProvider.sol#L97

```solidity
 function setRegistry(bytes32 _key, address _registry) external {
        _onlyGov();
        _ensureAddressProvider(_registry);
++      //@audit check _notNull(_registry);
        if (registries[_key] != address(0)) revert RegistryAlreadyExists();
        registries[_key] = _registry;//@audit can only be set once. what happens if there was a mistake.

        emit RegistryInitialised(_registry, _key);
    }


```

