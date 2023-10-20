## Summary

### Non-critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [[N&#x2011;01](#n01-utilize-is-owner)] | Prefer utilizing the `isOwner()` method to ascertain the owner of a sub-account. | 1 |
| [[N&#x2011;02](#n02-missing-address-zer-check)] | Missing zero-address check in the setter functions | 2 |
| [[N&#x2011;03](#n03-enhance-error-names)] | Enhance error names for better clarity and explanation | 3 |
| [[N&#x2011;04](#n04-impliment-modifier)] | Consider implementing a modifier instead of the view function for access control | 1 |
| [[N&#x2011;05](#n05-move-not-null-to-common-file)] | Consider moving `_notNull()` to a common file and use it instead of `addr == address(0)` checks in throughout the project | 1 |
| [[N&#x2011;06](#n06-move-constant-to-constant-file)] | Consider moving the constant variables to `Constants.sol` for better code styling | 13 |
| [[N&#x2011;07](#n07-provide-meaningful-messages)] | Provide meaningful  messages in `require` statement | 2 |


## Non-critical Issues

### [N&#x2011;01] Prefer utilizing the `isOwner()` method to ascertain the owner of a sub-account.

For the consistency in the code by employing `isOwner()` instead of verifying `subAccountToWallet(_subAccount) != msg.sender`.

*There is 1 instance of this issue:*

```solidity

File: src/core/registries/ExecutorRegistry.sol

55:        if (_walletRegistry.subAccountToWallet(_subAccount) != msg.sender) revert NotOwnerWallet();

```
*GitHub*: [55](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/ExecutorRegistry.sol#L55)

### [N&#x2011;02] Missing zero-address check in the setter functions

Some of the critical setter functions are missing address(0) checks

*There is 2 instance of this issue:*

```solidity

File: src/core/registries/ExecutorRegistry.sol

38:    function registerExecutor(address _subAccount, address _executor) external {
39:        WalletRegistry _walletRegistry = WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH));
40:        if (!_walletRegistry.isOwner(msg.sender, _subAccount)) revert NotOwnerWallet();

```
*GitHub*: [38](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/ExecutorRegistry.sol#L38C1-L40C88)

Lack of check for address(0) in _executor can result in accidentally setting the executor as the zero address.

```solidity

File: src/core/registries/WalletRegistry.sol

49:    function registerSubAccount(address _wallet, address _subAccount) external {
50:        if (msg.sender != AddressProviderService._getAuthorizedAddress(_SAFE_DEPLOYER_HASH)) revert InvalidSender();
51:        if (subAccountToWallet[_subAccount] != address(0)) revert AlreadyRegistered();

```
*GitHub*: [49](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/WalletRegistry.sol#L49)

This allow, address(0) can be set as sub account.

### [N&#x2011;03] Enhance error names for better clarity and explanation

The existing error messages lack sufficient detail to accurately communicate the reasons for the errors.

*There is 3 instance of this issue:*

```solidity

File: src/core/registries/ExecutorRegistry.sol

20:    error AlreadyExists();
21:    error DoesNotExist();

```
*GitHub*: [20](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/ExecutorRegistry.sol#L20C4-L21C26)

The above errors are not providing anything about what is already existing and what does not exists, Use something like 

```solidity

    error SubAccountAlreadyExists();
    error SubAccountDoesNotExist();
```

```solidity

File: src/core/registries/WalletRegistry.sol

15:    error AlreadyRegistered();

35:    function registerWallet() external {
36:        if (isWallet[msg.sender]) revert AlreadyRegistered();

49:    function registerSubAccount(address _wallet, address _subAccount) external {
50:        if (msg.sender != AddressProviderService._getAuthorizedAddress(_SAFE_DEPLOYER_HASH)) revert InvalidSender();
51:        if (subAccountToWallet[_subAccount] != address(0)) revert AlreadyRegistered();
```

*GitHub*: [15](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/WalletRegistry.sol#L15), [35](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/WalletRegistry.sol#L35), [49](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/WalletRegistry.sol#L49)

`AlreadyRegistered` is used for both wallet registration and sun account registration

### [N&#x2011;04] Consider implementing a modifier instead of the view function for access control

*There is 1 instance of this issue:*

```solidity

File: src/core/AddressProvider.sol

139:    function _onlyGov() internal view {
140:        if (msg.sender != governance) revert NotGovernance(msg.sender);
141:    }

```
*GitHub*: [139](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/AddressProvider.sol#L139C1-L141C6)

### [N&#x2011;05] Consider moving `_notNull()` to a common file and use it instead of `addr == address(0)` checks in throughout the project

*There is 1 instance of this issue:*

```solidity

File: src/core/AddressProvider.sol

147:    function _notNull(address addr) internal pure {
148:        if (addr == address(0)) revert NullAddress();
150:    }

```
*GitHub*: [147](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/AddressProvider.sol#L147C1-L149C6)

### [N&#x2011;06] Consider moving the constant variables to `Constants.sol` for better code styling

*There is 13 instance of this issue:*

```solidity

File: src/core/ConsoleFallbackHandler.sol

22:    bytes32 private constant SAFE_MSG_TYPEHASH = 0x60b3cbf8b4a223d68d641b3b6ddf9a298e7f33710cf3d3a9d1146b5a6150fbca;

24:    bytes4 internal constant SIMULATE_SELECTOR = bytes4(keccak256("simulate(address,bytes)"));

26:    address internal constant SENTINEL_MODULES = address(0x1);
27:    bytes4 internal constant UPDATED_MAGIC_VALUE = 0x1626ba7e;

```
*GitHub*: [22](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ConsoleFallbackHandler.sol#L22C1-L27C63)

```solidity

File: src/core/SafeDeployer.sol

29:    bytes32 internal constant _SAFE_CREATION_FAILURE_REASON =
30:        0xd7c71a0bdd2eb2834ad042153c811dd478e4ee2324e3003b9522e03e7b3735dc;
```

*GitHub*: [29](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeDeployer.sol#L29C4-L30C76)

```solidity

File: src/core/SafeEnabler.sol

30    bytes32 internal constant _GUARD_STORAGE_SLOT = 0x4a204f620c8c5ccdca3fd54d003badd85ba500436a431f0cbda4f558c93c34c8;

```

*GitHub*: [30](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeEnabler.sol#L30C1-L30C120)

```solidity

File: src/core/SafeModeratorOverridable.sol

21:    uint8 public constant DIFFER_SAFE_MOD = 0;

```

*GitHub*: [21](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeModeratorOverridable.sol#L21)

```solidity

File: src/libraries/SafeHelper.sol

23:    uint256 internal constant _GUARD_STORAGE_SLOT =
24:        33528237782592280163068556224972516439282563014722366175641814928123294921928;

27:    uint256 internal constant _FALLBACK_HANDLER_STORAGE_SLOT =
28        49122629484629529244014240937346711770925847994644146912111677022347558721749;

37:    bytes32 internal constant _GUARD_REMOVAL_CALLDATA_HASH =
38:        0xc0e2c16ecb99419a40dd8b9c0b339b27acebd27c481a28cd606927aeb86f5079;

47:    bytes32 internal constant _FALLBACK_REMOVAL_CALLDATA_HASH =
48:        0x5bdf8c44c012c1347b2b15694dc5cc39b899eb99e32614676b7661001be925b7;
```

*GitHub*: [23](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/libraries/SafeHelper.sol#L23C1-L24C87), [27](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/libraries/SafeHelper.sol#L27C1-L28C87), [37](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/libraries/SafeHelper.sol#L37C1-L38C76), [47](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/libraries/SafeHelper.sol#L47C1-L48C76) 

```solidity

File: src/libraries/TypeHashHelper.sol

49:    bytes32 public constant TRANSACTION_PARAMS_TYPEHASH =
50:        0xfd4628b53a91b366f1977138e2eda53b93c8f5cc74bda8440f108d7da1e99290;

56:    bytes32 public constant VALIDATION_PARAMS_TYPEHASH =
57:        0x0c7f653e0f641e41fbb4ed1440c7d0b08b8d2a19e1c35cfc98de2d47519e15b1;

```

*GitHub*: [49](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/libraries/TypeHashHelper.sol#L49C1-L50C76), [56](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/libraries/TypeHashHelper.sol#L56C5-L57C76)

### [N&#x2011;07] Provide meaningful  messages in `require` statement

*There is 2 instance of this issue:*

```solidity

File: src/core/SafeEnabler.sol

48:        require(module != address(0) && module != _SENTINEL_MODULES, "GS101");

52:        require(modules[module] == address(0), "GS102");

```
*GitHub*: [48](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeEnabler.sol#L48C1-L52C57)