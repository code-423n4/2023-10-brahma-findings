# 1. `wallet` and `subAccount` can be the same when `registerSubAccount`
Description: 

When building connections between `wallet` and `subAccount`, there is no validation to ensure the given `wallet` is not equal to `subAccount`, as a result, the subaccount owner can be itself. This can be violated when `_SAFE_DEPLOYER_HASH` is not setup correctly.

Recommend:

Adding validation to ensure the `wallet` and `subAccount` are not equal

# 2. code optimization in `AddressProviderService`
Description:
 
Inside the constructor, the validation on zero address can reuse the code `notNull` to decrease the code complexity
```solidity
    constructor(address _addressProvider) {
        if (_addressProvider == address(0)) revert InvalidAddressProvider(); //@audit report: using notnull isntead
        addressProvider = AddressProvider(_addressProvider);
    }
```

Recommend:

Try reuse the code to decrease the code complexity.

# 3. Missing deadline information in current signature
Description:

Typically, it is advisable to establish a deadline for obtaining signatures and a mechanism for invalidating them, such as in the case of a compromised private key. However, these measures are currently absent.
```solidity
        bytes32 _transactionStructHash = TypeHashHelper._buildTransactionStructHash(
            TypeHashHelper.Transaction({
                to: execRequest.exec.target,
                value: execRequest.exec.value,
                data: execRequest.exec.data,
                operation: uint8(SafeHelper._parseOperationEnum(execRequest.exec.callType)),
                account: execRequest.account,
                executor: execRequest.executor,
                nonce: executorNonce[execRequest.account][execRequest.executor]++
            }) //@audit Report: not deadline information
        );

```

# 4. code optimization in `ExecutorRegistry`

Description:

The function `deRegisterExecutor` is querying `subAccountToWallet` directly instead using `isOwner`.

```solidity
    /**
     * @notice De-registers an executor for subaccount
     * @dev removes an executor if it exists else reverts with DoesNotExist()
     * @dev Can be only called by owner of subAccount
     * @param _subAccount subAcc address remove executor from
     * @param _executor executor to remove
     */
    function deRegisterExecutor(address _subAccount, address _executor) external {
        WalletRegistry _walletRegistry = WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH));
        if (_walletRegistry.subAccountToWallet(_subAccount) != msg.sender) revert NotOwnerWallet(); //@audit report: using isowner

        if (!subAccountToExecutors[_subAccount].remove(_executor)) revert DoesNotExist();
        emit DeRegisterExecutor(_subAccount, msg.sender, _executor);
    }
```

Recommend:

Recommend using `isOwner` instead to decrease code complexity.

