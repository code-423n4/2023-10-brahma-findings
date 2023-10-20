# Gas-Optmization

| Number | Gas                                                                                                      | context |
| :----: | :------------------------------------------------------------------------------------------------------- | :-----: |
| [G-01] | NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS                      |   21    |
| [G-02] | Use hardcode address instead address(this)                                                               |    2    |
| [G-03] | Internal functions not called by the contract should be removed to save deployment Gas                   |    5    |
| [G-04] | Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4                    |    6    |
| [G-05] | Use assembly to write address storage values                                                             |    4    |
| [G-06] | abi.encode() is less efficient than abi.encodePacked()                                                   |    4    |
| [G-07] | Avoid contract existence checks by using low level calls                                                 |   10    |
| [G-08] | Using Storage instead of memory for structs/arrays saves gas                                             |    2    |
| [G-09] | Expressions for constant values such as a call to keccak256(), should use immutable rather than constant |    1    |
| [G-10] | State variables should be cached in stack variables rather than re-reading them from storage             |    7    |
| [G-11] | Use ++i instead of i++ to increment                                                                      |    2    |
| [G-12] | Bytes constants are more efficient than string constants                                                 |    5    |
| [G-13] | Use assembly to perform efficient back-to-back calls                                                     |    8    |
| [G-14] | Multiple accesses of a mapping/array should use a local variable cache                                   |    8    |
| [G-15] | missing zero address checks in the constructor                                                           |    2    |
| [G-16] | USE BITMAPS TO SAVE GAS                                                                                  |    1    |
| [G-17] | Empty blocks should be removed or emit something                                                         |   10    |
| [G-18] |                                                                                                          |   15    |

## [G-01] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS

```solidity
File: src/libraries/TypeHashHelper.sol

65        return keccak256(

85        return keccak256(

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/TypeHashHelper.sol#L65

```solidity
File: src/libraries/SafeHelper.sol

94        return signatures;

144        return address(uint160(uint256(bytes32(guardAddress))));

154        return address(uint160(uint256(bytes32(fallbackHandlerAddress))));

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L94

```solidity
File: src/core/TransactionValidator.sol

173        return false;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L173

```solidity
File: src/core/ConsoleFallbackHandler.sol

61        return getMessageHashForSafe(GnosisSafe(payable(msg.sender)), message);

70        return keccak256(abi.encodePacked(bytes1(0x19), bytes1(0x01), safe.domainSeparator(), safeMessageHash));

86        return (value == EIP1271_MAGIC_VALUE) ? UPDATED_MAGIC_VALUE : bytes4(0);

95        return array;

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/ #L61

```solidity
File: src/core/registries/ExecutorRegistry.sol

68        return subAccountToExecutors[_subAccount].contains(_executor);

76        return subAccountToExecutors[_subAccount].values();

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol#L68

```solidity
File: src/core/registries/WalletRegistry.sol

64        return walletToSubAccountList[_wallet];

74        return subAccountToWallet[_subAccount] == _wallet;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L64

```solidity
File:src/core/AddressProviderService.sol


36        return address(addressProvider);

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/ src/core/AddressProviderService.sol#L36

```solidity
File: src/core/AddressProvider.sol

113        return authorizedAddresses[_key];

122        return registries[_key];

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L113

```solidity
File: src/core/PolicyValidator.sol

79        return isPolicySignatureValid(account, transactionStructHash, signatures);

141        return SignatureCheckerLib.isValidSignatureNow(trustedValidator, txnValidityDigest, validatorSignature);

175        return (_NAME, _VERSION);
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L79

## [G-02] Use hardcode address instead address(this)

```solidity
File: src/libraries/SafeHelper.sol

74            _generateSingleThresholdSignature(address(this)) // signatures
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L74

```solidity
File: contracts/src/core/AddressProvider.sol

131        if (IAddressProviderService(_newAddress).addressProviderTarget() != address(this)) {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L131C1-L132C1

## [G-03] Internal functions not called by the contract should be removed to save deployment Gas

```solidity
File: src/core/PolicyValidator.sol

174    function _domainNameAndVersion() internal pure override returns (string memory name, string memory version) {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L174

```solidity
File: src/core/AddressProviderService.sol

44    function _getRegistry(bytes32 _key) internal view returns (address registry) {

54    function _getAuthorizedAddress(bytes32 _key) internal view returns (address authorizedAddress) {

62    function _onlyGov() internal view {

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol#L44C1-L45C1

```solidity
File: src/core/ExecutorPlugin.sol


159    function _domainNameAndVersion() internal pure override returns (string memory name, string memory version) {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol

## [G-04] Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4

```solidity
File: src/core/SafeDeployer.sol

144            data: abi.encodePacked(WalletRegistry.registerWallet.selector)

254        return uint256(keccak256(abi.encodePacked(_ownersHash, ownerSafeCount[_ownersHash]++, _salt, VERSION)));
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L144

```solidity
File: src/libraries/SafeHelper.sol

88        bytes memory signatures = abi.encodePacked(

119            bytes memory encodedTxn = abi.encodePacked(

125                packedTxns = abi.encodePacked(packedTxns, encodedTxn);

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L88

```solidity
File: src/core/ConsoleFallbackHandler.sol

70        return keccak256(abi.encodePacked(bytes1(0x19), bytes1(0x01), safe.domainSeparator(), safeMessageHash));

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L70

## [G-05] Use assembly to write address storage values

```solidity
27    address public governance;

29    address public pendingGovernance;

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L27

```solidity

33    address internal immutable _self;

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L33

```solidity

25    AddressProvider public immutable addressProvider;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol#L25C1-L26C1

## [G-06] abi.encode() is less efficient than abi.encodePacked()

```solidity

66            abi.encode(

86            abi.encode(

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/TypeHashHelper.sol#L86

```solidity

69        bytes32 safeMessageHash = keccak256(abi.encode(SAFE_MSG_TYPEHASH, keccak256(message)));

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L69C1-L70C1

```solidity

225        bytes32 ownersHash = keccak256(abi.encode(_owners));

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L225C1-L226C1

## [G-07] Avoid contract existence checks by using low level calls

```solidity

85        bytes4 value = validator.isValidSignature(abi.encode(_dataHash), _signature);

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L85C1-L86C1

```solidity

73        TransactionValidator(AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH))

148        TransactionValidator(AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH))

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L73C1-L74C1

```solidity

45        registry = addressProvider.getRegistry(_key);

55        authorizedAddress = addressProvider.getAuthorizedAddress(_key);

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol#L55

```solidity

63        uint256 nonce = IGnosisSafe(account).nonce();


107            PolicyRegistry(AddressProviderService._getRegistry(_POLICY_REGISTRY_HASH)).commitments(account);

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L63

```solidity


143        bytes memory guardAddress = IGnosisSafe(safe).getStorageAt(_GUARD_STORAGE_SLOT, 1);


153        bytes memory fallbackHandlerAddress = IGnosisSafe(safe).getStorageAt(_FALLBACK_HANDLER_STORAGE_SLOT, 1);

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L143C1-L144C1

```solidity

194            WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH)).subAccountToWallet(_subAccount);

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L194

## [G-08] Using Storage instead of memory for structs/arrays saves gas

```solidity
File: src/core/SafeDeployer.sol

116        Types.Executable[] memory txns;

174        Types.Executable[] memory txns = new Types.Executable[](2);

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L116C1

## [G-09] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

```solidity
File: src/core/ConsoleFallbackHandler.sol

24    bytes4 internal constant SIMULATE_SELECTOR = bytes4(keccak256("simulate(address,bytes)"));

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L24

## [G-10] State variables should be cached in stack variables rather than re-reading them from storage

You will see the following pattern frequently in efficient solidity code. Reading from a storage variable costs at least 100 gas as Solidity does not cache the storage read. Writes are considerably more expensive. Therefore, you should manually cache the variable to do exactly one storage read and exactly one storage write.

```solidity
File: contracts/src/core/AddressProvider.sol

66        emit GovernanceTransferred(governance, msg.sender);
67        governance = msg.sender;



63        if (msg.sender != pendingGovernance) {
68        delete pendingGovernance;

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L66

```solidity
File: contracts/src/core/SafeEnabler.sol


48        require(module != address(0) && module != _SENTINEL_MODULES, "GS101");

53        modules[module] = modules[_SENTINEL_MODULES];

54        modules[_SENTINEL_MODULES] = module;

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L48

## [G-11] Use ++i instead of i++ to increment

The reason behind this is in way ++i and i++ are evaluated by the compiler.

i++ returns i(its old value) before incrementing i to a new value. This means that 2 values are stored on the stack for usage whether you wish to use it or not. ++i on the other hand, evaluates the ++ operation on i (i.e it increments i) then returns i (its incremented value) which means that only one item needs to be stored on the stack

```solidity

254        return uint256(keccak256(abi.encodePacked(_ownersHash, ownerSafeCount[_ownersHash]++, _salt, VERSION)));
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L254

```solidity

131                nonce: executorNonce[execRequest.account][execRequest.executor]++
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L131

## [G-12] Bytes constants are more efficient than string constants

```solidity
File: contracts/src/core/ExecutorPlugin.sol

53    string private constant _NAME = "ExecutorPlugin";

55    string private constant _VERSION = "1.0";
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L53

```solidity
File: src/core/PolicyValidator.sol

26    string private constant _NAME = "PolicyValidator";

28    string private constant _VERSION = "1.0";

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L26

```solidity
File: src/core/SafeDeployer.sol

23    string public constant VERSION = "1";

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L23

## [G-13] Use assembly to perform efficient back-to-back calls

```solidity
File: src/core/ConsoleFallbackHandler.sol

44            require(safe.signedMessages(messageHash) != 0, "Hash not approved");

51            require(policyValidator.isPolicySignatureValid(msg.sender, messageHash, _signature), "Policy not approved");
52            safe.checkSignatures(messageHash, _data, _signature);
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L44

```solidity
File: src/core/registries/PolicyRegistry.sol

50        } else if (walletRegistry.isOwner(msg.sender, account)) {

52        } else if (msg.sender == account && walletRegistry.isWallet(account)) {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L50

```solidity
File: contracts/src/core/SafeDeployer.sol

92        if (!_walletRegistry.isWallet(msg.sender)) revert NotWallet();

98        _walletRegistry.registerSubAccount(msg.sender, _subAcc);

101        PolicyRegistry(AddressProviderService._getRegistry(_POLICY_REGISTRY_HASH)).updatePolicy(_subAcc, _policyCommit);

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L92

## [G-14] Multiple accesses of a mapping/array should use a local variable cache

```solidity

36        if (isWallet[msg.sender]) revert AlreadyRegistered();

38        isWallet[msg.sender] = true;


51        if (subAccountToWallet[_subAccount] != address(0)) revert AlreadyRegistered();
52        subAccountToWallet[_subAccount] = _wallet;


```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L51C1-L52C51

```solidity


101        if (registries[_key] != address(0)) revert RegistryAlreadyExists();
102        registries[_key] = _registry;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L101C1-L102C38

```solidity


67        emit UpdatedPolicyCommit(account, policyCommit, commitments[account]);
68        commitments[account] = policyCommit;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L67C1-L68C45

## [G-15] missing zero address checks in the constructor

```solidity
File: src/core/SafeEnabler.sol

33        _self = address(this);

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L33

```solidity
File: src/core/AddressProvider.sol

45        governance = _governance;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L45

## [G-16] USE BITMAPS TO SAVE GAS

However, since it only takes one bit to store this information, and each slot is 256 bits, that means one can store a 256 flags/booleans with one storage slot.

```solidity
File: src/core/registries/WalletRegistry.sol

27    mapping(address => bool) public isWallet;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L27

## [G-17] Empty blocks should be removed or emit something

```solidity
File: /src/core/SafeModeratorOverridable.sol

23    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}




86    function checkModuleTransaction(
87        address, /* to */
88        uint256, /* value */
89        bytes memory, /* data */
90        Enum.Operation, /* operation */
91        address /* module */
92    ) external override {}
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol#L23C1-L24C1

```solidity
File: src/core/SafeModerator.sol


17    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}



80    function checkModuleTransaction(
81        address, /* to */
82        uint256, /* value */
83        bytes memory, /* data */
84        Enum.Operation, /* operation */
85        address /* module */
86    ) external override {}
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModerator.sol#L17

```solidity
File: src/core/ConsoleFallbackHandler.sol

29    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L29

```solidity
File: src/core/PolicyValidator.sol

30    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L30

```solidity
File: src/core/registries/PolicyRegistry.sol


24    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L24

```solidity
File: src/core/registries/ExecutorRegistry.sol

29    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol#L29C1-L30C1

```solidity
File: src/core/TransactionValidator.sol

54    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}


81    function validatePostTransactionOverridable(bytes32, /*txHash */ bool, /*success */ address /*console */ )
82        external
83        view
84    {}
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L54

## [G-18] Use solidity version 0.8.20 to gain some gas boost

```solidity

5   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol#L5

```solidity

5   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L5

```solidity

5   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModerator.sol#L5

```solidity

5   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/Constants.sol#L5

```solidity

5   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L5

```solidity

5   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L5

```solidity

5   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L5

```solidity

5   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol#L5

```solidity

5   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L5

```solidity

5   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol#L5

```solidity

5   pragma solidity 0.8.19;

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/TypeHashHelper.sol#L5

```solidity

5   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L5

```solidity

5   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L5
