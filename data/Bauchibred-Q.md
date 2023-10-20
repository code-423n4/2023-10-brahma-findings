# QA Report

## **Table of Contents**

|              | Issue                                                                                          |
| ------------ | :--------------------------------------------------------------------------------------------- |
| QA&#x2011;01 | Remove Testing Variables Prior to Production Release                                           |
| QA&#x2011;02 | Address Typographical Errors                                                                   |
| QA&#x2011;03 | Struct Definitions Should Adhere to Established Standards in `TypeHashHelper.sol`              |
| QA&#x2011;04 | `registerSubAccount()` shouldn't accept an already active wallet                               |
| QA&#x2011;05 | Implementation to guarantee on-chain security for future use is flawed and requires correction |
| QA&#x2011;06 | `_executeOnSafe()`'s Execution is Flawed Due to Missing Value in Payable Call                  |
| QA&#x2011;07 | Potential Signature Bypass with Empty Signatures                                               |
| QA&#x2011;08 | Add a function to remove a wallet/subAccount                                                   |
| QA&#x2011;09 | Documentation Should Reference Active Links                                                    |
| QA&#x2011;10 | Unsafe Downcasting without Proper Checks                                                       |
| QA&#x2011;11 | Add `STATICCALL` Support in `_parseOperationEnum()`                                            |

## QA-01 Remove Testing Variables Prior to Production Release

### Impact

**Low** Due to this oversight an unused variable in final production would be passed to final production

### Proof of Concept

Examine the `SafeModeratorOverridable.sol` contract:

```solidity
contract SafeModeratorOverridable is AddressProviderService, IGuard {
    /**
     * @dev Token interface change used to bypass foundry coverage issue
     * Refer https://github.com/foundry-rs/foundry/issues/5729
     */
    uint8 public constant DIFFER_SAFE_MOD = 0;

    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}
```

### Recommended Mitigation Steps

Omit `DIFFER_SAFE_MOD` from the final production version.

## QA-02 Address Typographical Errors

### Impact

**Low:** Proper wording will enhance code clarity and structure.

### Proof of Concept

Inspect `validatePostTransactionOverridable()`:

```solidity
    /* solhint-disable no-empty-blocks */
    /**
     * @notice Provides on-chain guarantees on security critical expected states of a Brhma console account
     * @dev Empty hook available for future use
     */
    function validatePostTransactionOverridable(bytes32, /*txHash */ bool, /*success */ address /*console */ )
        external
        view
    {}
```

### Recommended Mitigation Steps

Update:
`     * @notice Provides on-chain guarantees on security critical expected states of a Brhma console account`
to:
`     * @notice Provides on-chain guarantees on security critical expected states of a *Brahma* console account`

---

## QA-03 Struct Definitions Should Adhere to Established Standards in `TypeHashHelper.sol`

### Impact

Misalignment of structs may lead to transaction and validation inconsistencies compared to EIP specified structs.

### Proof of Concept

Review `_buildTransactionStructHash()`:

```solidity
    function _buildTransactionStructHash(Transaction memory txn) internal pure returns (bytes32) {
        return keccak256(
            abi.encode(
                TRANSACTION_PARAMS_TYPEHASH,
                txn.to,
                txn.value,
                keccak256(txn.data),
                txn.operation,
                txn.account,
                txn.executor,
                txn.nonce
            )
        );
    }
```

However, the proposed structure is as follows:

```solidity
    struct Transaction {
        uint8 operation;
        address to;
        address account;
        address executor;
        uint256 value;
        uint256 nonce;
        bytes data;
    }
```

### Recommended Mitigation Steps

Maintain a consistent struct pattern, preferably aligning with the EIP712 standard.

---

## QA-04 `registerSubAccount()` shouldn't accept an already active wallet

## Impact

**Med to QA:**... An important invariant, which prohibits allowing a wallet to become a subaccount of another account, might be violated through the `registerSubAccount()` function.

## Proof of Concept

Consider the following:

```solidity
    function registerSubAccount(address _wallet, address _subAccount) external {
        if (msg.sender != AddressProviderService._getAuthorizedAddress(_SAFE_DEPLOYER_HASH)) revert InvalidSender();
        if (subAccountToWallet[_subAccount] != address(0)) revert AlreadyRegistered();
        subAccountToWallet[_subAccount] = _wallet;
        walletToSubAccountList[_wallet].push(_subAccount);
        emit RegisterSubAccount(_wallet, _subAccount);
    }
```

From the above, it's evident that this function registers a sub-account for a safe. However, the current codebase emphasizes an invariant that a wallet should not be registered as a sub-account of another wallet, and vice versa.

This invariant is clearly demonstrated in the `registerWallet()` function and its checks:

```solidity
    function registerWallet() external {
        if (isWallet[msg.sender]) revert AlreadyRegistered();
        if (subAccountToWallet[msg.sender] != address(0)) revert IsSubAccount();
        isWallet[msg.sender] = true;
        emit RegisterWallet(msg.sender);
    }
```

Yet, in the registration of the sub-account (as shown above), the only check applied ensures that the account hasn't been previously registered as a sub-account. This means an existing wallet can be set as a subaccount to another wallet, contradicting the stated invariants.

## Recommended Mitigation Steps

Ensure that an active wallet cannot be registered as a subaccount to another.

## QA-05 Implementation to guarantee on-chain security for future use is flawed and requires correction

### Impact

**Medium to Low** Due to this oversight, achieving critical functionality for guaranteeing on-chain security might not be feasible.

### Proof of Concept

Refer to the `validatePostTransactionOverridable()` function:

```solidity
    /* solhint-disable no-empty-blocks */
    /**
     * @notice Provides on-chain guarantees on security critical expected states of a Brahma console account
     * @dev Empty hook available for future use
     */
    function validatePostTransactionOverridable(bytes32, /*txHash */ bool, /*success */ address /*console */ )
        external
        view
    {}
```

This function is intended to be overridable to accommodate future implementations. However, it lacks the `virtual` keyword, meaning it cannot be overridden as anticipated.

### Recommended Mitigation Steps

## Incorporate the `virtual` keyword to the function.

## QA-06 `_executeOnSafe()`'s Execution is Flawed Due to Missing Value in Payable Call

### Impact

**Medium to Low:** The provided function `_executeOnSafe` is attempting to make a call to `IGnosisSafe(safe).execTransaction()`, which is marked as `payable`. However, it's doing so without sending any value (Ether) to it. If the `execTransaction` function is dependent on receiving Ether, it can lead to a failed transaction, potentially blocking crucial operations on the GnosisSafe contract or leading to unexpected behavior.

### Proof of Concept

```solidity
function _executeOnSafe(address safe, address target, Enum.Operation op, bytes memory data) internal {
    bool success = IGnosisSafe(safe).execTransaction(
        address(target), // to
        0, // value (Here lies the issue: This is a payable function, but we are passing 0 Ether)
        data, // data
        op, // operation
        0, // safeTxGas
        0, // baseGas
        0, // gasPrice
        address(0), // gasToken
        payable(address(0)), // refundReceiver
        _generateSingleThresholdSignature(address(this)) // signatures
    );

    if (!success) revert SafeExecTransactionFailed();
}
```

### Recommended Mitigation Steps

Correctly query `IGnosisSafe(safe).execTransaction()` and pass value.

## QA-07 Potential Signature Bypass with Empty Signatures

### Impact

**Medium to Low** If the `trustedValidator` is an EOA (Externally Owned Account) and the `validatorSignature` is empty, the function will revert, making it possible for a transaction without a valid signature to get processed.

### **Proof of Concept**

In the `isPolicySignatureValid()` function, the following check is made:

```solidity
if (trustedValidator.code.length == 0 && validatorSignature.length == 0) {
    // TrustedValidator is an EOA and no trustedValidator signature is provided
    revert InvalidSignature();
}
```

This logic checks if the `trustedValidator` is an EOA and if the provided `validatorSignature` is empty. However, this poses a security risk because if for some reason the `validatorSignature` isn't provided or is forgotten, the transaction will still pass through without a valid signature, potentially causing unauthorized changes or actions.

### Recommended Mitigation Steps

Always ensure that a valid `validatorSignature` is provided. Consider strengthening signature validation checks and ensuring that the system has multiple layers of validation to prevent potential misuse. Additionally, a more comprehensive comment or documentation on the expected behavior when dealing with EOAs and their signatures would be beneficial.

## QA-08 Add a function to remove a wallet/subAccount

### Impact

Low info, but in the case where there is a need to remove an authorized addresses, this would not work since no functionality regarding this has been provide

### Proof Of Concept

The wallets and subaccounts can be registered but not deregistered. This
can be potentially an issue in case of some disaster to keep the console
secure.

### Recommended Mitigation Steps

Add a function to remove a wallet

## QA-09 Documentation Should Reference Active Links

### Impact

**Low:** Proper links enhance code comprehension and traceability.

### Proof Of Concept

Observe the following:

```solidity
    /**
     * @notice Generates a pre-validated signature for a safe transaction
     * @dev Refer to https://docs.safe.global/learn/safe-core/safe-core-protocol/signatures#pre-validated-signatures
     * This calldata assumes that owner is the actual address that will be sending the execTransaction call to safe
     * @param owner Owner of the safe
     * @return signatures bytes array containing single pre validated owner signature
     */
    function _generateSingleThresholdSignature(address owner) internal pure returns (bytes memory) {
```

Accessing the provided link, `https://docs.safe.global/learn/safe-core/safe-core-protocol/signatures#pre-validated-signatures`, leads to a `404 Error`.

![](https://private-user-images.githubusercontent.com/107410002/276356701-9e02cef8-6819-404e-8dc2-71614b696902.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTEiLCJleHAiOjE2OTc2NTMzMTMsIm5iZiI6MTY5NzY1MzAxMywicGF0aCI6Ii8xMDc0MTAwMDIvMjc2MzU2NzAxLTllMDJjZWY4LTY4MTktNDA0ZS04ZGMyLTcxNjE0YjY5NjkwMi5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBSVdOSllBWDRDU1ZFSDUzQSUyRjIwMjMxMDE4JTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDIzMTAxOFQxODE2NTNaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT05ZDEwY2VlZDZkNDJiNWVhNjlmMzcyYjEyODM4NGVlY2Y1YTI1M2M2OWFmMWZlMTRlYzJjOTExNTYxOGM2Y2IzJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCZhY3Rvcl9pZD0wJmtleV9pZD0wJnJlcG9faWQ9MCJ9.gDnmzEcsPhKXkscX203Fr61eMVAWrlBCJCOSRfMOUNQ)

### Recommended Mitigation Steps

Replace the broken link with the accurate documentation URL.

## QA-10 Unsafe Downcasting without Proper Checks

### Impact

**Low** The unchecked conversion of `block.timestamp` from `uint256` to `uint32` can result in an incorrect value if `block.timestamp` exceeds the maximum value of `uint32`. This could result in unexpected behavior where signatures that should have expired are still considered valid.

### Proof of Concept

In the function `isPolicySignatureValid()`, the code checks if the `expiryEpoch` is less than the current `block.timestamp`:

```solidity
if (expiryEpoch < uint32(block.timestamp)) {
    revert TxnExpired(expiryEpoch);
}
```

Here, `block.timestamp` is explicitly cast to `uint32`. If the `block.timestamp` (which is a `uint256` by default) exceeds the `uint32` limit, the cast will truncate the value, leading to unexpected results. This can particularly cause an issue where an expired signature will not be flagged as expired.

### Recommended Mitigation Steps

Implement a safe casting mechanism to ensure that the `block.timestamp` is within the valid range of `uint32` before attempting the cast. Also, consider if the chosen datatype `uint32` for `expiryEpoch` is appropriate, given the potential lifespan and requirements of the contract.

## QA-11 Add `STATICCALL` Support in `_parseOperationEnum()`

### Impact

Low

### Proof of Concept

Take a look at `_parseOperationEnum()`

```solidity
/**
 * @notice Converts a CallType enum to an Operation enum.
 * @dev Reverts with UnableToParseOperation error if the CallType is not supported.
 * @param callType The CallType enum to be converted.
 * @return operation The converted Operation enum.
 */
function _parseOperationEnum(Types.CallType callType) internal pure returns (Enum.Operation operation) {
    if (callType == Types.CallType.DELEGATECALL) {
        operation = Enum.Operation.DelegateCall;
    } else if (callType == Types.CallType.CALL) {
        operation = Enum.Operation.Call;
    } else {
        // @audit Fails to handle STATICCALL type.
        revert UnableToParseOperation();
    }
}
```

As seen, the function `_parseOperationEnum()` is intended to convert a `CallType` enum to an `Operation` enum. However, it does not provide support for the `STATICCALL` type. This lack of support might lead to incomplete functionality or unintended behaviour if the `STATICCALL` type is used in other parts of the contract or in any future implementations. Users and developers might expect this conversion to be supported, given that other call types are.

### Recommended Mitigation Steps

Modify the `_parseOperationEnum()` function to handle the `STATICCALL` type. This will likely involve updating the `Enum.Operation` enum to include a `StaticCall` type if it does not already exist.
