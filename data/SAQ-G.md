## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4 | 6 | - |
| [G-02] | abi.encode() is less efficient than abi.encodePacked() | 4 | - |
| [G-03] | Using Storage instead of memory for structs/arrays saves gas | 2 | - |
| [G-04] | Use assembly to write address storage values | 4 | - |
| [G-05] | State variables should be cached in stack variables rather than re-reading them from storage | 7 | - |
| [G-06] | Use hardcode address instead address(this) | 2 | - |
| [G-07] | Not using the named return variable when a function returns, wastes deployment gas | 20 | - |
| [G-08] | Bytes constants are more efficient than string constants | 5 | - |
| [G-09] | Multiple accesses of a mapping/array should use a local variable cache | 8 | - |
| [G-10] | Internal functions not called by the contract should be removed to save deployment Gas | 5 | - |
| [G-11] | Avoid contract existence checks by using low level calls | 10 | - |
| [G-12] | missing zero address checks in the constructor | 2 | - |
| [G-13] | Empty blocks should be removed or emit something | 10 | - |
| [G-14] | Use solidity version 0.8.20 to gain some gas boost |13  | - |
| [G-15] | Expressions for constant values such as a call to keccak256(), should use immutable rather than constant | 1 | - |
| [G-16] | Use assembly to perform efficient back-to-back calls | 7 | - |
| [G-17] | Use ++i instead of i++ to increment | 2 | - |
| [G-18] | USE BITMAPS TO SAVE GAS | 1 | - |
| [G-19] | State variables only set in the constructor should be declared immutable | 1 | - |
| [G-20] | Make 3 event parameters indexed when possible | 3 | - |
| [G-21] | Do not initialize variables with default values | 2 | - |
| [G-22] | More likely to revert operations should be executed first | 1 | - |
| [G-23] | Access mappings directly rather than using accessor functions | 6 | - |

## Gas Optimizations  


## [G-01] Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4

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


## [G-02] abi.encode() is less efficient than abi.encodePacked()

In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

Refference: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison

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


## [G-03] Using Storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```solidity
File: src/core/SafeDeployer.sol

116        Types.Executable[] memory txns;

174        Types.Executable[] memory txns = new Types.Executable[](2);

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L116C1


## [G-04] Use assembly to write address storage values

By using assembly to write to address storage values, you can bypass some of these operations and lower the gas cost of writing to storage. Assembly code allows you to directly access the Ethereum Virtual Machine (EVM) and perform low-level operations that are not possible in Solidity.

example of using assembly to write to address storage values:
```solidity
contract MyContract {
    address private myAddress;
    
    function setAddressUsingAssembly(address newAddress) public {
        assembly {
            sstore(0, newAddress)
        }
    }
}
```
Instances:

```solidity
file: /contracts/src/core/AddressProvider.sol

27    address public governance;

29    address public pendingGovernance;

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L27

```solidity
file: /contracts/src/core/SafeEnabler.sol

33    address internal immutable _self;

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L33

```solidity
file: /contracts/src/core/AddressProviderService.sol

25    AddressProvider public immutable addressProvider;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol#L25C1-L26C1


## [G-05] State variables should be cached in stack variables rather than re-reading them from storage



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

## [G-06] Use hardcode address instead address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.
References: https://book.getfoundry.sh/reference/forge-std/compute-create-address

 if the contract's address is needed in the code, it's more gas-efficient to hardcode the address as a constant rather than using the address(this) expression. This is because using address(this) requires additional gas consumption to compute the contract's address at runtime.

an example :
```solidity
contract MyContract {
    address constant public CONTRACT_ADDRESS = 0x1234567890123456789012345678901234567890;
    
    function getContractAddress() public view returns (address) {
        return CONTRACT_ADDRESS;
    }
}
```
Instances:
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


## [G-07] Not using the named return variable when a function returns, wastes deployment gas

When you execute a function that returns values in Solidity, the EVM still performs the necessary operations to execute and return those values. This includes the cost of allocating memory and packing the return values. If the returned values are not utilized, it can be seen as wasteful since you are incurring gas costs for operations that have no effect.

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


## [G-08] Bytes constants are more efficient than string constants

If data can fit into 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is cheaper in solidity.

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


## [G-09] Multiple accesses of a mapping/array should use a local variable cache

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping’s value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Caching an array’s struct avoids recalculating the array offsets into memory/calldata.

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


## [G-10] Internal functions not called by the contract should be removed to save deployment Gas

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

## [G-11] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.							


```solidity
file: /contracts/src/core/ConsoleFallbackHandler.sol

85        bytes4 value = validator.isValidSignature(abi.encode(_dataHash), _signature);

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L85C1-L86C1

```solidity
file: contracts/src/core/ExecutorPlugin.sol

73        TransactionValidator(AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH))

148        TransactionValidator(AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH))

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L73C1-L74C1

```solidity
file: /contracts/src/core/AddressProviderService.sol

45        registry = addressProvider.getRegistry(_key);

55        authorizedAddress = addressProvider.getAuthorizedAddress(_key);

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol#L55

```solidity
file: /contracts/src/core/PolicyValidator.sol

63        uint256 nonce = IGnosisSafe(account).nonce();


107            PolicyRegistry(AddressProviderService._getRegistry(_POLICY_REGISTRY_HASH)).commitments(account);

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L63

```solidity
file: /contracts/src/libraries/SafeHelper.sol

143        bytes memory guardAddress = IGnosisSafe(safe).getStorageAt(_GUARD_STORAGE_SLOT, 1);


153        bytes memory fallbackHandlerAddress = IGnosisSafe(safe).getStorageAt(_FALLBACK_HANDLER_STORAGE_SLOT, 1);

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L143C1-L144C1

```solidity
file: /contracts/src/core/TransactionValidator.sol

194            WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH)).subAccountToWallet(_subAccount);

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L194


## [G-12] missing zero address checks in the constructor

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

## [G-13] Empty blocks should be removed or emit something

The gas cost of an empty constructor block in Solidity is typically in the range of 200-500 gas units. 

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.
If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation.

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

## [G-14] Use solidity version 0.8.20 to gain some gas boost

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


## [G-15] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

```solidity
File: src/core/ConsoleFallbackHandler.sol

24    bytes4 internal constant SIMULATE_SELECTOR = bytes4(keccak256("simulate(address,bytes)"));

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L24


## [G-16] Use assembly to perform efficient back-to-back calls

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

## [G-17] Use ++i instead of i++ to increment

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

## [G-18] USE BITMAPS TO SAVE GAS

However, since it only takes one bit to store this information, and each slot is 256 bits, that means one can store a 256 flags/booleans with one storage slot.

```solidity
File: src/core/registries/WalletRegistry.sol

27    mapping(address => bool) public isWallet;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L27

## [G‑19] State variables only set in the constructor should be declared immutable

Avoids a Gsset (20000 gas) in the constructor, replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

```solidity
file:  src/core/SafeEnabler.sol

34     _self = address(this);

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L34


## [G-20] Make 3 event parameters indexed when possible 


It’s the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

```solidity
file:  src/core/SafeEnabler.sol

19    event EnabledModule(address module);

20    event ChangedGuard(address guard);

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L19

```solidity
file:  src/core/registries/PolicyRegistry.sol

19    event UpdatedPolicyCommit(address indexed account, bytes32 policyCommit, bytes32 oldPolicyCommit);

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L19


## [G-21] Do not initialize variables with default values

Uninitialized variables are assigned with the types default value. Explicitly initializing a variable with it's default value costs unnecesary gas.

```solidity
file:  src/core/SafeModeratorOverridable.sol

21     uint8 public constant DIFFER_SAFE_MOD = 0;

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol#L21

```solidity
file:  /contracts/src/libraries/SafeHelper.sol

107     uint256 i = 0;

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L107

## [G-22] More likely to revert operations should be executed first

```solidity
file: /contracts/src/core/ExecutorPlugin.sol

117     if (execRequest.executor.code.length == 0 && execRequest.executorSignature.length == 0) {
            // Executor is an EOA and no executor signature is provided
            revert InvalidSignature();
120        }

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L117-L120

## [G-23] Access mappings directly rather than using accessor functions

When you have a mapping, accessing its values through accessor functions involves an additional layer of indirection, which can incur some gas cost. This is because accessing a value from a mapping typically involves two steps: first, locating the key in the mapping, and second, retrieving the corresponding value.

```solidity
file:  src/core/AddressProvider.sol

113    return authorizedAddresses[_key];

122    return registries[_key];

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L113

```solidity
file:  src/core/registries/ExecutorRegistry.sol

68     return subAccountToExecutors[_subAccount].contains(_executor);

76     return subAccountToExecutors[_subAccount].values();

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol#L68

```solidity
file:  src/core/registries/WalletRegistry.sol

64    return walletToSubAccountList[_wallet];

74    return subAccountToWallet[_subAccount] == _wallet;

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L64
