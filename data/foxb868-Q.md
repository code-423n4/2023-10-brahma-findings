## Impact

`_parseOperationEnum` does not handle invalid cases safely. This is a low-severity issue, since it does not directly cause asset loss or lock ups. However, it does undermine expected function behavior.

## Vuln Details

In `_parseOperationEnum`, invalid enum values are not handled safely: **[function _parseOperationEnum](https://github.com/code-423n4/2023-10-brahma/blob/c217699448ffd7ec0253472bf0d156e52d45ca71/contracts/src/libraries/SafeHelper.sol#L163-L170)**

```solidity
// contracts/libraries/SafeHelper.sol

function _parseOperationEnum(Types.CallType callType) internal pure returns (Enum.Operation operation) {

  if (callType == Types.CallType.DELEGATECALL) {
    return Enum.Operation.DelegateCall;
  }

  if (callType == Types.CallType.CALL) {
    return Enum.Operation.Call;
  }

  // Issue - lacks else revert  
}
```
If an invalid `CallType` is passed, it will silently continue instead of reverting.

This violates the expectation that unrecognized values should fail safely.

**Vulnerability:**

The `_parseOperationEnum` function does not revert or handle invalid `CallType` values passed in. Instead, it silently continues without indication of failure.

**Impact:**

- Allowing unrecognized enum values undermines the type safety that enums provide.
- Continuing execution silently masks any errors caused by invalid values.
- This violates the principle of "failing loudly" when bad input is provided.
- Callers may incorrectly assume the parsing succeeded when it actually failed.
- The invalid return value from the function could then be used, leading to unintended behavior.

**Context:** 

As `_parseOperationEnum` is designed to safely convert between two enum types, properly handling invalid values is important for reliability. Silently continuing leaves callers vulnerable to undefined behavior if incorrect inputs are passed. Reverting indicates failure and prevents propagation of bad values.

## Recommended Mitigation Steps

Should revert for unrecognized enum values.
Add an `else { revert("Invalid operation type") }` statement to throw on unrecognized enum values.

```solidity
// contracts/libraries/SafeHelper.sol

function _parseOperationEnum(Types.CallType callType) internal pure returns (Enum.Operation operation) {

  if (callType == Types.CallType.DELEGATECALL) {
    return Enum.Operation.DelegateCall;
  }

  if (callType == Types.CallType.CALL) {
    return Enum.Operation.Call; 
  }

  else {
    revert("Invalid operation type"); // Handles invalid cases
  }

}
```

Adding this on line 166 would make `_parseOperationEnum` safer.
