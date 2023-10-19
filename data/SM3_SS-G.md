
## Gas Optimizations

| Number | Issue | Instances | 
|--------|-------|-----------|
|[G-01]| Use hardcoded address instead of address(this)  |  4  |
|[G-02]| Use uint256(1)/uint256(2) instead for true and false boolean states   |  6  |
|[G-03]| abi.encode() is less efficient than abi.encodepacked()   |  5  |
|[G-04]| Pack structs by putting data types that can fit together next to each other  |  2  |
|[G-05]| State variables only set in the constructor should be declared immutable  |  1  |
|[G-06]| Gas saving is achieved by removing the delete keyword (~60k)  |  1  |
|[G-07]| Make 3 event parameters indexed when possible   |  3  |
|[G-08]| Empty blocks should be removed or emit something  |  13  |
|[G-09]| Using fixed bytes is cheaper than using string   |  5  |
|[G-10]| Do not initialize variables with default values  |  2  |
|[G-11]| More likely to revert operations should be executed first  |  1  |
|[G-12]| Avoid contract existence checks by using low level calls  |  3  |
|[G-13]| Use assembly to perform efficient back-to-back calls  |  2  |
|[G-14]| Access mappings directly rather than using accessor functions  |  6  |
|[G-15]| public functions not called by the contract should be declared external instead  |  3  |

## [G-01] Use hardcoded address instead of address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this. Refrences

```solidity
file:  src/core/AddressProvider.sol

131    if (IAddressProviderService(_newAddress).addressProviderTarget() != address(this)) {

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L131

```solidity
file:   src/core/SafeEnabler.sol

33     _self = address(this);

82     if (address(this) == _self) revert OnlyDelegateCall();

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L33

```solidity
file:  src/libraries/SafeHelper.sol

74    _generateSingleThresholdSignature(address(this)) // signatures

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L74

## [G-02] Use uint256(1)/uint256(2) instead for true and false boolean states 

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. see source:

```solidity
file:  src/core/registries/WalletRegistry.sol

38     isWallet[msg.sender] = true;

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L38

```solidity
file:  src/core/SafeDeployer.sol

51     bool _policyHashValid = _policyCommit != bytes32(0);

110    function _setupConsoleAccount(address[] memory _owners, uint256 _threshold, bool _policyHashValid)

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L110

```solidity
file:  src/core/AddressProvider.sol

77    function setAuthorizedAddress(bytes32 _key, address _authorizedAddress, bool _overrideCheck) external {

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L77

```solidity
file:  src/core/SafeModerator.sol

70   function checkAfterExecution(bytes32 txHash, bool success) external view override {

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModerator.sol#L70

```solidity
file:  src/core/SafeModeratorOverridable.sol

76    function checkAfterExecution(bytes32 txHash, bool success) external view override {
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol#L76

## [G-03] abi.encode() is less efficient than abi.encodepacked() 

In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

Refference: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison

```solidity
file: src/core/ConsoleFallbackHandler.sol

69    bytes32 safeMessageHash = keccak256(abi.encode(SAFE_MSG_TYPEHASH, keccak256(message)));

85    bytes4 value = validator.isValidSignature(abi.encode(_dataHash), _signature);

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L69

```solidity
file: src/core/SafeDeployer.sol

225    bytes32 ownersHash = keccak256(abi.encode(_owners));

``` 
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L225

```solidity
file: src/libraries/TypeHashHelper.sol

66     abi.encode(

86     abi.encode(

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/TypeHashHelper.sol#L66


## [G-04] Pack structs by putting data types that can fit together next to each other

As the solidity EVM works with 32 bytes, variables less than 32 bytes should be packed inside a struct so that they can be stored in the same slot. This saves gas when writing to storage ~20000 gas



```solidity
file:   src/core/TransactionValidator.sol

39       struct SafeTransactionParams {
        Enum.Operation operation;
        address from;
        address to;
        address payable refundReceiver;
        address gasToken;
        address msgSender;
        uint256 value;
        uint256 safeTxGas;
        uint256 baseGas;
        uint256 gasPrice;
        bytes data;
        bytes signatures;
    }

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L39-L52

```solidity
file:  src/libraries/TypeHashHelper.sol

23    struct Transaction {
        uint8 operation;
        address to;
        address account;
        address executor;
        uint256 value;
        uint256 nonce;
        bytes data;
    }

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/TypeHashHelper.sol#L23-L31

## [G‑05] State variables only set in the constructor should be declared immutable

Avoids a Gsset (20000 gas) in the constructor, replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

```solidity
file:  src/core/SafeEnabler.sol

34     _self = address(this);

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L34

## [G-06] Gas saving is achieved by removing the delete keyword (~60k)

30k gas savings were made by removing the delete keyword. The reason for using the delete keyword here is to reset the struct values (set to default value) in every operation. However, the struct values do not need to be zero each time the function is run. Therefore, the delete keyword is unnecessary. If it is removed, around 30k gas savings will be achieved.

```solidity
file:  src/core/AddressProvider.sol

68     delete pendingGovernance;

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L68

## [G-07] Make 3 event parameters indexed when possible 

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


## [G-08] Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}}).

```solidity
file:  src/core/ConsoleFallbackHandler.sol

29   constructor(address _addressProvider) AddressProviderService(_addressProvider) {}

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L29

```solidity
file:  src/core/ExecutorPlugin.sol

60     constructor(address _addressProvider) AddressProviderService(_addressProvider) {}

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L60

```solidity
file: src/core/PolicyValidator.sol

30   constructor(address _addressProvider) AddressProviderService(_addressProvider) {}

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L30

```solidity
file: src/core/SafeDeployer.sol

42    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L42

```solidity
file: src/core/SafeModerator.sol

17    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}

80        function checkModuleTransaction(
        address, /* to */
        uint256, /* value */
        bytes memory, /* data */
        Enum.Operation, /* operation */
        address /* module */
    ) external override {}

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModerator.sol#L17

```solidity
file:   src/core/SafeModeratorOverridable.sol

23    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}

86    function checkModuleTransaction(
        address, /* to */
        uint256, /* value */
        bytes memory, /* data */
        Enum.Operation, /* operation */
        address /* module */
    ) external override {}

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol#L23

```solidity
file: src/core/TransactionValidator.sol

54    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}

81   function validatePostTransactionOverridable(bytes32, /*txHash */ bool, /*success */ address /*console */ )
        external
        view
    {}

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L54

```solidity
file:  src/core/registries/ExecutorRegistry.sol
  
29    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol#L29

```solidity
file:  src/core/registries/PolicyRegistry.sol

24     constructor(address _addressProvider) AddressProviderService(_addressProvider) {}

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L24

```solidity
file:  src/core/registries/WalletRegistry.sol

29    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L29

## [G-09] Using fixed bytes is cheaper than using string 


```solidity
file: src/core/ExecutorPlugin.sol

53    string private constant _NAME = "ExecutorPlugin";

55    string private constant _VERSION = "1.0";

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L53

```solidity
file:  src/core/PolicyValidator.sol

26     string private constant _NAME = "PolicyValidator";

28      string private constant _VERSION = "1.0";

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L26

```solidity
file:  src/core/SafeDeployer.sol

23     string public constant VERSION = "1";

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L23

## [G-10] Do not initialize variables with default values


```solidity
file:  src/core/SafeModeratorOverridable.sol

21     uint8 public constant DIFFER_SAFE_MOD = 0;

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol#L21

```solidity
file:  src/libraries/SafeHelper.sol

107     uint256 i = 0;

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L107

## [G-11] More likely to revert operations should be executed first

```solidity
file:   src/core/ExecutorPlugin.sol

117     if (execRequest.executor.code.length == 0 && execRequest.executorSignature.length == 0) {
            // Executor is an EOA and no executor signature is provided
            revert InvalidSignature();
        }

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L117-L120

## [G-12] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

```solidity
file:  src/core/PolicyValidator.sol

106    bytes32 policyHash =
            PolicyRegistry(AddressProviderService._getRegistry(_POLICY_REGISTRY_HASH)).commitments(account);

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L106-L107

```solidity
file:  src/libraries/SafeHelper.sol

143     bytes memory guardAddress = IGnosisSafe(safe).getStorageAt(_GUARD_STORAGE_SLOT, 1);

153    bytes memory fallbackHandlerAddress = IGnosisSafe(safe).getStorageAt(_FALLBACK_HANDLER_STORAGE_SLOT, 1);

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L143

## [G-13] Use assembly to perform efficient back-to-back calls

If similar external calls are performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer), which can potentially allow us to avoid memory expansion costs. In this case, we are also able to efficiently store the function signatures together in memory as one word, saving multiple MLOADs in the process.

Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.

### Use for this AddressProviderService._getAuthorizedAddress() back to back external call assembly

```solidity
file:   src/core/SafeDeployer.sol

120      fallbackHandler = AddressProviderService._getAuthorizedAddress(_CONSOLE_FALLBACK_HANDLER_HASH);

            // Enable guard on console account
            txns[1] = Types.Executable({
                callType: Types.CallType.DELEGATECALL,
                target: AddressProviderService._getAuthorizedAddress(_SAFE_ENABLER_HASH),
                value: 0,
                data: abi.encodeCall(
                    IGnosisSafe.setGuard, (AddressProviderService._getAuthorizedAddress(_SAFE_MODERATOR_OVERRIDABLE_HASH))
                    )
            });
        } else {
            txns = new Types.Executable[](1);
            fallbackHandler = AddressProviderService._getAuthorizedAddress(_GNOSIS_FALLBACK_HANDLER_HASH);
        }


223    address gnosisProxyFactory = AddressProviderService._getAuthorizedAddress(_GNOSIS_PROXY_FACTORY_HASH);
        address gnosisSafeSingleton = AddressProviderService._getAuthorizedAddress(_GNOSIS_SINGLETON_HASH);

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L120-L134

## [G-14] Access mappings directly rather than using accessor functions

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

## [G-15] public functions not called by the contract should be declared external instead

when a function is declared as public, it is generated with an internal and an external interface. This means the function can be called both internally (within the contract) and externally (by other contracts or accounts). However, if a public function is never called internally and is only expected to be invoked externally, it is more gas-efficient to explicitly declare it as external.

```solidity
file:  src/core/ConsoleFallbackHandler.sol

60    function getMessageHash(bytes memory message) public view returns (bytes32) {

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L60

```solidity
file: src/core/SafeEnabler.sol

43    function enableModule(address module) public {

66    function setGuard(address guard) public {

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L43