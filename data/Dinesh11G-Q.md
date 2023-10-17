## Title: Uncovered Code in `_buildTransactionStructHash` Function

### Impact:
This finding is related to a lack of test coverage in the `_buildTransactionStructHash` function. While the code itself does not appear to contain any security issues, the lack of test coverage for this function can pose a risk. Without thorough testing, vulnerabilities or unintended behaviors may remain undiscovered, affecting the reliability of the hashing process.

### Proof of Concept:
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/TypeHashHelper.sol#L65

### Tools Used:
Manual code audit

### Recommended Mitigation Steps with Fix Code:
To mitigate this issue, increase test coverage for the `_buildTransactionStructHash` function by adding test cases that thoroughly exercise its functionality. Ensure that the struct's data is properly validated before hashing. Here's an example of how to increase test coverage:

```solidity
// Test coverage for _buildTransactionStructHash function
function testBuildTransactionStructHash() public {
    TypeHashHelper.Transaction memory txn;
    // Initialize txn with valid data
    // ...
    bytes32 hash = TypeHashHelper._buildTransactionStructHash(txn);
    // Add assertions to ensure correct hashing
    // ...
}
```

Implement comprehensive test cases to validate the correctness of the hashing process.

--------------------------------------------------------------------------------------------

## Title: Uncovered Code in `_buildValidationStructHash` Function

### Impact:
This finding is related to a lack of test coverage in the `_buildValidationStructHash` function. While the code itself appears secure, the lack of test coverage could result in vulnerabilities or unintended behaviors remaining undetected. This may impact the reliability and security of the hashing process.

### Proof of Concept:
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/TypeHashHelper.sol#L85

### Tools Used:
Manual code audit

### Recommended Mitigation Steps with Fix Code:
To mitigate this issue, increase test coverage for the `_buildValidationStructHash` function by adding test cases that thoroughly exercise its functionality. Ensure that the data processed by this function is properly validated and secure. Here's an example of how to increase test coverage:

```solidity
// Test coverage for _buildValidationStructHash function
function testBuildValidationStructHash() public {
    TypeHashHelper.Validation memory validation;
    // Initialize validation with valid data
    // ...
    bytes32 hash = TypeHashHelper._buildValidationStructHash(validation);
    // Add assertions to ensure correct hashing
    // ...
}
```

Implement comprehensive test cases to validate the correctness of the hashing process.