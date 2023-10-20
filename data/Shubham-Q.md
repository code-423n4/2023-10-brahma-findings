## Gas Optimization

| |Issue|Instances|
|-|:-|:-:|
| [GAS-01](#GAS-01) | Use Modifiers instead of calling the functions for checks  | 2 |
| [GAS-02](#GAS-02) | Unused function in `AddressProviderService` | 1 |
| [GAS-03](#GAS-03) | Checking that the new policy commit isn't already assigned saves gas  | 1 |
| [GAS-04](#GAS-04) | Don't initialize default values to variables | 2 |
| [GAS-05](#GAS-05) | Variables should be cached outside loop | 2 |
| [GAS-06](#GAS-06) | Don't cache state variables or functions only used once | 8 |
| [GAS-07](#GAS-07) | Checks should be placed in the beginning of the function | 1 |
| [GAS-08](#GAS-08) | State variables that are used multiple times in a function should be cached in stack variables | 2 |

## [GAS-01] Use Modifiers instead of calling the functions for checks

Various functions inside `AddressProvider` are calling other functions just to ensure some valid needed checks.
But calling those functions each time incur high gas cost each time. 
Instead replace those functions with suitable modifiers.

*There are 2 instances of this issue.*
```solidity
File: AddressProvider.sol

-    function _onlyGov() internal view {
-        if (msg.sender != governance) revert NotGovernance(msg.sender);
-    }

+    modifier _onlyGov() {
+        if (msg.sender != governance) revert NotGovernance(msg.sender);
+        _;
+    }



-    function _notNull(address addr) internal pure {
-        if (addr == address(0)) revert NullAddress();
-    }

+    modifier _notNull(address addr) {
+        if (addr == address(0)) revert NullAddress();
+        _;
+    }

```

## [GAS-02] Remove unused function in `AddressProviderService`

*There is 1 instance of this issue.*
```solidity
File: AddressProviderService.sol

    /**
     * @notice Checks if msg.sender is governance
     */
    function _onlyGov() internal view {
        if (msg.sender != addressProvider.governance()) {
            revert NotGovernance(msg.sender);
        }
    }
```

## [GAS-03] Checking that the new policy commit isn't already assigned saves gas

Checking that the new policy commit being updated in `updatePolicy()` isn't already assigned saves the execution of the rest of the function & reduces gas.
*There is 1 instance of this issue.*

```solidity
File: PolicyRegistry.sol

      function updatePolicy(address account, bytes32 policyCommit) external {
        if (policyCommit == bytes32(0)) {
            revert PolicyCommitInvalid();
        }
       
+       if (commitments[account] == policyCommit) {
+           revert PolicyExists();
        ......
```

## [GAS-04] Don't initialize default values to variables

Saves 13 GAS for local variable and 2000 GAS for state variable.
*There are 2 instances of this issue.*

```solidity
File: SafeModeratorOverridable.sol

21:    uint8 public constant DIFFER_SAFE_MOD = 0;
```

```solidity
File: SafeHelper.sol

107:    uint256 i = 0;
```

## [GAS-05] Variables should be cached outside loop

Caching of a state variable replaces each Gwarmaccess (100 gas) with a cheaper stack read. 
Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.
*There are 2 instances of this issue.*

```solidity
File: SafeHelper.sol

+     Types.CallType delType = Types.CallType.DELEGATECALL;
+     Types.CallType staticType = Types.CallType.STATICCALL;
      do {
            // Enum.Operation.Call is 0
            uint8 call = uint8(Enum.Operation.Call);
-           if (_txns[i].callType == Types.CallType.DELEGATECALL) {
+           if (_txns[i].callType == delType) {
                call = uint8(Enum.Operation.DelegateCall);
-           } else if (_txns[i].callType == Types.CallType.STATICCALL) {
+           } else if (_txns[i].callType == staticType) {
                revert InvalidMultiSendCall(i);
            }
           ...........
```

## [GAS-06] Don't cache state variables or functions only used once

```solidity
File: TransactionValidator.sol

    function _checkSubAccountSecurityConfig(address _subAccount) internal view {
-       address guard = SafeHelper._getGuard(_subAccount);
-       address fallbackHandler = SafeHelper._getFallbackHandler(_subAccount);

        // Ensure guard has not been disabled
-       if (guard != AddressProviderService._getAuthorizedAddress(_SAFE_MODERATOR_HASH)) revert InvalidGuard();
+       if (SafeHelper._getGuard(_subAccount) != AddressProviderService._getAuthorizedAddress(_SAFE_MODERATOR_HASH)) {
        revert InvalidGuard();
        }

        // Ensure fallback handler has not been altered
-       if (fallbackHandler != AddressProviderService._getAuthorizedAddress(_CONSOLE_FALLBACK_HANDLER_HASH)) {
+       if (SafeHelper._getFallbackHandler(_subAccount) != AddressProviderService._getAuthorizedAddress(_CONSOLE_FALLBACK_HANDLER_HASH)) {
            revert InvalidFallbackHandler();
        }
``` 
```solidity
File: ExecutorRegistry.sol

function registerExecutor(address _subAccount, address _executor) external {
-       WalletRegistry _walletRegistry = WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH));
-       if (!_walletRegistry.isOwner(msg.sender, _subAccount)) revert NotOwnerWallet();
+       if (!WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH)).isOwner(msg.sender, _subAccount)) revert NotOwnerWallet();

        if (!subAccountToExecutors[_subAccount].add(_executor)) revert AlreadyExists();
        emit RegisterExecutor(_subAccount, msg.sender, _executor);
    }


    function deRegisterExecutor(address _subAccount, address _executor) external {
-       WalletRegistry _walletRegistry = WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH));
-       if (_walletRegistry.subAccountToWallet(_subAccount) != msg.sender) revert NotOwnerWallet();
+       if (WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH)) != msg.sender) revert NotOwnerWallet();

        if (!subAccountToExecutors[_subAccount].remove(_executor)) revert DoesNotExist();
        emit DeRegisterExecutor(_subAccount, msg.sender, _executor);
    }
```
```solidity
File: ExecutorPlugin.sol

    function _validateExecutionRequest(ExecutionRequest calldata execRequest) internal {
        // Fetch executor registry
-       ExecutorRegistry executorRegistry =
            ExecutorRegistry(AddressProviderService._getRegistry(_EXECUTOR_REGISTRY_HASH));

        // Check if executor is valid for given account
-       if (!executorRegistry.isExecutor(execRequest.account, execRequest.executor)) {
+       if (!ExecutorRegistry(AddressProviderService._getRegistry(_EXECUTOR_REGISTRY_HASH)).isExecutor(execRequest.account, execRequest.executor)) {
            revert InvalidExecutor();
        }
```

```solidity
File: SafeDeployer.sol

- 223     address gnosisProxyFactory = AddressProviderService._getAuthorizedAddress(_GNOSIS_PROXY_FACTORY_HASH);
- 230     try IGnosisProxyFactory(gnosisProxyFactory).createProxyWithNonce(gnosisSafeSingleton, _initializer, nonce)

+ 230     try IGnosisProxyFactory(AddressProviderService._getAuthorizedAddress(_GNOSIS_PROXY_FACTORY_HASH)).createProxyWithNonce(gnosisSafeSingleton, _initializer, nonce)
```

```solidity
File: AddressProviderService.sol

    function _getRegistry(bytes32 _key) internal view returns (address registry) {
-       registry = addressProvider.getRegistry(_key);
-       _notNull(registry);
+       _notNull(addressProvider.getRegistry(_key));
    }

    function _getAuthorizedAddress(bytes32 _key) internal view returns (address authorizedAddress) {
-       authorizedAddress = addressProvider.getAuthorizedAddress(_key);
-       _notNull(authorizedAddress);
+       _notNull(addressProvider.getAuthorizedAddress(_key));
    }
```

## [GAS-07] Checks should be placed in the beginning of the function

Necessary checks that are needed for proper execution of a function should be placed early on in the function to save gas.

```solidity
File: ExecutorPlugin.sol

    function executeTransaction(ExecutionRequest calldata execRequest) external nonReentrant returns (bytes memory) {
        _validateExecutionRequest(execRequest);
        
+       TransactionValidator(AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH))
            .validatePostExecutorTransaction(msg.sender, execRequest.account);

        bytes memory txnResult = _executeTxnAsModule(execRequest.account, execRequest.exec);

-       TransactionValidator(AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH))
            .validatePostExecutorTransaction(msg.sender, execRequest.account);

        return txnResult;
    }
```

## [GAS-08] State variables that are used multiple times in a function should be cached in stack

Caching state variable with memory saves 100 GAS , 1 SLOD 

```solidity
File: ExecutorPlugin.sol

+       address execRequestAcc = execRequest.account;
+       address execRequestExe = execRequest.executor;

- 112          if (!executorRegistry.isExecutor(execRequest.account, execRequest.executor)) {
+ 112          if (!executorRegistry.isExecutor(execRequestAcc, execRequestExe)) {


- 129          account: execRequest.account,
- 130          executor: execRequest.executor,
- 131          nonce: executorNonce[execRequest.account][execRequest.executor]++

+ 129          account: execRequestAcc,
+ 130          executor: execRequestExe,
+ 131          nonce: executorNonce[execRequestAcc][execRequestExe]++

- 141          execRequest.executor, _transactionDigest, execRequest.executorSignature
+ 141          execRequestExe, _transactionDigest, execRequest.executorSignature

- 150          msg.sender, execRequest.account, _transactionStructHash, execRequest.validatorSignature
+ 150          msg.sender, execRequestAcc, _transactionStructHash, execRequest.validatorSignature

```
