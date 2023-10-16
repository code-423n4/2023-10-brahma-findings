## 1. Low. Incorrect check in `_isConsoleBeingOverriden()` requires 2 transactions to override Guard or FallbackHandler
### Vulnerability Details
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/TransactionValidator.sol#L149-L174

If user wants to replace Guard with another address, he needs to send 2 transactions: 1) `setGuard(address(0))`; 2) `setGuard(newAddress)`
`_GUARD_REMOVAL_CALLDATA_HASH` here means function `setGuard(address(0))`, the same with fallbackHandler
That's because whole transaction data is hashed, instead of checking only function selector
```solidity
    function _isConsoleBeingOverriden(
        address _from,
        address _to,
        uint256 _value,
        bytes memory _data,
        Enum.Operation _operation
    ) internal pure returns (bool) {
        /**
         * Following conditions validate if the transaction aims to remove guard or fallback handler on Safe
         *         from == to (safe sending txn to itself)
         *         value == 0
         *         data == abi.encodeCall(IGnosisSafe.setGuard, (address(0))) || abi.encodeCall(IGnosisSafe.setFallbackHandler, (address(0)))
         *         operation == Enum.Operation.Call
         *
         * In case these conditions are met, the guard is being removed, return true
         */
        if (_from == _to && _value == 0 && _operation == Enum.Operation.Call) {
 @>         if (SafeHelper._GUARD_REMOVAL_CALLDATA_HASH == keccak256(_data)) {
                return true;
 @>         } else if (SafeHelper._FALLBACK_REMOVAL_CALLDATA_HASH == keccak256(_data)) {
                return true;
            }
        }

        return false;
    }
```

### Recommendation
Check selector instead of whole data. It will allow user to submit just 1 transaction with `setGuard(newAddress)` instead of 2:
```diff
        if (_from == _to && _value == 0 && _operation == Enum.Operation.Call) {
-           if (SafeHelper._GUARD_REMOVAL_CALLDATA_HASH == keccak256(_data)) {
+           if (IGnosisSafe.setGuard.selector == _data[:4]) {
                return true;
-           } else if (SafeHelper._FALLBACK_REMOVAL_CALLDATA_HASH == keccak256(_data)) {
-           } else if (IGnosisSafe.setFallbackHandler.selector == _data[:4]) {
                return true;
            }
        }
```
