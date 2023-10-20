## Gas Optimizations

| Number                                                                                         | Issue                                                                           | Instances |
| ---------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------ | :-------: |
| [[G-01](#g-01-refactor-code-to-fail-early-by-using-requirecustom-error-early-in-the-function)] | Refactor code to fail early by using require/custom error early in the function |     1     |
| [[G-02](#g-02-remove-unused-eventerror)]                                                       | Remove unused event/error                                                       |     2     |
| [[G-03](#g-03-dont-cache-value-if-it-is-only-used-once)]                                       | `Don’t cache` value if it is only used once                                     |     8     |
| [[G-04](#g-04-can-make-the-variable-outside-the-loop-to-save-gas)]                             | Can Make The Variable Outside The Loop To Save Gas                              |     4     |

## [G-01] Refactor code to fail early by using require/custom error early in the function.

Using require/custom error before other part of code saves gas when reverting if this code is not used in failing that statement.

**_1_instance_**

```solidity

  File : contracts/src/core/PolicyValidator.sol

    bytes32 txnValidityDigest = _hashTypedData(validationStructHash);

        address trustedValidator = AddressProviderService._getAuthorizedAddress(_TRUSTED_VALIDATOR_HASH);

        // Empty Signature check for EOA signer
        if (trustedValidator.code.length == 0 && validatorSignature.length == 0) {
            // TrustedValidator is an EOA and no trustedValidator signature is provided
            revert InvalidSignature();
        }

        // Validate signature
        return SignatureCheckerLib.isValidSignatureNow(trustedValidator, txnValidityDigest, validatorSignature);
    }

```

[130-142](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L130C7-L142C6)

```diff
  File : contracts/src/core/PolicyValidator.sol

-    bytes32 txnValidityDigest = _hashTypedData(validationStructHash);

        address trustedValidator = AddressProviderService._getAuthorizedAddress(_TRUSTED_VALIDATOR_HASH);

        // Empty Signature check for EOA signer
        if (trustedValidator.code.length == 0 && validatorSignature.length == 0) {
            // TrustedValidator is an EOA and no trustedValidator signature is provided
            revert InvalidSignature();
        }
+    bytes32 txnValidityDigest = _hashTypedData(validationStructHash);
        // Validate signature
        return SignatureCheckerLib.isValidSignatureNow(trustedValidator, txnValidityDigest, validatorSignature);
    }


```

## [G-02] Remove unused event/error

**_2_instances_**

```solidity

File : contracts/src/core/SafeDeployer.sol

  event PreComputeAccount(address[] indexed owners, uint256 indexed threshold);

  error InvalidCommitment();
  error NotWallet();
  error PreComputedAccount(address addr);

```

[35-39](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L35C5-L39C44)

```diff
File : contracts/src/core/SafeDeployer.sol

-  event PreComputeAccount(address[] indexed owners, uint256 indexed threshold);

   error InvalidCommitment();
   error NotWallet();
-  error PreComputedAccount(address addr);

```

## [G-03] `Don’t cache` value if it is only used once

**\_8_instances**

```solidity

File : contracts/src/core/SafeEnabler.sol

  function setGuard(address guard) public {
        _onlyDelegateCall();

        bytes32 slot = _GUARD_STORAGE_SLOT;
        // solhint-disable-next-line no-inline-assembly
        assembly {
            sstore(slot, guard)
        }
        emit ChangedGuard(guard);
    }

```

[66-75](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L66C3-L75C6)

```diff
File : contracts/src/core/SafeEnabler.sol

  function setGuard(address guard) public {
        _onlyDelegateCall();

-        bytes32 slot = _GUARD_STORAGE_SLOT;
        // solhint-disable-next-line no-inline-assembly
        assembly {
-            sstore(slot, guard)
+            sstore(_GUARD_STORAGE_SLOT, guard)
        }
        emit ChangedGuard(guard);
    }

```

```solidity

File : contracts/src/core/TransactionValidator.sol

    function _checkSubAccountSecurityConfig(address _subAccount) internal view {
        address guard = SafeHelper._getGuard(_subAccount);
        address fallbackHandler = SafeHelper._getFallbackHandler(_subAccount);

        // Ensure guard has not been disabled
        if (guard != AddressProviderService._getAuthorizedAddress(_SAFE_MODERATOR_HASH)) revert InvalidGuard();

        // Ensure fallback handler has not been altered
        if (fallbackHandler != AddressProviderService._getAuthorizedAddress(_CONSOLE_FALLBACK_HANDLER_HASH)) {
            revert InvalidFallbackHandler();
        }

        address ownerConsole =
            WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH)).subAccountToWallet(_subAccount);

        // Ensure owner console as a module has not been disabled
        if (!IGnosisSafe(_subAccount).isModuleEnabled(ownerConsole)) revert InvalidModule();
    }

```

[181-198](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L181C1-L198C6)

```diff
File : contracts/src/core/TransactionValidator.sol

    function _checkSubAccountSecurityConfig(address _subAccount) internal view {
-        address guard = SafeHelper._getGuard(_subAccount);
-        address fallbackHandler = SafeHelper._getFallbackHandler(_subAccount);

        // Ensure guard has not been disabled
-        if (guard != AddressProviderService._getAuthorizedAddress(_SAFE_MODERATOR_HASH)) revert InvalidGuard();
+        if (SafeHelper._getGuard(_subAccount) != AddressProviderService._getAuthorizedAddre(_SAFE_MODERATOR_HASH))     revert InvalidGuard();

        // Ensure fallback handler has not been altered
-        if (fallbackHandler != AddressProviderService._getAuthorizedAddress(_CONSOLE_FALLBACK_HANDLER_HASH)) {
+       if (SafeHelper._getFallbackHandler(_subAccount) != AddressProviderService._getAuthorizedAddress          (_CONSOLE_FALLBACK_HANDLER_HASH)) {
            revert InvalidFallbackHandler();
        }

        address ownerConsole =
            WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH)).subAccountToWallet(_subAccount);

        // Ensure owner console as a module has not been disabled
        if (!IGnosisSafe(_subAccount).isModuleEnabled(ownerConsole)) revert InvalidModule();
    }

```

```solidity

File : contracts/src/libraries/SafeHelper.sol

           uint256 calldataLength = _txns[i].data.length;

            bytes memory encodedTxn = abi.encodePacked(
                bytes1(call), bytes20(_txns[i].target), bytes32(_txns[i].value), bytes32(calldataLength), _txns[i].data
            );

```

[117-121](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L117C2-L121C15)

```diff
File : contracts/src/libraries/SafeHelper.sol

-           uint256 calldataLength = _txns[i].data.length;

            bytes memory encodedTxn = abi.encodePacked(
-                bytes1(call), bytes20(_txns[i].target), bytes32(_txns[i].value), bytes32(calldataLength), _txns[i].data
+                bytes1(call), bytes20(_txns[i].target), bytes32(_txns[i].value), bytes32(_txns[i].data.length), _txns[i].data
            );

```

## [G-04] Can Make The Variable Outside The Loop To Save Gas

When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements. By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract

**_4_instances_**

```solidity

File : contracts/src/libraries/SafeHelper.sol

 uint256 i = 0;
        do {
            // Enum.Operation.Call is 0
            uint8 call = uint8(Enum.Operation.Call);
            if (_txns[i].callType == Types.CallType.DELEGATECALL) {
                call = uint8(Enum.Operation.DelegateCall);
            } else if (_txns[i].callType == Types.CallType.STATICCALL) {
                revert InvalidMultiSendCall(i);
            }

            uint256 calldataLength = _txns[i].data.length;

            bytes memory encodedTxn = abi.encodePacked(
                bytes1(call), bytes20(_txns[i].target), bytes32(_txns[i].value), bytes32(calldataLength), _txns[i].data
            );

            if (i != 0) {
                // If not first transaction, append to packedTxns
                packedTxns = abi.encodePacked(packedTxns, encodedTxn);
            } else {
                // If first transaction, set packedTxns to encodedTxn
                packedTxns = encodedTxn;
            }

            unchecked {
                ++i;
            }
        } while (i < len);

```

[107-134](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L107C8-L134C27)

```diff
File : contracts/src/libraries/SafeHelper.sol

   uint256 i = 0;
+  uint8 call;
+  bytes memory encodedTxn;
        do {
            // Enum.Operation.Call is 0
-            uint8 call = uint8(Enum.Operation.Call);
+            call = uint8(Enum.Operation.Call);
            if (_txns[i].callType == Types.CallType.DELEGATECALL) {
                call = uint8(Enum.Operation.DelegateCall);
            } else if (_txns[i].callType == Types.CallType.STATICCALL) {
                revert InvalidMultiSendCall(i);
            }

            uint256 calldataLength = _txns[i].data.length;

-            bytes memory encodedTxn = abi.encodePacked(
+            encodedTxn = abi.encodePacked(
                bytes1(call), bytes20(_txns[i].target), bytes32(_txns[i].value), bytes32(calldataLength), _txns[i].data
            );

            if (i != 0) {
                // If not first transaction, append to packedTxns
                packedTxns = abi.encodePacked(packedTxns, encodedTxn);
            } else {
                // If first transaction, set packedTxns to encodedTxn
                packedTxns = encodedTxn;
            }

            unchecked {
                ++i;
            }
        } while (i < len);

```
