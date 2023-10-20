## [L-01] `_validatePolicySignature` function no checks validity of policy signatures

The TransactionValidator contract, which allows unauthorized parties to call the `_validatePolicySignature` function. This function is critical for verifying the validity of policy signatures, and as it stands, there are no access controls or checks in place to ensure that only authorized parties can invoke it. This opens the door to potential unauthorized transactions and poses a significant security risk.

```solidity
File: src/core/TransactionValidator.sol

   function _validatePolicySignature(
        address _from,
        address _to,
        uint256 _value,
        bytes memory _data,
        Enum.Operation _operation,
        bytes memory _signatures
    ) internal view {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L210C2-L217C22

## [L-02] `checkTransaction` function in SafeModeratorOverridable contract lacks comprehensive checks to verify the contract's state after a transaction is executed

The contract performs pre-transaction validation using the "checkTransaction" function, it lacks comprehensive checks to verify the contract's state after a transaction is executed. This limited post-transaction validation can lead to unforeseen issues and potentially compromise the contract's security.

```solidity
File: src/core/SafeModeratorOverridable.sol

    function checkTransaction(
        address to,
        uint256 value,
        bytes memory data,
        Enum.Operation operation,
        uint256 safeTxGas,
        uint256 baseGas,
        uint256 gasPrice,
        address gasToken,
        address payable refundReceiver,
        bytes memory signatures,
        address msgSender
    ) external view override {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol#L39C1-L51C31

so it is advisable to expand the post-transaction validation checks. This expansion should focus on ensuring that the contract remains in a consistent and secure state following the execution of transactions.

## [L-03] Unrestricted simulate Function Execution in ConsoleFallbackHandler.sol contract

The contract contains an unrestricted simulate function, which allows external parties to execute delegate calls to any target contract in the context of the contract itself. While this functionality can be legitimate and useful in certain cases, it poses a security risk when not properly controlled. Malicious actors may exploit this to execute arbitrary code and potentially harm the contract's integrity and user funds.

```solidity
File: src/core/ConsoleFallbackHandler.sol

    function simulate(address targetContract, bytes calldata calldataPayload)
        external
        returns (bytes memory response)
    {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L104C1-L160C10

## [L-04] Inadequate Gas Limit Check

he simulate function executes delegate calls to target contracts. However, it lacks a check for the gas limit, which could lead to out-of-gas errors during execution.
To prevent out-of-gas errors and ensure reliable execution, it is recommended to implement a gas limit check within the simulate function.

```solidity
           pop(
                call(
                    gas(),
                    // address() has been changed to caller() to use the implementation of the Safe
                    caller(),
                    0,
                    internalCalldata,
                    calldatasize(),
                    // The `simulateAndRevert` call always reverts, and
                    // instead encodes whether or not it was successful in the return
                    // data. The first 32-byte word of the return data contains the
                    // `success` value, so write it to memory address 0x00 (which is
                    // reserved Solidity scratch space and OK to use).
                    0x00,
                    0x20
                )
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L131C2-L146C18

## [L-05] There is no Check on Governance Transfer in AddressProvider.sol contract

The contract allows for the transfer of governance using the setGovernance and acceptGovernance functions. However, it lacks proper authorization checks during the governance transfer process. Any address can initiate a governance transfer using the setGovernance function, and the acceptGovernance function doesn't have an explicit check to ensure that the pending governance is a trusted address. This could lead to unauthorized changes in governance, potentially causing security and control issues.

```solidity
File:

    function setGovernance(address _newGovernance) external {
        _notNull(_newGovernance);
        _onlyGov();
        emit GovernanceTransferRequested(governance, _newGovernance);
        pendingGovernance = _newGovernance;
    }



        function acceptGovernance() external {
        if (msg.sender != pendingGovernance) {
            revert NotPendingGovernance(msg.sender);
        }
        emit GovernanceTransferred(governance, msg.sender);
        governance = msg.sender;
        delete pendingGovernance;
    }
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L52C1-L69C6
