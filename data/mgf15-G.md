G-1 don't cache .length 

```diff 
diff --git a/PolicyValidator.sol b/aPolicyValidator.sol
index 70d672f..75ada89 100644
--- a/PolicyValidator.sol
+++ b/aPolicyValidator.sol
@@ -158,12 +158,12 @@ contract PolicyValidator is AddressProviderService, EIP712 {
         pure
         returns (uint32 expiryEpoch, bytes memory validatorSignature)
     {
-        uint256 length = _signatures.length;
-        if (length < 8) revert InvalidSignatures();
+        //uint256 length = _signatures.length;
+        if (_signatures.length < 8) revert InvalidSignatures();
 
-        uint32 sigLength = uint32(bytes4(_signatures[length - 8:length - 4]));
-        expiryEpoch = uint32(bytes4(_signatures[length - 4:length]));
-        validatorSignature = _signatures[length - 8 - sigLength:length - 8];
+        uint32 sigLength = uint32(bytes4(_signatures[_signatures.length - 8:_signatures.length - 4]));
+        expiryEpoch = uint32(bytes4(_signatures[_signatures.length - 4:_signatures.length]));
+        validatorSignature = _signatures[_signatures.length - 8 - sigLength:_signatures.length - 8];
     }

```

Gas Savings : 

```diff
| src/core/PolicyValidator.sol:PolicyValidator contract                   |                 |       |        |       |         |
 |-------------------------------------------------------------------------|-----------------|-------|--------|-------|---------|
 | Deployment Cost                                                         | Deployment Size |       |        |       |         |
-| 789675                                                                  | 4406            |       |        |       |         |
+| 789275                                                                  | 4404            |       |        |       |         |
 | Function Name                                                           | min             | avg   | median | max   | # calls |
 | addressProviderTarget                                                   | 180             | 180   | 180    | 180   | 205     |
 | eip712Domain                                                            | 1192            | 1192  | 1192   | 1192  | 1       |
-| isPolicySignatureValid(address,address,uint256,bytes,uint8,bytes)(bool) | 4870            | 19983 | 22381  | 22736 | 39      |
-| isPolicySignatureValid(address,bytes32,bytes)(bool)                     | 3885            | 14109 | 12525  | 20389 | 31      |
+| isPolicySignatureValid(address,address,uint256,bytes,uint8,bytes)(bool) | 4867            | 19978 | 22376  | 22731 | 39      |
+| isPolicySignatureValid(address,bytes32,bytes)(bool)                     | 3880            | 14104 | 12520  | 20384 | 31      |

```
```diff
diff --git a/PolicyValidator.sol b/aPolicyValidator.sol
index b39f78b..75ada89 100644
--- a/PolicyValidator.sol
+++ b/aPolicyValidator.sol
@@ -158,12 +158,12 @@ contract PolicyValidator is AddressProviderService, EIP712 {
         pure
         returns (uint32 expiryEpoch, bytes memory validatorSignature)
     {
-        uint256 length = _signatures.length;
-        if (length < 8) revert InvalidSignatures();
+        //uint256 length = _signatures.length;
+        if (_signatures.length < 8) revert InvalidSignatures();
 
-        uint32 sigLength = uint32(bytes4(_signatures[length - 8:length - 4]));
-        expiryEpoch = uint32(bytes4(_signatures[length - 4:length]));
-        validatorSignature = _signatures[length - 8 - sigLength:length - 8];
+        uint32 sigLength = uint32(bytes4(_signatures[_signatures.length - 8:_signatures.length - 4]));
+        expiryEpoch = uint32(bytes4(_signatures[_signatures.length - 4:_signatures.length]));
+        validatorSignature = _signatures[_signatures.length - 8 - sigLength:_signatures.length - 8];
     }

```
```diff
 | src/core/PolicyValidator.sol:PolicyValidator contract                   |                 |       |        |       |         |
 |-------------------------------------------------------------------------|-----------------|-------|--------|-------|---------|
 | Deployment Cost                                                         | Deployment Size |       |        |       |         |
-| 789675                                                                  | 4406            |       |        |       |         |
+| 789275                                                                  | 4404            |       |        |       |         |
 | Function Name                                                           | min             | avg   | median | max   | # calls |
 | addressProviderTarget                                                   | 180             | 180   | 180    | 180   | 205     |
 | eip712Domain                                                            | 1192            | 1192  | 1192   | 1192  | 1       |
-| isPolicySignatureValid(address,address,uint256,bytes,uint8,bytes)(bool) | 4870            | 19983 | 22381  | 22736 | 39      |
-| isPolicySignatureValid(address,bytes32,bytes)(bool)                     | 3885            | 14109 | 12525  | 20389 | 31      |
+| isPolicySignatureValid(address,address,uint256,bytes,uint8,bytes)(bool) | 4867            | 19978 | 22376  | 22731 | 39      |
+| isPolicySignatureValid(address,bytes32,bytes)(bool)                     | 3880            | 14104 | 12520  | 20384 | 31      |

```
G-2 Use `assembly` to write address storage values 

```diff 
diff --git a/AddressProvider.sol b/aAddressProvider.sol
index 5ec51f9..4eb7be9 100644
--- a/AddressProvider.sol
+++ b/aAddressProvider.sol
@@ -42,18 +42,24 @@ contract AddressProvider {
 
     constructor(address _governance) {
         _notNull(_governance);
-        governance = _governance;
+        //governance = _governance;
+        assembly {
+            sstore(governance.slot, _governance)
+        }
     }
 
     /**
      * @notice Governance setter
      * @param _newGovernance address of new governance
-     */
+     */ 
     function setGovernance(address _newGovernance) external {
         _notNull(_newGovernance);
         _onlyGov();
         emit GovernanceTransferRequested(governance, _newGovernance);
-        pendingGovernance = _newGovernance;
+        //pendingGovernance = _newGovernance;
+        assembly {
+            sstore(pendingGovernance.slot, _newGovernance)
+        }
     }
 
     /**
@@ -64,7 +70,11 @@ contract AddressProvider {
             revert NotPendingGovernance(msg.sender);
         }
         emit GovernanceTransferred(governance, msg.sender);
-        governance = msg.sender;
+        //governance = msg.sender;
+        address sender = msg.sender;
+        assembly{
+            sstore(governance.slot,sender)
+        }
         delete pendingGovernance;
     }
 

```
Gas Savings : 

```diff
 | src/core/AddressProvider.sol:AddressProvider contract |                 |       |        |       |         |
 |-------------------------------------------------------|-----------------|-------|--------|-------|---------|
 | Deployment Cost                                       | Deployment Size |       |        |       |         |
-| 431462                                                | 2337            |       |        |       |         |
+| 416183                                                | 2232            |       |        |       |         |
 | Function Name                                         | min             | avg   | median | max   | # calls |
-| acceptGovernance                                      | 2371            | 3267  | 3267   | 4163  | 2       |
+| acceptGovernance                                      | 2371            | 3216  | 3216   | 4061  | 2       |
 | getAuthorizedAddress                                  | 464             | 787   | 464    | 2464  | 1775    |
 | getRegistry                                           | 463             | 866   | 463    | 2463  | 779     |
 | governance                                            | 368             | 2186  | 2368   | 2368  | 11      |
 | pendingGovernance                                     | 345             | 1345  | 1345   | 2345  | 4       |
 | setAuthorizedAddress                                  | 2459            | 24189 | 25123  | 27202 | 2979    |
-| setGovernance                                         | 405             | 13906 | 14452  | 26317 | 4       |
+| setGovernance                                         | 405             | 13887 | 14433  | 26278 | 4       |
 | setRegistry                                           | 1476            | 24985 | 25093  | 29671 | 622     |
```

G-3 Split require 

Note: *increase in Deployment Cost but decrease on Runtime Cost*

```diff
diff --git a/SafeEnabler.sol b/aSafeEnabler.sol
index 02137cc..2f5aad5 100644
--- a/SafeEnabler.sol
+++ b/aSafeEnabler.sol
@@ -45,8 +45,8 @@ contract SafeEnabler is GnosisSafeStorage {
 
         // Module address cannot be null or sentinel.
         // solhint-disable-next-line custom-errors
-        require(module != address(0) && module != _SENTINEL_MODULES, "GS101");
-
+        require(module != address(0), "GS101");
+        require(module != address(0x1), "GS101");
         // Module cannot be added twice.
         // solhint-disable-next-line custom-errors
         require(modules[module] == address(0), "GS102");
```

```diff
 | src/core/SafeEnabler.sol:SafeEnabler contract |                 |       |        |       |         |
 |-----------------------------------------------|-----------------|-------|--------|-------|---------|
 | Deployment Cost                               | Deployment Size |       |        |       |         |
-| 190258                                        | 998             |       |        |       |         |
+| 214683                                        | 1120            |       |        |       |         |
 | Function Name                                 | min             | avg   | median | max   | # calls |
-| enableModule                                  | 311             | 23365 | 24170  | 28970 | 106     |
+| enableModule                                  | 311             | 23348 | 24153  | 28953 | 106     |
 | setGuard                                      | 333             | 23176 | 23483  | 23483 | 158     |
```

G-4 assembly method available 

```diff
diff --git a/AddressProvider.sol b/aAddressProvider.sol
index 5ec51f9..607d99a 100644
--- a/AddressProvider.sol
+++ b/aAddressProvider.sol
@@ -81,7 +81,13 @@ contract AddressProvider {
         /// @dev skips checks for supported `addressProvider()` if `_overrideCheck` is true
         if (!_overrideCheck) {
             /// @dev skips checks for supported `addressProvider()` if `_authorizedAddress` is an EOA
-            if (_authorizedAddress.code.length != 0) _ensureAddressProvider(_authorizedAddress);
+            uint256 _codesize;
+            assembly{
+                _codesize := extcodesize(_authorizedAddress)
+            }
+            if (_codesize != 0) _ensureAddressProvider(_authorizedAddress);
+            
+        
         }

```

```diff
diff --git a/PolicyValidator.sol b/aPolicyValidator.sol
index 70d672f..42cf312 100644
--- a/PolicyValidator.sol
+++ b/aPolicyValidator.sol
@@ -132,7 +132,11 @@ contract PolicyValidator is AddressProviderService, EIP712 {
         address trustedValidator = AddressProviderService._getAuthorizedAddress(_TRUSTED_VALIDATOR_HASH);
 
         // Empty Signature check for EOA signer
-        if (trustedValidator.code.length == 0 && validatorSignature.length == 0) {
+        uint256 _codesize;
+        assembly{
+            _codesize := extcodesize(trustedValidator)
+        }
+        if (_codesize == 0 && validatorSignature.length == 0) {
             // TrustedValidator is an EOA and no trustedValidator signature is provided
             revert InvalidSignature();
         }

```
```diff
| src/core/AddressProvider.sol:AddressProvider contract |                 |       |        |       |         |
 |-------------------------------------------------------|-----------------|-------|--------|-------|---------|
 | Deployment Cost                                       | Deployment Size |       |        |       |         |
-| 431462                                                | 2337            |       |        |       |         |
+| 427662                                                | 2318            |       |        |       |         |
 | Function Name                                         | min             | avg   | median | max   | # calls |
 | acceptGovernance                                      | 2371            | 3267  | 3267   | 4163  | 2       |
 | getAuthorizedAddress                                  | 464             | 787   | 464    | 2464  | 1775    |
```

```diff
| src/core/PolicyValidator.sol:PolicyValidator contract                   |                 |       |        |       |         |
 |-------------------------------------------------------------------------|-----------------|-------|--------|-------|---------|
 | Deployment Cost                                                         | Deployment Size |       |        |       |         |
-| 789675                                                                  | 4406            |       |        |       |         |
+| 785668                                                                  | 4386            |       |        |       |         |
 | Function Name                                                           | min             | avg   | median | max   | # calls |
 | addressProviderTarget                                                   | 180             | 180   | 180    | 180   | 205     |
 | eip712Domain                                                            | 1192            | 1192  | 1192   | 1192  | 1       |
-| isPolicySignatureValid(address,address,uint256,bytes,uint8,bytes)(bool) | 4870            | 19983 | 22381  | 22736 | 39      |
-| isPolicySignatureValid(address,bytes32,bytes)(bool)                     | 3885            | 14109 | 12525  | 20389 | 31      |
+| isPolicySignatureValid(address,address,uint256,bytes,uint8,bytes)(bool) | 4870            | 19982 | 22380  | 22735 | 39      |
+| isPolicySignatureValid(address,bytes32,bytes)(bool)                     | 3885            | 14108 | 12524  | 20388 | 31      |
```

G-5 use calldata for bytes


```diff
diff --git a/TransactionValidator.sol b/aTransactionValidator.sol
index f31fe06..b9d1865 100644
--- a/TransactionValidator.sol
+++ b/aTransactionValidator.sol
@@ -120,7 +120,7 @@ contract TransactionValidator is AddressProviderService {
         address, /*msgSender */
         address from,
         bytes32 transactionStructHash,
-        bytes memory signatures
+        bytes calldata signatures
     ) external view {
         _validatePolicySignature(from, transactionStructHash, signatures);
     }

```


Note : *increase in Deployment Cost but decrease on Runtime Cost*

```diff
| src/core/TransactionValidator.sol:TransactionValidator contract |                 |       |        |       |         |
 |-----------------------------------------------------------------|-----------------|-------|--------|-------|---------|
 | Deployment Cost                                                 | Deployment Size |       |        |       |         |
-| 840784                                                          | 4429            |       |        |       |         |
+| 863203                                                          | 4541            |       |        |       |         |
 | Function Name                                                   | min             | avg   | median | max   | # calls |
 | addressProvider                                                 | 215             | 215   | 215    | 215   | 3       |
 | addressProviderTarget                                           | 179             | 179   | 179    | 179   | 205     |
 | validatePostExecutorTransaction                                 | 6242            | 14879 | 10621  | 36621 | 16      |
 | validatePostTransaction                                         | 10218           | 20027 | 18599  | 36599 | 28      |
 | validatePostTransactionOverridable                              | 431             | 431   | 431    | 431   | 9       |
-| validatePreExecutorTransaction                                  | 3577            | 11982 | 11859  | 27359 | 17      |
+| validatePreExecutorTransaction                                  | 3487            | 11892 | 11769  | 27269 | 17      |
 | validatePreTransaction                                          | 6036            | 28348 | 31637  | 31998 | 30      |
 | validatePreTransactionOverridable                               | 2424            | 14112 | 7459   | 32333 | 14      |

```