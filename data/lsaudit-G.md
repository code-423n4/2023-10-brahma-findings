# [G-01] Unnecessary variables in `src/libraries/SafeHelper.sol` 

* Function `_generateSingleThresholdSignature`

Variable `signatures` is being used only once, which means it does not need to be declared at all. 
There's no need to declare this variable - code can be changed to:

[File: src/libraries/SafeHelper.so](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/libraries/SafeHelper.sol#L87-L94)
```
function _generateSingleThresholdSignature(address owner) internal pure returns (bytes memory) {
        
        return abi.encodePacked(
            bytes12(0), // Padding for signature verifier address
            bytes20(owner), // Signature Verifier
            bytes32(0), // Position of extra data bytes (last set of data)
            bytes1(hex"01") // Signature Type - 1 (presigned transaction)
        );
    }
```

* Function `_getGuard`
Variable `guardAddress` is being used only once, which means it does not need to be declared at all. 
There's no need to declare this variable - code can be changed to:
[File: src/libraries/SafeHelper.so](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/libraries/SafeHelper.sol#L142-L145)
```
    function _getGuard(address safe) internal view returns (address) {
        return address(uint160(uint256(bytes32(IGnosisSafe(safe).getStorageAt(_GUARD_STORAGE_SLOT, 1)))));
    }
```

* Function `_getFallbackHandler`
Variable `fallbackHandlerAddress` is being used only once, which means it does not need to be declared at all. 
There's no need to declare this variable - code can be changed to:
[File: src/libraries/SafeHelper.so](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/libraries/SafeHelper.sol#L152-L155)
```
    function _getFallbackHandler(address safe) internal view returns (address) {
        return address(uint160(uint256(bytes32(IGnosisSafe(safe).getStorageAt(_FALLBACK_HANDLER_STORAGE_SLOT, 1)))));
    }
```


# [G-02] Function `_packMultisendTxns` from `src/libraries/SafeHelper.sol` can be optimized 

* Inside the loop, there are initialization of two variables: `calldataLength` and `encodedTxn`. Declaring them is redundant, since they are used only once:
`calldataLength` is used only in line 120.
`encodedTxn` is used only once, in line 125 or 128 (depending on the `if/else` condition)

This suggests that instead of declaring this variables, we can use them directly:

```

            if (i != 0) {
                // If not first transaction, append to packedTxns
                packedTxns = abi.encodePacked(packedTxns, abi.encodePacked(
                bytes1(call), bytes20(_txns[i].target), bytes32(_txns[i].value), bytes32(_txns[i].data.length), _txns[i].data
            ));
            } else {
                // If first transaction, set packedTxns to encodedTxn
                packedTxns = abi.encodePacked(
                bytes1(call), bytes20(_txns[i].target), bytes32(_txns[i].value), bytes32(_txns[i].data.length), _txns[i].data
            );
            }
```

* Count down, instead of up, in a loop.
Counting from `_txns.length` to 0 (instead of counting from 0 to `_txns.length`) will save you from declaring `uint256 i = 0` variable.
It will be `uint256 len = _txns.length;` which will be used as a counter.
Instead of doing `unchecked { ++i; }`, you will be doing: `unchecked { --len; }` and `do/while` condition will be checking if `len != 0`.
That way, you will be counting down and saving gas from declaring additional variable: `uint256 i`.
In the current implementation, you need to declare `uint256 i` which works as a loop counter (from 0 to `len`). When you will be counting down, the counter will be the variable `len` itself, which will be decreased on every loop iteration.

# [G-03] Unnecessary variables in `src/core/ConsoleFallbackHandler.sol` 

* Function `_generateSingleThresholdSignature`
Variable `safeMessageHash` is being used only once, which means it does not need to be declared at all. 
There's no need to declare this variable - code can be changed to:

[File: src/core/ConsoleFallbackHandler.sol](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ConsoleFallbackHandler.sol#L68-L71)
```
function getMessageHashForSafe(GnosisSafe safe, bytes memory message) public view returns (bytes32) {
        return keccak256(abi.encodePacked(bytes1(0x19), bytes1(0x01), safe.domainSeparator(), keccak256(abi.encode(SAFE_MSG_TYPEHASH, keccak256(message)))));
    }
```

* Function `isValidSignature`
Variable `value ` is being used only once, which means it does not need to be declared at all. 
There's no need to declare this variable - code can be changed to:

[File: src/core/ConsoleFallbackHandler.sol](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ConsoleFallbackHandler.sol#L83-L87)
```
 function isValidSignature(bytes32 _dataHash, bytes calldata _signature) external view returns (bytes4) {
        ISignatureValidator validator = ISignatureValidator(msg.sender);
        return (validator.isValidSignature(abi.encode(_dataHash), _signature) == EIP1271_MAGIC_VALUE) ? UPDATED_MAGIC_VALUE : bytes4(0);
    }
```

# [G-04] Unnecessary variables in `src/core/PolicyValidator.sol` 

* Function `isPolicySignatureValid`
Variable `nonce` is being used only once, which means it does not need to be declared at all. 
There's no need to declare this variable - code can be changed to:

[File: src/core/PolicyValidator.sol](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/PolicyValidator.sol#L54-L76)
```
74:		 nonce: IGnosisSafe(account).nonce()
```

* Function `isPolicySignatureValid`

Variable `txnValidityDigest` is being used only once, which means it does not need to be declared at all. 
There's no need to declare this variable - code can be changed to:

[File: src/core/PolicyValidator.sol](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/PolicyValidator.sol#L100-L141)
```
141:	return SignatureCheckerLib.isValidSignatureNow(trustedValidator, _hashTypedData(validationStructHash), validatorSignature);
```

* Function `_decompileSignatures`

Variable `sigLength` is being used only once, which means it does not need to be declared at all. 
There's no need to declare this variable - code can be changed to:

[File: src/core/PolicyValidator.sol](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/PolicyValidator.sol#L156-L167)
```
        expiryEpoch = uint32(bytes4(_signatures[length - 4:length]));
        validatorSignature = _signatures[length - 8 - uint32(bytes4(_signatures[length - 8:length - 4])):length - 8];
```

# [G-05] Operations in `src/core/PolicyValidator.sol` which won't underflow can be unchecked

In function `_decompileSignatures`, there's a check which requires that `length >= 8` (otherwise, function will revert with `InvalidSignatures()`).
This implies, that below lines can be unchecked:

```
        uint32 sigLength = uint32(bytes4(_signatures[length - 8:length - 4]));
        expiryEpoch = uint32(bytes4(_signatures[length - 4:length]));
```

since we know, that since `length >= 8`, then: `length - 8` and `length - 4` won't underflow.

Please keep in mind, that the last line cannot be `unchecked`: `_signatures[length - 8 - sigLength:length - 8];`, since there's no guarantee (especially for malformed signatures), that `length - 8 - sigLength` won't underflow.


# [G-06] Operations in `src/core/SafeDeployer.sol`, which won't overflow can be unchecked

[File: src/core/SafeDeployer.sol](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeDeployer.sol#L253-L255)
```
    function _genNonce(bytes32 _ownersHash, bytes32 _salt) private returns (uint256) {
        return uint256(keccak256(abi.encodePacked(_ownersHash, ownerSafeCount[_ownersHash]++, _salt, VERSION)));
    }
```

`ownerSafeCount` is defined as `mapping(bytes32 ownersHash => uint256 count) public ownerSafeCount;`.
This implies that the key is uint256, thus incrementing it (`++`) will practically never overflow (`_genNonce` will need to be called `0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff` times).
THis leads to the conclusion, that the whole code of `_getNonce` can be `unchecked`, since `ownerSafeCount[_ownersHash]++` will not overflow.

# [G-07]  Unnecessary variables in `src/core/ExecutorPlugin.sol` 

* Function `_validateExecutionRequest`
Variable `_transactionDigest` is being used only once, which means it does not need to be declared at all. 
There's no need to declare this variable - code can be changed to:

[File: src/core/ExecutorPlugin.sol](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ExecutorPlugin.sol#L106-152)
```

        if (
            !SignatureCheckerLib.isValidSignatureNow(
                execRequest.executor, _hashTypedData(_transactionStructHash), execRequest.executorSignature
            )
        ) {
            revert InvalidExecutor();
        }
```


# [G-08]  Unnecessary variables in `src/core/TransactionValidator.sol` 

* Function `_validateExecutionRequest`
Variables `guard`, `fallbackHandler`, `ownerConsole` are being used only once, which means they do not need to be declared at all. 
There's no need to declare those variables - code can be changed to:

[File: src/core/TransactionValidator.sol](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/TransactionValidator.sol#L181-L197)
```
 function _checkSubAccountSecurityConfig(address _subAccount) internal view {
        // Ensure guard has not been disabled
        if (SafeHelper._getGuard(_subAccount) != AddressProviderService._getAuthorizedAddress(_SAFE_MODERATOR_HASH)) revert InvalidGuard();

        // Ensure fallback handler has not been altered
        if (SafeHelper._getFallbackHandler(_subAccount) != AddressProviderService._getAuthorizedAddress(_CONSOLE_FALLBACK_HANDLER_HASH)) {
            revert InvalidFallbackHandler();
        }

        // Ensure owner console as a module has not been disabled
        if (!IGnosisSafe(_subAccount).isModuleEnabled(WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH)).subAccountToWallet(_subAccount))) revert InvalidModule();
    }
```

# [G-09] Pre-calculate constants in `src/core/ConsoleFallbackHandler.sol`

[File: src/core/ConsoleFallbackHandler.sol](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ConsoleFallbackHandler.sol#L24)
```
bytes4 internal constant SIMULATE_SELECTOR = bytes4(keccak256("simulate(address,bytes)"));
```

Since constants have value which is known before compiling-time, their values should be pre-calculated.

