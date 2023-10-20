# Summary

| Id  | Title                      |
|:---:|:---------------------------|
| [L-01](#l-01-authorized-addresses-cant-be-unset) | Authorized addresses can't be unset |
| [L-02](#l-02-registries-cannot-be-removed) | Registries cannot be removed |
| [L-03](#l-03-console-accounts-are-missing-post-transaction-validations) | Console accounts are missing post transaction validations |


### [L-01] Authorized addresses can't be unset

Once set, these addresses cannot be removed. If they contain bugs, or they are malicious, the governance will be unable to remove them:

```solidity
	function setAuthorizedAddress(bytes32 _key, address _authorizedAddress, bool _overrideCheck) external {
	    _onlyGov();
>	    _notNull(_authorizedAddress);

	    /// @dev skips checks for supported `addressProvider()` if `_overrideCheck` is true
	    if (!_overrideCheck) {
	        /// @dev skips checks for supported `addressProvider()` if `_authorizedAddress` is an EOA
	        if (_authorizedAddress.code.length != 0) _ensureAddressProvider(_authorizedAddress);
	    }

	    authorizedAddresses[_key] = _authorizedAddress; //@audit L - can't unset

	    emit AuthorizedAddressInitialised(_authorizedAddress, _key);
	}
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L79

### [L-02] Registries cannot be removed

Registries cannot be removed once set, and if they contain bugs, the whole system must be redeployed:

```solidity
	function setRegistry(bytes32 _key, address _registry) external {
	    _onlyGov();
	    _ensureAddressProvider(_registry);

>	    if (registries[_key] != address(0)) revert RegistryAlreadyExists();
	    registries[_key] = _registry;

	    emit RegistryInitialised(_registry, _key);
	}
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L101


### [L-03] Console accounts are missing post transaction validations


Unlike SubAccounts, Console accounts lack any kind of post transaction validation after a transaction:  

```solidity
	function validatePostTransactionOverridable(bytes32, /*txHash */ bool, /*success */ address /*console */ )
	    external
	    view
	{}
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L81-L84



