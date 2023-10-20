# Low Risk Issues

## Summary
|	|	Issue								| Instances	|
| ---	|	-----								|	-----	|
| [L-01]| Missing checks for address(0x0) when assigning values to address state variables	|	03	|
| [L-02]|	Missing checks for bytes32(0)								|	04	|
| [L-03]| Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from various type int/uint values									|	06	|

# [L-01] Missing checks for address(0x0) when assigning values to address state variables 
Zero-address check should be used, to avoid the risk of setting a storage variable as address(0) at deploying time.

Link to the code:
1.	https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L35
2.	https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L68
3.	https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol#L39

## Recommendation
The recommendation is made for adding the below check to avoid zero address setting.
```diff
File: contracts/src/core/registries/PolicyRegistry.sol

    function updatePolicy(address account, bytes32 policyCommit) external {
+	require(account != address(0), address cannot be zero);
        if (policyCommit == bytes32(0)) {
            revert PolicyCommitInvalid();
        }

    function _updatePolicy(address account, bytes32 policyCommit) internal {
+	require(account != address(0), address cannot be zero);
        emit UpdatedPolicyCommit(account, policyCommit, commitments[account]);
        commitments[account] = policyCommit;
    }
```

```diff
File: contracts/src/core/registries/ExecutorRegistry.sol

    function registerExecutor(address _subAccount, address _executor) external {
+	require(_executor != address(0), address cannot be zero);
        WalletRegistry _walletRegistry = WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH));
        if (!_walletRegistry.isOwner(msg.sender, _subAccount)) revert NotOwnerWallet();

        if (!subAccountToExecutors[_subAccount].add(_executor)) revert AlreadyExists();
        emit RegisterExecutor(_subAccount, msg.sender, _executor);
    }
```


# [L-02] Missing checks for bytes32(0)
Zero-byte check should be used, to avoid the risk of setting a storage variable as address(0) at deploying time.

Link to the code:
1.	https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol#L46
2.	https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol#L56
3.	https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L102
4.	https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L68

## Recommendation
The recommendation is made for adding the below check to avoid zero bytes setting.

```diff
File: contracts/src/core/registries/PolicyRegistry.sol

L:66   function _updatePolicy(address account, bytes32 policyCommit) internal {
+	require(policyCommit != bytes32(0), bytes cannot be zero);
        emit UpdatedPolicyCommit(account, policyCommit, commitments[account]);
        commitments[account] = policyCommit;
    }
```

```diff
File: contracts/src/core/AddressProvider.sol

    function setRegistry(bytes32 _key, address _registry) external {
     		 _onlyGov();
       	 _ensureAddressProvider(_registry);
+ L100:	require(_key != bytes32(0), key cannot be zero);
     	     if (registries[_key] != address(0)) revert RegistryAlreadyExists();
      		  registries[_key] = _registry;
      	  emit RegistryInitialised(_registry, _key);
    }
```

```diff
File: contracts/src/core/AddressProviderService.sol

    function _getRegistry(bytes32 _key) internal view returns (address registry) {
+	require(_key != bytes32(0), key cannot be zero);
        registry = addressProvider.getRegistry(_key);
        _notNull(registry);
    }

    function _getAuthorizedAddress(bytes32 _key) internal view returns (address authorizedAddress) {
+	require(_key != bytes32(0), key cannot be zero);
        authorizedAddress = addressProvider.getAuthorizedAddress(_key);
        _notNull(authorizedAddress);
    }

```

# [L-03] Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from various type int/uint values

Link to the code:
1.	https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L110
2.	https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L112
3.	https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L144
4.	https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L154
5.	https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L116
6.	https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L128

