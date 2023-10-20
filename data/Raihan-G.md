# gas optimization

# summmary
|      |  issue   | instance  |
|------|----------|-----------|
|[G‑01]|Avoid contract existence checks by using low level calls|20|
|[G‑02]|Cache external calls outside of loop to avoid re-calling function on each iteration|1|
|[G‑03]|Use assembly to write address storage values|2|
|[G‑04]|Use Modifiers Instead of Functions To Save Gas|2|
|[G‑05]|Delete variables that you don’t need|1|
|[G‑06]|bytes constants are more eficient than string constans|6|
|[G‑07]|Stack variable used as a cheaper cache for a state variable is only used once|1|
|[G‑08]|Access mappings directly rather than using accessor functions|3|
|[G‑09]|abi.encode() is less efficient than abi.encodePacked()|9|
|[G‑10]|Expressions for constant values such as a call to keccak256(), should use immutable rather than constant|1|
|[G‑11]|When possible, use assembly instead of unchecked{++i}|1|
|[G‑12]|Empty blocks should be removed or emit something|3|
|[G‑13]|Use of emit inside a loop|1|
|[G‑14]|Make 3 event parameters indexed when possible|3|
|[G‑15]|Avoid updating storage when the value hasn't changed|1|





## [G‑01] Avoid contract existence checks by using low level calls
Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

There are 20 instances of this issue:
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


```solidity
File:  contracts/src/core/PolicyValidator.sol
63   uint256 nonce = IGnosisSafe(account).nonce();

106     bytes32 policyHash =
            PolicyRegistry(AddressProviderService._getRegistry(_POLICY_REGISTRY_HASH)).commitments(account);
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L63


```solidity
File:  contracts/src/core/SafeDeployer.sol
66           PolicyRegistry(AddressProviderService._getRegistry(_POLICY_REGISTRY_HASH)).updatePolicy(
                _safe, _policyCommit
            );

101   PolicyRegistry(AddressProviderService._getRegistry(_POLICY_REGISTRY_HASH)).updatePolicy(_subAcc, _policyCommit);

230   try IGnosisProxyFactory(gnosisProxyFactory).createProxyWithNonce(gnosisSafeSingleton, _initializer, nonce)
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L66


```solidity
File:  contracts/src/core/SafeModerator.sol
46    TransactionValidator(AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH))

71    TransactionValidator(AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH))
         .validatePostTransaction(txHash, success, msg.sender);
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModerator.sol#L46


```solidity
File:  contracts/src/core/SafeModeratorOverridable.sol
52    TransactionValidator(AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH))

77    TransactionValidator(AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH))
            .validatePostTransactionOverridable(txHash, success, msg.sender);
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol#L52


```solidity
File:  contracts/src/core/TransactionValidator.sol
194   address ownerConsole =
            WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH)).subAccountToWallet(_subAccount);

107  if (!IGnosisSafe(_subAccount).isModuleEnabled(ownerConsole)) revert InvalidModule();

219  !PolicyValidator(AddressProviderService._getAuthorizedAddress(_POLICY_VALIDATOR_HASH)).isPolicySignatureValid(

239  !PolicyValidator(AddressProviderService._getAuthorizedAddress(_POLICY_VALIDATOR_HASH)).isPolicySignatureValid(    
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L194


```solidity
File:  contracts/src/libraries/SafeHelper.sol
64   bool success = IGnosisSafe(safe).execTransaction(

143  bytes memory guardAddress = IGnosisSafe(safe).getStorageAt(_GUARD_STORAGE_SLOT, 1);

153  bytes memory fallbackHandlerAddress = IGnosisSafe(safe).getStorageAt(_FALLBACK_HANDLER_STORAGE_SLOT, 1);
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L64


## [G-02] Cache external calls outside of loop to avoid re-calling function on each iteration

```solidity
File:  contracts/src/core/SafeDeployer.sol
// @audit IGnosisProxyFactory is external
230   try IGnosisProxyFactory(gnosisProxyFactory).createProxyWithNonce(gnosisSafeSingleton, _initializer, nonce)
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L230


## [G-03] Use assembly to write address storage values
By using assembly to write to address storage values, you can bypass some of these operations and lower the gas cost of writing to storage. Assembly code allows you to directly access the Ethereum Virtual Machine (EVM) and perform low-level operations that are not possible in Solidity.

example of using assembly to write to address storage values:
```
contract MyContract {
    address private myAddress;
    
    function setAddressUsingAssembly(address newAddress) public {
        assembly {
            sstore(0, newAddress)
        }
    }
}
```

There are 2 instances of this issue:
```solidity
File:  contracts/src/core/AddressProvider.sol
45    governance = _governance;

56    pendingGovernance = _newGovernance;
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L45

## [G-04] Use Modifiers Instead of Functions To Save Gas
Example of two contracts with modifiers and internal view function:
```
// SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
contract Inlined {
    function isNotExpired(bool _true) internal view {
        require(_true == true, "Exchange: EXPIRED");
    }
function foo(bool _test) public returns(uint){
            isNotExpired(_test);
            return 1;
    }
}
// SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
contract Modifier {
modifier isNotExpired(bool _true) {
        require(_true == true, "Exchange: EXPIRED");
        _;
    }
function foo(bool _test) public isNotExpired(_test)returns(uint){
        return 1;
    }
}
```
Differences:
```
Deploy Modifier.sol
108727
Deploy Inlined.sol
110473
Modifier.foo
21532
Inlined.foo
21556
```
This with 0.8.9 compiler and optimization enabled. As you can see it's cheaper to deploy with a modifier, and it will save you about 30 gas. But sometimes modifiers increase code size of the contract.
There are 1 instances of this issue:


```solidity
File:  contracts/src/core/AddressProvider.sol
139  function _onlyGov() internal view {
        if (msg.sender != governance) revert NotGovernance(msg.sender);
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L139


```solidity
File:  contracts/src/core/SafeEnabler.sol
81   function _onlyDelegateCall() private view {
        if (address(this) == _self) revert OnlyDelegateCall();
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L81-L82


## [G-05] Delete variables that you don’t need
In Ethereum, you get a gas refund for freeing up storage space.
Deleting a variable refund 15,000 gas up to a maximum of half the gas cost of the transaction. Deleting with the delete keyword is equivalent to assigning the initial value for the data type, such as 0 for integers.

There are 1 instances of this issue:
```solidity
File:  contracts/src/core/AddressProvider.sol
68    delete pendingGovernance;
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L68


## [G-06] bytes constants are more eficient than string constans
If the data can fit in 32 bytes, the bytes32 data type can be used instead of bytes or strings, as it is less robust in terms of robustness.

```solidity
File:  contracts/src/core/ExecutorPlugin.sol
53   string private constant _NAME = "ExecutorPlugin";

55   string private constant _VERSION = "1.0";
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L53


```solidity
File:   contracts/src/core/PolicyValidator.sol
26   string private constant _NAME = "PolicyValidator";

28   string private constant _VERSION = "1.0";
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L26


```solidity
File:  contracts/src/core/SafeDeployer.sol
23   string public constant VERSION = "1";
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L23

## [G‑07] Stack variable used as a cheaper cache for a state variable is only used once
If the variable is only accessed once, it's cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend

```solidity
File:  contracts/src/core/registries/PolicyRegistry.sol
// @audit currentCommit variable only use in line number 46
42    bytes32 currentCommit = commitments[account];
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L42


## [G-08] Access mappings directly rather than using accessor functions
Saves having to do two JUMP instructions, along with stack setup

There are 3 instances of this issue:
```solidity
File:  contracts/src/core/AddressProvider.sol
113   return authorizedAddresses[_key];

122   return registries[_key];
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L113


```solidity
File:  contracts/src/core/registries/WalletRegistry.sol
64  return walletToSubAccountList[_wallet];
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L64

## [G-09] abi.encode() is less efficient than abi.encodePacked()
Changing abi.encode function to abi.encodePacked can save gas since the abi.encode function pads extra null bytes at the end of the call data, which is unnecessary. Also, in general, abi.encodePacked is more gas-efficient (see [Solidity-Encode-Gas-Comparison](https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison)).

There are 1 instances of this issue:
```solidity
File:  contracts/src/core/ConsoleFallbackHandler.sol
//  @audit there is no chance of colesion if change to abi.encodePacked()
69     bytes32 safeMessageHash = keccak256(abi.encode(SAFE_MSG_TYPEHASH, keccak256(message)));
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L69


## [G-10] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant
The reason for this is that constant variables are evaluated at runtime and their value is included in the bytecode of the contract. This means that any expensive operations performed as part of the constant expression, such as a call to keccak256(), will be executed every time the contract is deployed, even if the result is always the same. This can result in higher gas costs.

In contrast, immutable variables are evaluated at compilation time, and their values are included in the bytecode of the contract as constants. This means that any expensive operations performed as part of the immutable expression are only executed once, when the contract is compiled, and the result is reused every time the contract is deployed. This can result in lower gas costs compared to using constant variables.

Let's consider an example to illustrate this. Suppose we want to store the hash of a string as a constant value in our contract. We could do this using a constant variable, like so:

```
including the code for that function in the contract bytecode is unnecessary and can lead to higher deployment gas costs.

By removing unused internal functions, you can reduce the size of your contract bytecode and lower the overall deployment gas cost of your contract.

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword
bytes32 constant MY_HASH = keccak256("my string");
```
Alternatively, we could use an immutable variable, like so:
```
bytes32 immutable MY_HASH = keccak256("my string");
```
There are 1 instances of this issue:

```solidity
File:  contracts/src/core/ConsoleFallbackHandler.sol
24   bytes4 internal constant SIMULATE_SELECTOR = bytes4(keccak256("simulate(address,bytes)"));
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L24


## [G-11] When possible, use assembly instead of unchecked{++i}
You can also use unchecked{++i;} for even more gas savings but this will not check to see if i overflows. For best gas savings, use inline assembly, however, this limits the functionality you can achieve.

```
//loop with unchecked{++i}
function uncheckedPlusPlusI() public pure {
    uint256 j = 0;
    for (uint256 i; i < 10; ) {
        j++;
        unchecked {
            ++i;
        }
    }
}
```
Gas: 1329
```
//loop with inline assembly
function inlineAssemblyLoop() public pure {
    assembly {
        let j := 0
        for {
            let i := 0
        } lt(i, 10) {
            i := add(i, 0x01)
        } {
            j := add(j, 0x01)
        }
    }
}
```
Gas: 709


```solidity
File:  contracts/src/libraries/SafeHelper.sol
131         unchecked {
                ++i;
            }
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L131-L133


## [G‑12] Empty blocks should be removed or emit something
The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.
If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation.


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


## [G-13] Use of emit inside a loop
Emitting an event inside a loop performs a LOG op N times, where N is the loop length. Consider refactoring the code to emit the event only once at the end of loop. Gas savings should be multiplied by the average loop length.

```solidity
File:  contracts/src/core/SafeDeployer.sol
239   emit SafeProxyCreationFailure(gnosisSafeSingleton, nonce, _initializer);
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L239


## [G-14] Make 3 event parameters indexed when possible
It is the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

There are 3 instances of this issue:
```solidity
File: contracts/src/core/SafeEnabler.sol
19   event EnabledModule(address module);

20   event ChangedGuard(address guard);
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L19-L20

```solidity
File:  contracts/src/core/registries/PolicyRegistry.sol
19    event UpdatedPolicyCommit(address indexed account, bytes32 policyCommit, bytes32 oldPolicyCommit);
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L19


## [G‑15] Avoid updating storage when the value hasn't changed
If the old value is equal to the new value, not re-storing the value will avoid a Gsreset (2900 gas), potentially at the expense of a Gcoldsload (2100 gas) or a Gwarmaccess (100 gas)

`Note`: this instance is missd from bots

There are 1 instance of this issue:

```solidity
File:   contracts/src/core/registries/WalletRegistry.sol
52   subAccountToWallet[_subAccount] = _wallet;
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L52

