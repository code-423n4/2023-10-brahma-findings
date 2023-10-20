## Gas Optimizations

## Summary

|No |Issue|Instances|
|-|:-|:-:
|[G-01]|Use hardcode address instead address(this)|3|
|[G-02]|Use assembly to check for address(0)|13|
|[G-03]|abi.encode() is less efficient than abi.encodePacked()|5|
|[G-04]|Checking msg.sender to not be zero address is redundant|2
|[G-05]|Gas saving is achieved by removing the delete keyword (~60k)|1|
|[G-06]|Make 3 event parameters indexed when possible|2|
|[G-07]|Loop best practice to save gas|1|
|[G-08]|Avoid emitting constants.|14
|[G-09]|Use selfbalance() instead of address(this).balance|4|
|[G-10]|Pre-increments and pre-decrements are cheaper than post-increments and post-decrements|1|
|[G-11]|Using storage instead of memory for structs/arrays saves gas|16

## [G-1]  Use hardcode address instead address(this)
Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.

```solidity

file: ibraries/SafeHelper.sol

74  _generateSingleThresholdSignature(address(this))
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L74

```solidity

file: ore/SafeEnabler.sol 

33   _self = address(this);

82   if (address(this) == _self) revert OnlyDelegateCall();
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L33
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L82

```solidity

file: core/AddressProvider.sol

131  if (IAddressProviderService(_newAddress).addressProviderTarget() != address(this)) 
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L131

## [G-2] Use assembly to check for address(0)
Saves 6 gas per instance

```solidity

file: Libraries/SafeHelper.sol

72 address(0), // gasToken
73            payable(address(0))
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L72-L173

```solidity

file: core/SafeEnabler.sol

48  require(module != address(0) && module != _SENTINEL_MODULES, "GS101");

52 require(modules[module] == address(0), "GS102");
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L48
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L52

```solidity

file: core/AddressProvider.sol

101 if (registries[_key] != address(0)) revert RegistryAlreadyExists();

148  if (addr == address(0)) revert NullAddress();
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L101
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L148

```solidity

file: core/PolicyValidator.sol

73   executor: address(0),
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L73

```solidity

file: core/registries/WalletRegistry.sol

37   if (subAccountToWallet[msg.sender] != address(0)) revert IsSubAccount();

51  if (subAccountToWallet[_subAccount] != address(0)) revert AlreadyRegistered();
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L37
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L51

```solidity

file: core/AddressProviderService.sol

28  if (_addressProvider == address(0)) revert InvalidAddressProvider();

73  if (_addr == address(0)) revert InvalidAddress();
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol#L28
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol#L73

````solidity

file: core/SafeDeployer.sol

155     address(0),
                0,
157                address(0)

200    address(0),
                0,
202                address(0)

244 while (_safe == address(0));
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L155-L157
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L200-L202
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L244

## [G-3]  abi.encode() is less efficient than abi.encodePacked()
Consider changing it if possible.

```solidity

file: libraries/TypeHashHelper.sol

66 abi.encode(

86 abi.encode(    
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/TypeHashHelper.sol#L66
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/TypeHashHelper.sol#L86

```solidity

file: core/ConsoleFallbackHandler.sol

69   bytes32 safeMessageHash = keccak256(abi.encode(SAFE_MSG_TYPEHASH, keccak256(message)));

85  bytes4 value = validator.isValidSignature(abi.encode(_dataHash), _signature);
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L69
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L85

```solidity

file: core/SafeDeployer.sol

225  bytes32 ownersHash = keccak256(abi.encode(_owners));
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L225

## [G-4] Checking msg.sender to not be zero address is redundant
There is an instance where msg.sender is checked not to be zero address. This check is redundant as no private key is known for this address, hence there can be no transactions coming from the zero address. 

```solidity

file: core/registries/WalletRegistry.sol

37  if (subAccountToWallet[msg.sender] != address(0)) revert IsSubAccount();
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L37

## [G-5] Gas saving is achieved by removing the delete keyword (~60k)
30k gas savings were made by removing the delete keyword. The reason for using the delete keyword here is to reset the struct values (set to default value) in every operation. However, the struct values do not need to be zero each time the function is run. Therefore, the delete keyword is unnecessary. If it is removed, around 30k gas savings will be achieved.

```solidity

file: core/AddressProvider.sol

148  delete pendingGovernance;

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L148

## [G-6] Make 3 event parameters indexed when possible
It’s the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

```solidity

file: core/AddressProvider.sol

21  event RegistryInitialised(address indexed registry, bytes32 indexed key);
    event AuthorizedAddressInitialised(address indexed authorizedAddress, bytes32 indexed key);
    event GovernanceTransferRequested(address indexed previousGovernance, address indexed newGovernance);
24  event GovernanceTransferred(address indexed previousGovernance, address indexed newGovernance);
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L21-L24

```solidity

file: core/SafeDeployer.sol

32 event SafeProxyCreationFailure(address indexed singleton, uint256 indexed nonce, bytes initializer);
    event ConsoleAccountDeployed(address indexed consoleAddress);
    event SubAccountDeployed(address indexed subAccountAddress, address indexed consoleAddress);
35  event PreComputeAccount(address[] indexed owners, uint256 indexed threshold);
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L32-L35

## [G-7] Loop best practice to save gas

```solidity

file: libraries/SafeHelper.sol

131    unchecked {
                ++i;
            }
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L131

## [G-8] Avoid emitting constants.
A log topic (declared with indexed) has a gas cost of Glogtopic (375 gas).

```solidity

file: core/SafeEnabler.sol

55  emit EnabledModule(module);

74  emit ChangedGuard(guard);
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L55
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L74

```solidity

file: core/AddressProvider.sol

55 emit GovernanceTransferRequested(governance, _newGovernance);

66  emit GovernanceTransferred(governance, msg.sender);

89   emit AuthorizedAddressInitialised(_authorizedAddress, _key);

104 emit RegistryInitialised(_registry, _key);
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L55
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L66
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L89
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L104

```solidity

file: core/registries/PolicyRegistry.sol

67  emit UpdatedPolicyCommit(account, policyCommit, commitments[account]);
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L67

```solidity

file: core/registries/ExecutorRegistry.sol

43 emit RegisterExecutor(_subAccount, msg.sender, _executor);

58  emit DeRegisterExecutor(_subAccount, msg.sender, _executor);
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol#L43
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol#L58

```solidity

file: core/registries/WalletRegistry.sol

39   emit RegisterWallet(msg.sender);

54  emit RegisterSubAccount(_wallet, _subAccount);
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L39
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L54

```solidity

file: core/SafeDeployer.sol

70  emit ConsoleAccountDeployed(_safe);

102    emit SubAccountDeployed(_subAcc, msg.sender);

239   emit SafeProxyCreationFailure(gnosisSafeSingleton, nonce, _initializer);
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L70
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L102
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L239

## [G-9]  Use selfbalance() instead of address(this).balance

```solidity

file: libraries/SafeHelper.sol

74 _generateSingleThresholdSignature(address(this))
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L74



```solidity

file: core/AddressProvider.sol

131    if (IAddressProviderService(_newAddress).addressProviderTarget() != address(this)) 

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L131

```solidity

file: ore/SafeEnabler.sol 

33   _self = address(this);

82   if (address(this) == _self) revert OnlyDelegateCall();
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L33
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L82

## [G-10] Pre-increments and pre-decrements are cheaper than post-increments and post-decrements

Saves 5 gas per iteration

```solidity

file: libraries/SafeHelper.sol

131    unchecked {
                ++i;
            }
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L131

## [G-11] Using storage instead of memory for structs/arrays saves gas

```solidity

file: https:/libraries/TypeHashHelper.sol#L64

64 function _buildTransactionStructHash(Transaction memory txn) internal pure returns (bytes32)

84 function _buildValidationStructHash(Validation memory validation) internal pure returns (bytes32) 
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/TypeHashHelper.sol#L64
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/TypeHashHelper.sol#L84


```solidity

file: libraries/SafeHelper.sol

63 function _executeOnSafe(address safe, address target, Enum.Operation op, bytes memory data) internal 

87 function _generateSingleThresholdSignature(address owner) internal pure returns (bytes memory) {
        bytes memory signatures = abi.encodePacked(

103   function _packMultisendTxns(Types.Executable[] memory _txns) internal pure returns (bytes memory packedTxns) 
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L63
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L87
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L103

```solidity

file: core/TransactionValidator.sol

64  function validatePreTransactionOverridable(SafeTransactionParams memory txParams) external view 

95  function validatePreTransaction(SafeTransactionParams memory txParams) external view

123  bytes memory signatures

153 bytes memory _data,

214 bytes memory _data,
        
216 bytes memory _signatures

234  function _validatePolicySignature(address _from, bytes32 _transactionStructHash, bytes memory _signatures)
        internal
        view
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L64
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L95
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L123
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L153
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L214-L216
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L234

```solidity

file: core/SafeModeratorOverridable.sol

39  function checkTransaction(
        address to,
        uint256 value,
        bytes memory data,
        Enum.Operation operation,
        uint256 safeTxGas,
        uint256 baseGas,
        uint256 gasPrice,
        address gasToken,
        address payable refundReceiver,
        bytes memory signatures,
        address msgSender
52    )

86  function checkModuleTransaction(
        address, /* to */
        uint256, /* value */
        bytes memory, /* data */
        Enum.Operation, /* operation */
        address /* module */
    )

```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol#L39
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol#L86

```solidity

file: core/ConsoleFallbackHandler.sol

39 function isValidSignature(bytes memory _data, bytes memory _signature) public view override returns (bytes4) 

60 function getMessageHash(bytes memory message) public view returns (bytes32) 

68   function getMessageHash(bytes memory message) public view returns (bytes32)
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L39
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L60
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L68