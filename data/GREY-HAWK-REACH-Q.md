# [N-01] Redundant getters for public mappings 
Additional getters of public mappings are implemented. Either directly call the public mapping to retrieve the desired values, or make the mappings internal or private.

There are 3 instances of this issue:

```
File: src/core/AddressProvider.sol
function getAuthorizedAddress(bytes32 _key) external view returns (address) {
        return authorizedAddresses[_key];
    }
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L112-L114

```
File: src/core/AddressProvider.sol
 function getRegistry(bytes32 _key) external view returns (address) {
        return registries[_key];
    }
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L121-L123

```
File: src/core/registries/WalletRegistry.sol
function getSubAccountsForWallet(address _wallet) external view returns (address[] memory) {
        return walletToSubAccountList[_wallet];
    }
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L63-L65

# [N-02] Typo in the function name
[TransactionValidator.sol#L149](https://github.com/code-423n4/2023-10-brahma/blob/a6424230052fc47c4215200c19a8eef9b07dfccc/contracts/src/core/TransactionValidator.sol#L149)
[TransactionValidator.sol#L66](https://github.com/code-423n4/2023-10-brahma/blob/a6424230052fc47c4215200c19a8eef9b07dfccc/contracts/src/core/TransactionValidator.sol#L66)
```diff
-   function _isConsoleBeingOverriden(
+   function _isConsoleBeingOverridden(
```
```diff
-       if (_isConsoleBeingOverriden(txParams.from, txParams.to, txParams.value, txParams.data, txParams.operation)) {
+       if (_isConsoleBeingOverridden(txParams.from, txParams.to, txParams.value, txParams.data, txParams.operation)) {
```

# [N-03] EOAs access controls

EOAs can registerWallet() themselves and deploySubAccount's. Consider adding extra checks so only SafeWallets are able to do that.

# [N-04] SafeHelper#_executeOnSafe - redundant code
[SafeHelper.sol#L50-L78](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/libraries/SafeHelper.sol#L50-L78)

Safe's code has the following [line](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/lib/safe-contracts/contracts/GnosisSafe.sol#L180):
```
require(success || safeTxGas != 0 || gasPrice != 0, "GS013");
```
Because `safeTxGas` and `gasPrice` are hardcoded as zeroes, it can be deduced to `require(success)`. 

`success == false` - the execution reverts immediately.
`success == true` - the execution is successful; `execTransaction` returns `true`.

Thus, it is impossible for execTransaction to return `false` if `safeTxGas == 0` and `gasPrice == 0`.

## Recommendation
```diff
-   error SafeExecTransactionFailed();
    /* ... */
    function _executeOnSafe(address safe, address target, Enum.Operation op, bytes memory data) internal {
-       bool success = IGnosisSafe(safe).execTransaction(
+       IGnosisSafe(safe).execTransaction(
            address(target), // to
            0, // value
            data, // data
            op, // operation
            0, // safeTxGas
            0, // baseGas
            0, // gasPrice
            address(0), // gasToken
            payable(address(0)), // refundReceiver
            _generateSingleThresholdSignature(address(this)) // signatures
        );

-       if (!success) revert SafeExecTransactionFailed();
    }
```
# [N-05] Guardian role is not implemented
[RegistriesRoles.md](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/docs/RegistriesRoles.md#:~:text=Guardian,security%20and%20management.)
# [N-06] Consider enforcing the order of owner addresses in deployConsoleAccount
[SafeDeployer.sol#L49](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeDeployer.sol#L49)
```diff
function deployConsoleAccount(
        address[] calldata _owners,
        uint256 _threshold,
        bytes32 _policyCommit,
        bytes32 _salt
    ) external nonReentrant returns (address _safe) {
+   uint256 ownersLengthMinusOne = _owners.length - 1;
+   for (uint i; i < ownersLengthMinusOne; i++){
+       require(_owners[i] < _owners[i + 1], "Wrong order");
+   }
    /* ... */
```
