# Gas Report

- [Gas Report](#gas-report)
  - [Checks can be done more efficiently](#checks-can-be-done-more-efficiently)
    - [Save ~2100 Gas in case of early reverts](#save-2100-gas-in-case-of-early-reverts)
  - [We can optimize this function by executing cheap checks first(save 1514 Gas on average)](#we-can-optimize-this-function-by-executing-cheap-checks-firstsave-1514-gas-on-average)
  - [Avoid making 2 redundant external calls here(save 543 Gas on average)](#avoid-making-2-redundant-external-calls-heresave-543-gas-on-average)
  - [Optimize the function `updatePolicy()`](#optimize-the-function-updatepolicy)
    - [An alternative optimization to the above finding](#an-alternative-optimization-to-the-above-finding)
  - [We can optimize the validity checks to save 1 SLOAD](#we-can-optimize-the-validity-checks-to-save-1-sload)
  - [Making external calls is more expensive than internal function calls(reorder the checks to have the cheap one come first)](#making-external-calls-is-more-expensive-than-internal-function-callsreorder-the-checks-to-have-the-cheap-one-come-first)
  - [Reordering this operations can save us gas in case of a revert on the require check](#reordering-this-operations-can-save-us-gas-in-case-of-a-revert-on-the-require-check)
  - [Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate](#multiple-address-mappings-can-be-combined-into-a-single-mapping-of-an-address-to-a-struct-where-appropriate)
  - [Do not cache variable/calls used only once](#do-not-cache-variablecalls-used-only-once)
    - [No need to cache the call `ISignatureValidator(msg.sender)`](#no-need-to-cache-the-call-isignaturevalidatormsgsender)
    - [No need to cache the call `GnosisSafe(payable(msg.sender))`](#no-need-to-cache-the-call-gnosissafepayablemsgsender)

## Checks can be done more efficiently

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/AddressProvider.sol#L77-L90

### Save ~2100 Gas in case of early reverts

```solidity
File: /contracts/src/core/AddressProvider.sol
77:    function setAuthorizedAddress(bytes32 _key, address _authorizedAddress, bool _overrideCheck) external {
78:        _onlyGov();
79:        _notNull(_authorizedAddress);
```
The function `setAuthorizedAddress()` is making two function calls ie `_onlyGov()` and `_notNull()` 
The two function have the following implementation respectively

```solidity
139:    function _onlyGov() internal view {
140:        if (msg.sender != governance) revert NotGovernance(msg.sender);
141:    }
```

```solidity
147:    function _notNull(address addr) internal pure {
148:        if (addr == address(0)) revert NullAddress();
149:    }
```
The `_onlyGov()` function is more expensive to execute as it involves making a state read(2100 Gas) as it's cold.
The `_notNull()` is way cheaper as it's only comparing a function parameter against `address(0)`

Since we revert if any of the checks from the two functions do not pass, when calling this functions, we should ensure that the cheaper function is being called first to ensure, in case of a revert we spend as little gas as possible.


```diff
diff --git a/contracts/src/core/AddressProvider.sol b/contracts/src/core/AddressProvider.sol
index 5ec51f9..c2e06b9 100644
--- a/contracts/src/core/AddressProvider.sol
+++ b/contracts/src/core/AddressProvider.sol
@@ -75,8 +75,8 @@ contract AddressProvider {
      * @param _overrideCheck overrides check for supported address provider
      */
     function setAuthorizedAddress(bytes32 _key, address _authorizedAddress, bool _overrideCheck) external {
-        _onlyGov();
         _notNull(_authorizedAddress);
+        _onlyGov();

         /// @dev skips checks for supported `addressProvider()` if `_overrideCh
```

With that if the first check fails, then we wouldn't read any state variable,saving us ~2100 gas compared to the original implementation where we had to make the state read first.

## We can optimize this function by executing cheap checks first(save 1514 Gas on average)

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ExecutorPlugin.sol#L106-L120
`forge test --fork-url $INFURA_API_KEY --gas-report`

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 12935    | 40333   | 13881 | 94185 |
| After  | 8391    | 38819   | 13881 | 94185 |

```solidity
File: /contracts/src/core/ExecutorPlugin.sol
106:    function _validateExecutionRequest(ExecutionRequest calldata execRequest) internal {
107:        // Fetch executor registry
108:        ExecutorRegistry executorRegistry =
109:          ExecutorRegistry(AddressProviderService._getRegistry(_EXECUTOR_REGISTRY_HASH));
110:
111:        // Check if executor is valid for given account
112:        if (!executorRegistry.isExecutor(execRequest.account, execRequest.executor)) {
113:            revert InvalidExecutor();
114:        }

116:        // Empty Signature check for EOA executor
117:        if (execRequest.executor.code.length == 0 && execRequest.executorSignature.length == 0) {
118:            // Executor is an EOA and no executor signature is provided
119:            revert InvalidSignature();
120:        }
```
The second if block is cheaper compared to the first one, we should reorder the checks to minimize the gas spent in case of a revert

```diff
diff --git a/contracts/src/core/ExecutorPlugin.sol b/contracts/src/core/ExecutorPlugin.sol
index a790269..5eab142 100644
--- a/contracts/src/core/ExecutorPlugin.sol
+++ b/contracts/src/core/ExecutorPlugin.sol
@@ -104,6 +104,11 @@ contract ExecutorPlugin is AddressProviderService, ReentrancyGuard, EIP712 {
      * @param execRequest params for execution request
      */
     function _validateExecutionRequest(ExecutionRequest calldata execRequest) internal {
+                // Empty Signature check for EOA executor
+        if (execRequest.executor.code.length == 0 && execRequest.executorSignature.length == 0) {
+            // Executor is an EOA and no executor signature is provided
+            revert InvalidSignature();
+        }
         // Fetch executor registry
         ExecutorRegistry executorRegistry =
             ExecutorRegistry(AddressProviderService._getRegistry(_EXECUTOR_REGISTRY_HASH));
@@ -113,11 +118,6 @@ contract ExecutorPlugin is AddressProviderService, ReentrancyGuard, EIP712 {
             revert InvalidExecutor();
         }

-        // Empty Signature check for EOA executor
-        if (execRequest.executor.code.length == 0 && execRequest.executorSignature.length == 0) {
-            // Executor is an EOA and no executor signature is provided
-            revert InvalidSignature();
-        }

         // Build transaction struct hash
         bytes32 _transactionStructHash = TypeHashHelper._buildTransactionStructHash(
```

To get the gas estimates the test suite was modified to have this function as the only test by renaming the test `testExecuteTransaction_ShouldRevertInvalidExecutorSignature()`  

```solidity
    function testExecuteTransaction() public {
        _setupRegisterExecAndExecPluginModule(executorAddress);
        DigestData memory _txn = DigestData({
            to: address(subSafe),
            txnData: abi.encodeCall(IGnosisSafe.enableModule, makeAddr("module")),
            operation: Enum.Operation.Call,
            value: 0,
            from: address(subSafe),
            nonce: executorPlugin.executorNonce(address(subSafe), executorAddress),
            policyCommit: commit,
            expiryEpoch: uint32(block.timestamp + 1000)
        });


        ExecutorPlugin.ExecutionRequest memory _execReq = ModSignature._buildExecPluginRequest(
            _txn,
            notExecutorPrivKey,
            validatorPrivKey,
            executorAddress,
            address(executorPlugin),
            address(policyValidator)
        );
        vm.expectRevert(abi.encodeWithSelector(ExecutorPlugin.InvalidExecutor.selector));
        executorPlugin.executeTransaction(_execReq);
    }
```

## Avoid making 2 redundant external calls here(save 543 Gas on average)

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ExecutorPlugin.sol#L68-L77
`forge test --fork-url $INFURA_API_KEY --gas-report`

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 11943    | 62659   | 72381 | 117703 |
| After  | 11914    | 62116   | 71399 | 116729 |
```solidity
File: /contracts/src/core/ExecutorPlugin.sol
68:    function executeTransaction(ExecutionRequest calldata execRequest) external nonReentrant returns (bytes memory) {
69:        _validateExecutionRequest(execRequest);
70:
71:        bytes memory txnResult = _executeTxnAsModule(execRequest.account, execRequest.exec);
72:
73:    TransactionValidator(AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH))
74:            .validatePostExecutorTransaction(msg.sender, execRequest.account);
75:
76:        return txnResult;
77:    }
```

The above function makes a call to an internal function `_validateExecutionRequest()` which is implemented as follows
```solidity    
106:function _validateExecutionRequest(ExecutionRequest calldata execRequest) internal {
<-----TRUNCATED------>
148: TransactionValidator(AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH))
149:            .validatePreExecutorTransaction(
150:            msg.sender, execRequest.account, _transactionStructHash, execRequest.validatorSignature
151:        );
152:    }
```
I've truncated the function to only show the relevant part in this context. In this function, of interest to us is [LINE 148](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ExecutorPlugin.sol#L148) where we make an external call `AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH))` 
If you go back to the calling(main) function ie `executeTransaction()` you'll note a similar call is being made on [Line 73](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ExecutorPlugin.sol#L73)
This means, we make this external call twice which is quite expensive especially given that this calls launch a series of other external calls

We can save some gas by in lining the internal function call and caching the results of the external call instead of calling it twice


```diff
diff --git a/contracts/src/core/ExecutorPlugin.sol b/contracts/src/core/ExecutorPlugin.sol
index a790269..027437f 100644
--- a/contracts/src/core/ExecutorPlugin.sol
+++ b/contracts/src/core/ExecutorPlugin.sol
@@ -66,11 +66,56 @@ contract ExecutorPlugin is AddressProviderService, ReentrancyGuard, EIP712 {
      * @return returnData of txn
      */
     function executeTransaction(ExecutionRequest calldata execRequest) external nonReentrant returns (bytes memory) {
-        _validateExecutionRequest(execRequest);
+                // Fetch executor registry
+        ExecutorRegistry executorRegistry =
+            ExecutorRegistry(AddressProviderService._getRegistry(_EXECUTOR_REGISTRY_HASH));
+
+        // Check if executor is valid for given account
+        if (!executorRegistry.isExecutor(execRequest.account, execRequest.executor)) {
+            revert InvalidExecutor();
+        }
+
+        // Empty Signature check for EOA executor
+        if (execRequest.executor.code.length == 0 && execRequest.executorSignature.length == 0) {
+            // Executor is an EOA and no executor signature is provided
+            revert InvalidSignature();
+        }
+
+        // Build transaction struct hash
+        bytes32 _transactionStructHash = TypeHashHelper._buildTransactionStructHash(
+            TypeHashHelper.Transaction({
+                to: execRequest.exec.target,
+                value: execRequest.exec.value,
+                data: execRequest.exec.data,
+                operation: uint8(SafeHelper._parseOperationEnum(execRequest.exec.callType)),
+                account: execRequest.account,
+                executor: execRequest.executor,
+                nonce: executorNonce[execRequest.account][execRequest.executor]++
+            })
+        );
+
+        // Build EIP712 digest for transaction struct hash
+        bytes32 _transactionDigest = _hashTypedData(_transactionStructHash);
+
+        // Validate executor signature
+        if (
+            !SignatureCheckerLib.isValidSignatureNow(
+                execRequest.executor, _transactionDigest, execRequest.executorSignature
+            )
+        ) {
+            revert InvalidExecutor();
+        }
+
+        // Validate policy signature
+        address _authorizedAddress = AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH);
+        TransactionValidator(_authorizedAddress)
+            .validatePreExecutorTransaction(
+            msg.sender, execRequest.account, _transactionStructHash, execRequest.validatorSignature
+        );

         bytes memory txnResult = _executeTxnAsModule(execRequest.account, execRequest.exec);

-        TransactionValidator(AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH))
+        TransactionValidator(_authorizedAddress)
             .validatePostExecutorTransaction(msg.sender, execRequest.account);

         return txnResult;
```

## Optimize the function `updatePolicy()`

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/PolicyRegistry.sol#L42-L59
`forge test --fork-url $INFURA_API_KEY --gas-report`
**we can save 191 Gas on average**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 379    | 26599   | 26550 | 39152 |
| After  | 379    | 26408   | 26360 | 38952 |
```solidity
File: /contracts/src/core/registries/PolicyRegistry.sol
42:        bytes32 currentCommit = commitments[account];

44:        // solhint-disable no-empty-blocks
45:        if (
46:            currentCommit == bytes32(0)
47:                && msg.sender == AddressProviderService._getAuthorizedAddress(_SAFE_DEPLOYER_HASH)
48:        ) {
49:            // In case invoker is safe  deployer
50:        } else if (walletRegistry.isOwner(msg.sender, account)) {
51:            //In case invoker is updating on behalf of sub account
52:        } else if (msg.sender == account && walletRegistry.isWallet(account)) {
53:            // In case invoker is a registered wallet
54:        } else {
55:            revert UnauthorizedPolicyUpdate();
56:        }
57:        // solhint-enable no-empty-blocks
58:        _updatePolicy(account, policyCommit);
59:    }
```

The function above is calling `_updatePolicy(account,policyCommit)` which is implemented as follows

```solidity
66:    function _updatePolicy(address account, bytes32 policyCommit) internal {
67:        emit UpdatedPolicyCommit(account, policyCommit, commitments[account]);
68:        commitments[account] = policyCommit;
69:    }
```

This function is emitting a state variable `commitments[account]` which we can see was cached in our main function as `currentCommit`, see https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/PolicyRegistry.sol#L42

We can avoid one state read by in lining the internal function and making use of the cached value instead as shown below

```diff
diff --git a/contracts/src/core/registries/PolicyRegistry.sol b/contracts/src/core/registries/PolicyRegistry.sol
index 559d5ca..0fe3053 100644
--- a/contracts/src/core/registries/PolicyRegistry.sol
+++ b/contracts/src/core/registries/PolicyRegistry.sol
@@ -55,16 +55,7 @@ contract PolicyRegistry is AddressProviderService {
             revert UnauthorizedPolicyUpdate();
         }
         // solhint-enable no-empty-blocks
-        _updatePolicy(account, policyCommit);
-    }
-
-    /**
-     * @notice Internal function to update policy commit for an account
-     * @param account address of account to set policy commit for
-     * @param policyCommit policy commit hash to set
-     */
-    function _updatePolicy(address account, bytes32 policyCommit) internal {
-        emit UpdatedPolicyCommit(account, policyCommit, commitments[account]);
+        emit UpdatedPolicyCommit(account, policyCommit, currentCommit);
         commitments[account] = policyCommit;
     }
 }
```

### An alternative optimization to the above finding

**If we cannot in line the function as above, we can avoid caching the function as shown below**
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/PolicyRegistry.sol#L42-L59
```solidity
File: /contracts/src/core/registries/PolicyRegistry.sol
42:        bytes32 currentCommit = commitments[account];

44:        // solhint-disable no-empty-blocks
45:        if (
46:            currentCommit == bytes32(0)
47:                && msg.sender == AddressProviderService._getAuthorizedAddress(_SAFE_DEPLOYER_HASH)
48:        ) {
```

```diff
diff --git a/contracts/src/core/registries/PolicyRegistry.sol b/contracts/src/core/registries/PolicyRegistry.sol
index 559d5ca..2655bff 100644
--- a/contracts/src/core/registries/PolicyRegistry.sol
+++ b/contracts/src/core/registries/PolicyRegistry.sol
@@ -39,11 +39,9 @@ contract PolicyRegistry is AddressProviderService {

         WalletRegistry walletRegistry = WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH));

-        bytes32 currentCommit = commitments[account];
-
         // solhint-disable no-empty-blocks
         if (
-            currentCommit == bytes32(0)
+            commitments[account] == bytes32(0)
                 && msg.sender == AddressProviderService._getAuthorizedAddress(_SAFE_DEPLOYER_HASH)
         ) {
             // In case invoker is safe  deployer
```

## We can optimize the validity checks to save 1 SLOAD 

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/AddressProvider.sol#L52-L57

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 405    | 9770   | 2588 | 26317 |
| After  | 405    | 9721   | 2573 | 26186 |

```solidity
File: /contracts/src/core/AddressProvider.sol
52:    function setGovernance(address _newGovernance) external {
53:        _notNull(_newGovernance);
54:        _onlyGov();
55:        emit GovernanceTransferRequested(governance, _newGovernance);
56:        pendingGovernance = _newGovernance;
57:    }
```

We are calling two an internal function `_onlyGov()` that has the following implementation
```solidity
139:    function _onlyGov() internal view {
140:        if (msg.sender != governance) revert NotGovernance(msg.sender);
141:    }
```
 
This function has a check that `msg.sender` is the same as `governance` which is a state variable.
When this function is called by the main function `setGovernance()`, the only way the emit `GovernanceTransferRequested` event fires is if indeed `msg.sender` == `governance` , therefore we do not have to read state variable twice. 
If we inline the internal function `_onlyGov()` we can utilise `msg.sender` in place of the more expensive `governance`

```diff
diff --git a/contracts/src/core/AddressProvider.sol b/contracts/src/core/AddressProvider.sol
index 5ec51f9..d331cb5 100644
--- a/contracts/src/core/AddressProvider.sol
+++ b/contracts/src/core/AddressProvider.sol
@@ -51,8 +51,8 @@ contract AddressProvider {
      */
     function setGovernance(address _newGovernance) external {
         _notNull(_newGovernance);
-        _onlyGov();
-        emit GovernanceTransferRequested(governance, _newGovernance);
+        if (msg.sender != governance) revert NotGovernance(msg.sender);
+        emit GovernanceTransferRequested(msg.sender, _newGovernance);
         pendingGovernance = _newGovernance;
     }
```
This change would save us one SLOAD 

## Making external calls is more expensive than internal function calls(reorder the checks to have the cheap one come first)

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/PolicyValidator.sol#L100-L118
```solidity
File: /contracts/src/core/PolicyValidator.sol
100:    function isPolicySignatureValid(address account, bytes32 transactionStructHash, bytes calldata signatures)
101:        public
102:        view
103:        returns (bool)
104:    {
105:        // Get policy hash from registry
106:        bytes32 policyHash =
107:            PolicyRegistry(AddressProviderService._getRegistry(_POLICY_REGISTRY_HASH)).commitments(account);
108:        if (policyHash == bytes32(0)) {
109:            revert NoPolicyCommit();
110:        }

112:        // Get expiry epoch and validator signature from signatures
113:        (uint32 expiryEpoch, bytes memory validatorSignature) = _decompileSignatures(signatures);

115:        // Ensure transaction has not expired
116:        if (expiryEpoch < uint32(block.timestamp)) {
117:            revert TxnExpired(expiryEpoch);
118:        }
```

The first if condition `policyHash == bytes32(0)` forces us to do an external function call `AddressProviderService._getRegistry(_POLICY_REGISTRY_HASH)).commitments(account)` in order to get the value of `policyHash`.
To execute the second if check, the only function being called is an internal function `_decompileSignatures` which means only JUMPS are involved which is cheaper compared to making external calls
Calling an internal function costs ~20 gas for the JUMPs.
If the first check passes while the second check fails, we would end up reverting but with a lot of gas having been spent to make the external call
We should reorder the checks to have the cheaper checks come first so that if we do revert, the gas spent would not be  too much


```diff
diff --git a/contracts/src/core/PolicyValidator.sol b/contracts/src/core/PolicyValidator.sol
index 70d672f..51bb21c 100644
--- a/contracts/src/core/PolicyValidator.sol
+++ b/contracts/src/core/PolicyValidator.sol
@@ -102,6 +102,13 @@ contract PolicyValidator is AddressProviderService, EIP712 {
         view
         returns (bool)
     {
+                // Get expiry epoch and validator signature from signatures
+        (uint32 expiryEpoch, bytes memory validatorSignature) = _decompileSignatures(signatures);
+
+        // Ensure transaction has not expired
+        if (expiryEpoch < uint32(block.timestamp)) {
+            revert TxnExpired(expiryEpoch);
+        }
         // Get policy hash from registry
         bytes32 policyHash =
             PolicyRegistry(AddressProviderService._getRegistry(_POLICY_REGISTRY_HASH)).commitments(account);
@@ -109,13 +116,6 @@ contract PolicyValidator is AddressProviderService, EIP712 {
             revert NoPolicyCommit();
         }

-        // Get expiry epoch and validator signature from signatures
-        (uint32 expiryEpoch, bytes memory validatorSignature) = _decompileSignatures(signatures);
-
-        // Ensure transaction has not expired
-        if (expiryEpoch < uint32(block.timestamp)) {
-            revert TxnExpired(expiryEpoch);
-        }

         // Build validation struct hash
         bytes32 validationStructHash = TypeHashHelper._buildValidationStructHash(
```

## Reordering this operations can save us gas in case of a revert on the require check

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeEnabler.sol#L43-L48
```solidity
File: /contracts/src/core/SafeEnabler.sol
43:    function enableModule(address module) public {
44:        _onlyDelegateCall();

48:        require(module != address(0) && module != _SENTINEL_MODULES, "GS101");
```
Calling the private function `_onlydelegate()` costs ~20-40 gas due to the JUMPS involved.
The require statement is way cheaper as it only reads a function parameter and a constant variable
We should reorder the checks to have the require statement come before the function call so as to reduce the amount of gas being used in case we end up reverting 

```diff
diff --git a/contracts/src/core/SafeEnabler.sol b/contracts/src/core/SafeEnabler.sol
index 02137cc..9ed92f3 100644
--- a/contracts/src/core/SafeEnabler.sol
+++ b/contracts/src/core/SafeEnabler.sol
@@ -41,11 +41,10 @@ contract SafeEnabler is GnosisSafeStorage {
      * @param module Module to be whitelisted.
      */
     function enableModule(address module) public {
-        _onlyDelegateCall();
-
         // Module address cannot be null or sentinel.
         // solhint-disable-next-line custom-errors
         require(module != address(0) && module != _SENTINEL_MODULES, "GS101");
+        _onlyDelegateCall();

```

## Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate

**Missed by the bot**
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/AddressProvider.sol#L35-L41
```solidity
File: /contracts/src/core/AddressProvider.sol
35:    mapping(bytes32 => address) public authorizedAddresses;

41:    mapping(bytes32 => address) public registries;
```

## Do not cache variable/calls used only once

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ConsoleFallbackHandler.sol#L83-L87

### No need to cache the call `ISignatureValidator(msg.sender)`

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 2553    | 2553   | 2553 | 2553 |
| After  | 2535    | 2535   | 2535 | 2535 |
```solidity
File: /contracts/src/core/ConsoleFallbackHandler.sol
83:    function isValidSignature(bytes32 _dataHash, bytes calldata _signature) external view returns (bytes4) {
84:        ISignatureValidator validator = ISignatureValidator(msg.sender);
85:        bytes4 value = validator.isValidSignature(abi.encode(_dataHash), _signature);
86:        return (value == EIP1271_MAGIC_VALUE) ? UPDATED_MAGIC_VALUE : bytes4(0);
87:    }
```

```diff
     function isValidSignature(bytes32 _dataHash, bytes calldata _signature) external view returns (bytes4) {
-        ISignatureValidator validator = ISignatureValidator(msg.sender);
-        bytes4 value = validator.isValidSignature(abi.encode(_dataHash), _signature);
-        return (value == EIP1271_MAGIC_VALUE) ? UPDATED_MAGIC_VALUE : bytes4(0);
+        return (ISignatureValidator(msg.sender).isValidSignature(abi.encode(_dataHash), _signature) == EIP1271_MAGIC_VALUE) ? UPDATED_MAGIC_VALUE : bytes4(0);
     }
```


https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ConsoleFallbackHandler.sol#L91-L96

### No need to cache the call `GnosisSafe(payable(msg.sender))`

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 2763    | 6013   | 6013 | 9263 |
| After  | 2755    | 6005   | 6005 | 9255 |
```solidity
File: /contracts/src/core/ConsoleFallbackHandler.sol
91:    function getModules() external view returns (address[] memory) {
92:        // Caller should be a Safe
93:        GnosisSafe safe = GnosisSafe(payable(msg.sender));
94:        (address[] memory array,) = safe.getModulesPaginated(SENTINEL_MODULES, 10);
95:        return array;
96:    }
```

```diff
diff --git a/contracts/src/core/ConsoleFallbackHandler.sol b/contracts/src/core/ConsoleFallbackHandler.sol
index a8a1ae1..b12a8a5 100644
--- a/contracts/src/core/ConsoleFallbackHandler.sol
+++ b/contracts/src/core/ConsoleFallbackHandler.sol
@@ -90,8 +90,7 @@ contract ConsoleFallbackHandler is AddressProviderService, DefaultCallbackHandle
     /// @return Array of modules.
     function getModules() external view returns (address[] memory) {
         // Caller should be a Safe
-        GnosisSafe safe = GnosisSafe(payable(msg.sender));
-        (address[] memory array,) = safe.getModulesPaginated(SENTINEL_MODULES, 10);
+        (address[] memory array,) = GnosisSafe(payable(msg.sender)).getModulesPaginated(SENTINEL_MODULES, 10);
         return array;
     }
```

