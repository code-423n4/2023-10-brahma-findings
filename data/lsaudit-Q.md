# [QA-01] Link to documentation in NatSpec leads to non-existing webpage

[File: src/libraries/SafeHelper.sol](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/libraries/SafeHelper.sol#L82)
```
 * @dev Refer to https://docs.safe.global/learn/safe-core/safe-core-protocol/signatures#pre-validated-signatures
```

Above link returns 404 webpage. Update the link so it will point out to a current documentation.

# [QA-2] `getModules()` returns always 10 first modules

If there are more than 10 modules, user should have an ability to list more than just 10 modules:

[File: src/core/ConsoleFallbackHandler.sol](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ConsoleFallbackHandler.sol#L91-L96)
```
    function getModules() external view returns (address[] memory) {
        // Caller should be a Safe
        GnosisSafe safe = GnosisSafe(payable(msg.sender));
        (address[] memory array,) = safe.getModulesPaginated(SENTINEL_MODULES, 10);
        return array;
    }
```
Our recommendation is to rewrite `getModules()`, so that it will take the number of returned modules from the user:

```
    function getModules(uint numberOfModulesToReturn) external view returns (address[] memory) {
        // Caller should be a Safe
        GnosisSafe safe = GnosisSafe(payable(msg.sender));
        (address[] memory array,) = safe.getModulesPaginated(SENTINEL_MODULES, numberOfModulesToReturn);
        return array;
    }
```

# [QA-3] PolicyValidator will revert after Sun Feb 07 2106

[File: src/core/PolicyValidator.sol](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/PolicyValidator.sol#L116)
```
if (expiryEpoch < uint32(block.timestamp)) {
(...)
```

Function `isPolicySignatureValid` from `PolicyValidator.sol` casts `block.timestamp` to `uint32`. Since the max value of `uint32` is `4294967295`, converting this number to timestamp leads to `Sun Feb 07 2106 06:28:15 GMT+0000`.

This implies that contract will stop working correctly after Feb 07 2106. Most of the reverts during improper datatype casting are categorized as Medium/High. However, since this is almost 83 years from now - the severity of this issue had been reduced to QA.

# [QA-4] There's no explicitly way to remove previously added Authorized Address

`AddressProvider.sol` implements a function which allows to set new authorized address, however, it does not implement any functionality which would allow removing that address.

Function `setAuthorizedAddress`, while adding new authorized address performs below operation:

[File: src/core/AddressProvider.sol](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/AddressProvider.sol#L87)
```
authorizedAddresses[_key] = _authorizedAddress;
```
The only way, to remove `_authorizedAddress` is to assign `_key` to a different address. The most reasonable solution would be overwritting `authorizedAddresses[_key]` with `address(0)`.
However, there's an issue with that idea, thus `setAuthorizedAddress` checks if  `_authorizedAddress` is not address(0) at line 79: `_notNull(_authorizedAddress);`

This implies that there's no explicitly way to remove previously set authorized address. To remove it, we would need to assign `key` to any other `non-zero` address. Assigning `key` to a different non-zero address basically means that, that non-zero address becomes authorized address. This might lead to confusion. There should be more explicit way of removing previously added authorized address.

Our recommendation is to implement additional function (make sure to make it callable only by governance - `_onlyGov()`) which allows to remove authorized address.

# [QA-5] Previously registered wallet cannot be removed

[File: src/core/registries/WalletRegistry.sol](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/WalletRegistry.sol#L35-L40)
```
function registerWallet() external {
        if (isWallet[msg.sender]) revert AlreadyRegistered();
        if (subAccountToWallet[msg.sender] != address(0)) revert IsSubAccount();
        isWallet[msg.sender] = true;
        emit RegisterWallet(msg.sender);
    }
```

`WalletRegistry.sol` implements only `registerWallet` function which is responsible for registering new wallet. There isn't any function which is responsible for removing previously registered wallet. When user - for any reason - would like to remove previously registered wallet - he/she won't be able to do that. Our recommendation is to implement additional function which would allow to remove previously registered wallet. While implementing that function - please keep in mind, that corresponding subAccounts should also be removed, when the wallet will be removed.


# [N-01] Grammar errors in comments

[File: src/core/ConsoleFallbackHandler.sol](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ConsoleFallbackHandler.sol#L89)
```
/// @dev Returns array of first 10 modules.
```

`Returns array of first 10` should be changed to `Returns array of the first 10`.

# [N-02] Inconsistency in emitting events

In `src/core/AddressProvider.sol`, functions `setGovernance` and `acceptGovernance` emits the event before the state changes:

```
emit GovernanceTransferRequested(governance, _newGovernance);
pendingGovernance = _newGovernance;

(...)

emit GovernanceTransferred(governance, msg.sender);
governance = msg.sender;
delete pendingGovernance;
```

while in functions `setAuthorizedAddress` and  `setRegistry`, the event is emitted after the state changes:

```
authorizedAddresses[_key] = _authorizedAddress;
emit AuthorizedAddressInitialised(_authorizedAddress, _key);

(...)

registries[_key] = _registry;
emit RegistryInitialised(_registry, _key);
```

# [N-03] Inconsistency in checking for `onlyGov` and `notNull`

In `src/core/AddressProvider.sol`:

function `setAUthorizedAddress` firsly checks if function is called by governance, and then, if provided address is not address(0), while function `setGovernance` firstly checks if provided address is not address(0), and then - if function was called by governance:

```
 function setGovernance(address _newGovernance) external {
        _notNull(_newGovernance);
        _onlyGov();

        (...)

function setAuthorizedAddress(bytes32 _key, address _authorizedAddress, bool _overrideCheck) external {
        _onlyGov();
        _notNull(_authorizedAddress);
```

This issue does not pose any security risk, thus it was categorized as `N`. Nonetheless, it's good practice to prepare every checks and validations in a robust, consistent way.

