# Gas Optimization

# summmary
|  no    |  issue   | instance  |
|------|----------|-----------|
|[G‑01]|bytes constants are more eficient than string constans|5|
|[G‑02]|Access mappings directly rather than using accessor functions|3|
|[G‑03]|Avoid contract existence checks by using low level calls|20|
|[G‑04]|Expressions for constant values such as a call to keccak256(), should use immutable rather than constant|1|
|[G‑05]|When possible, use assembly instead of unchecked{++i}|1|
|[G‑06]|Empty blocks should be removed or emit something|3|
|[G‑07]|Use of emit inside a loopEmitting |1|
|[G‑08]|Delete variables that you don’t need|1|
|[G‑09]|Stack variable used as a cheaper cache for a state variable is only used once|1|
|[G‑10]| abi.encode() is less efficient than abi.encodePacked()|1|
|[G‑11]|Make 3 event parameters indexed when possible|3|
|[G‑12]|Avoid updating storage when the value hasn't changed|1|
|[G‑13]|Use assembly to write address storage values|2|
|[G‑14]|Use Modifiers Instead of Functions To Save Gas|2|



## [G-01] bytes constants are more eficient than string constans


If the data can fit in 32 bytes, the bytes32 data type can be used instead of bytes or strings, as it is less robust in terms of robustness.

```solidity
53   string private constant _NAME = "ExecutorPlugin";

55   string private constant _VERSION = "1.0";
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L53
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L55



```solidity
26   string private constant _NAME = "PolicyValidator";

28   string private constant _VERSION = "1.0";
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L26
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L28



```solidity
23   string public constant VERSION = "1";
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L23


## [G-02] Access mappings directly rather than using accessor functions
When you access a mapping directly, the Solidity compiler generates code that performs a direct lookup in the mapping's storage slot. This can be more efficient in terms of gas usage compared to using accessor functions, which involve additional function call overhead.


```solidity
113   return authorizedAddresses[_key];

122   return registries[_key];
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L113
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L122


```solidity
64  return walletToSubAccountList[_wallet];
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L64



## [G‑03] Avoid contract existence checks by using low level calls

When you use a low-level call, you directly specify the target contract's address and the function signature or data to be executed. This bypasses some of the checks and overhead associated with regular function calls, resulting in potential gas savings. However, it's important to note that low-level calls come with their own considerations and risks, so they should be used judiciously.

```solidity
File:  contracts/src/core/AddressProvider.sol
131   if (IAddressProviderService(_newAddress).addressProviderTarget() != address(this)) {
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L131


```solidity
File:  contracts/src/core/ExecutorPlugin.sol
73       TransactionValidator(AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH))
              .validatePostExecutorTransaction(msg.sender, execRequest.account);

90       (bool success, bytes memory txnResult) = IGnosisSafe(_account).execTransactionFromModuleReturnData(
            _executable.target,
            _executable.value,
            _executable.data,
            SafeHelper._parseOperationEnum(_executable.callType)
        );

148      TransactionValidator(AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH))
            .validatePreExecutorTransaction(
            msg.sender, execRequest.account, _transactionStructHash, execRequest.validatorSignature
        );                
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L73
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L90
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L148



```solidity
File:  contracts/src/core/PolicyValidator.sol
63   uint256 nonce = IGnosisSafe(account).nonce();

106     bytes32 policyHash =
            PolicyRegistry(AddressProviderService._getRegistry(_POLICY_REGISTRY_HASH)).commitments(account);
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L63
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L106



```solidity
File:  contracts/src/core/SafeDeployer.sol
66           PolicyRegistry(AddressProviderService._getRegistry(_POLICY_REGISTRY_HASH)).updatePolicy(
                _safe, _policyCommit
            );

101   PolicyRegistry(AddressProviderService._getRegistry(_POLICY_REGISTRY_HASH)).updatePolicy(_subAcc, _policyCommit);

230   try IGnosisProxyFactory(gnosisProxyFactory).createProxyWithNonce(gnosisSafeSingleton, _initializer, nonce)
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L66
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L101
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L230



```solidity
File:  contracts/src/core/SafeModerator.sol
46    TransactionValidator(AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH))

71    TransactionValidator(AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH))
         .validatePostTransaction(txHash, success, msg.sender);
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModerator.sol#L46
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModerator.sol#L71



```solidity
File:  contracts/src/core/SafeModeratorOverridable.sol
52    TransactionValidator(AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH))

77    TransactionValidator(AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH))
            .validatePostTransactionOverridable(txHash, success, msg.sender);
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol#L52
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol#L77


```solidity
File:  contracts/src/core/TransactionValidator.sol
194   address ownerConsole =
            WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH)).subAccountToWallet(_subAccount);

107  if (!IGnosisSafe(_subAccount).isModuleEnabled(ownerConsole)) revert InvalidModule();

219  !PolicyValidator(AddressProviderService._getAuthorizedAddress(_POLICY_VALIDATOR_HASH)).isPolicySignatureValid(

239  !PolicyValidator(AddressProviderService._getAuthorizedAddress(_POLICY_VALIDATOR_HASH)).isPolicySignatureValid(    
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L194
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L107
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L219
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L239



```solidity
File:  contracts/src/libraries/SafeHelper.sol
64   bool success = IGnosisSafe(safe).execTransaction(

143  bytes memory guardAddress = IGnosisSafe(safe).getStorageAt(_GUARD_STORAGE_SLOT, 1);

153  bytes memory fallbackHandlerAddress = IGnosisSafe(safe).getStorageAt(_FALLBACK_HANDLER_STORAGE_SLOT, 1);
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L64
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L143
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L153





## [G-04] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

The reason for this is that constant variables are evaluated at runtime and their value is included in the bytecode of the contract. This means that any expensive operations performed as part of the constant expression, such as a call to keccak256(), will be executed every time the contract is deployed, even if the result is always the same. This can result in higher gas costs.


```

```solidity
24   bytes4 internal constant SIMULATE_SELECTOR = bytes4(keccak256("simulate(address,bytes)"));
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L24


## [G-05] When possible, use assembly instead of unchecked{++i}

You can also use unchecked{++i;} for even more gas savings but this will not check to see if i overflows. For best gas savings, use inline assembly, however, this limits the functionality you can achieve.



```solidity
File:  contracts/src/libraries/SafeHelper.sol
131         unchecked {
                ++i;
            }
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L131-L133


## [G‑06] Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.



```solidity
File:  contracts/src/core/SafeModerator.sol
80  function checkModuleTransaction(
        address, /* to */
        uint256, /* value */
        bytes memory, /* data */
        Enum.Operation, /* operation */
        address /* module */
    ) external override {}
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModerator.sol#L80-L86


```solidity
File:  contracts/src/core/SafeModeratorOverridable.sol
86  function checkModuleTransaction(
        address, /* to */
        uint256, /* value */
        bytes memory, /* data */
        Enum.Operation, /* operation */
        address /* module */
    ) external override {}
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol#L86-L92


```solidity
File:  contracts/src/core/TransactionValidator.sol
81   function validatePostTransactionOverridable(bytes32, /*txHash */ bool, /*success */ address /*console */ )
        external
        view
    {}
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L81-L84


## [G-07] Use of emit inside a loopEmitting 

events can consume a significant amount of gas, especially if the loop has a large number of iterations.

```solidity
239   emit SafeProxyCreationFailure(gnosisSafeSingleton, nonce, _initializer);
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L239





## [G-08] Delete variables that you don’t need


Deleting a variable refund 15,000 gas up to a maximum of half the gas cost of the transaction. Deleting with the delete keyword is equivalent to assigning the initial value for the data type, such as 0 for integers.

```solidity
File:  contracts/src/core/AddressProvider.sol
68    delete pendingGovernance;
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L68




## [G‑09] Stack variable used as a cheaper cache for a state variable is only used once

 it's cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend

```solidity
42    bytes32 currentCommit = commitments[account];
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L42




## [G-10] abi.encode() is less efficient than abi.encodePacked()

Changing abi.encode function to abi.encodePacked can save gas since the abi.encode function pads extra null bytes at the end of the call data, which is unnecessary. Also, in general, abi.encodePacked is more gas-efficient (see [Solidity-Encode-Gas-Comparison](https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison)).

```solidity
File:  contracts/src/core/ConsoleFallbackHandler.sol
69     bytes32 safeMessageHash = keccak256(abi.encode(SAFE_MSG_TYPEHASH, keccak256(message)));
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L69





## [G-11] Make 3 event parameters indexed when possible

It is the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

```solidity
File: contracts/src/core/SafeEnabler.sol
19   event EnabledModule(address module);

20   event ChangedGuard(address guard);
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L19-L19
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L19-L20


```solidity
File:  contracts/src/core/registries/PolicyRegistry.sol
19    event UpdatedPolicyCommit(address indexed account, bytes32 policyCommit, bytes32 oldPolicyCommit);
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L19


## [G‑12] Avoid updating storage when the value hasn't changed


`Note`: missd from bots

There are 1 instance of this issue:

```solidity
File:   contracts/src/core/registries/WalletRegistry.sol
52   subAccountToWallet[_subAccount] = _wallet;
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L52



## [G-13] Use assembly to write address storage values

By using assembly to write to address storage values, you can bypass some of these operations and lower the gas cost of writing to storage. Assembly code allows you to directly access the Ethereum Virtual Machine (EVM) and perform low-level operations that are not possible in Solidity.

```solidity
File:  contracts/src/core/AddressProvider.sol
45    governance = _governance;

56    pendingGovernance = _newGovernance;
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L45
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L56


## [G-14] Use Modifiers Instead of Functions To Save Gas

Using modifiers instead of functions can be a gas-saving technique in Solidity, especially when you have repetitive logic that needs to be executed before or after multiple functions.



```solidity
139  function _onlyGov() internal view {
        if (msg.sender != governance) revert NotGovernance(msg.sender);
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L139


```solidity
81   function _onlyDelegateCall() private view {
        if (address(this) == _self) revert OnlyDelegateCall();
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L81-L82
