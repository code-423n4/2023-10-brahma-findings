# Summary

| Id  | Title                      |
|:---:|:---------------------------|
| [L-01](#l-01-authorized-addresses-cant-be-unset) | Authorized addresses can't be unset |
| [L-02](#l-02-registries-cannot-be-removed) | Registries cannot be removed |
| [L-03](#l-03-policies-cant-be-unset-in-a-console-account) | Policies can't be unset in a console account |


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


### [L-03] Policies can't be unset in a console account


Console accounts allow a zero policy when they are initialized:

```solidity
	function deployConsoleAccount(address[] calldata _owners, uint256 _threshold, bytes32 _policyCommit, bytes32 _salt)
	    external
	    nonReentrant
	    returns (address _safe)
	{
>	    bool _policyHashValid = _policyCommit != bytes32(0);

	    _safe = _createSafe(_owners, _setupConsoleAccount(_owners, _threshold, _policyHashValid), _salt);

>	    if (_policyHashValid) {
	        PolicyRegistry(AddressProviderService._getRegistry(_POLICY_REGISTRY_HASH)).updatePolicy(
	            _safe, _policyCommit
	        ); 
	    }
	    emit ConsoleAccountDeployed(_safe);
	}
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L65-L69

However, if a console account initializes without policy, and sets a policy after the initialization, the account will never be able to unset it, as the transaction will revert:

```solidity
	function updatePolicy(address account, bytes32 policyCommit) external {
>	    if (policyCommit == bytes32(0)) {
	        revert PolicyCommitInvalid();
	    }

	    WalletRegistry walletRegistry = WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH));

	    bytes32 currentCommit = commitments[account];

	    // solhint-disable no-empty-blocks
	    if (
	        currentCommit == bytes32(0)
	            && msg.sender == AddressProviderService._getAuthorizedAddress(_SAFE_DEPLOYER_HASH)
	    ) {
	        // In case invoker is safe  deployer
	    } else if (walletRegistry.isOwner(msg.sender, account)) {
	        //In case invoker is updating on behalf of sub account
	    } else if (msg.sender == account && walletRegistry.isWallet(account)) {  
	        // In case invoker is a registered wallet
	    } else {
	        revert UnauthorizedPolicyUpdate();
	    }
	    // solhint-enable no-empty-blocks
	    _updatePolicy(account, policyCommit);
	}
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L36