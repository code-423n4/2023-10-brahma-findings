# QA Report - Brahma Audit Contest | 13 Oct 2023 - 20 Oct 2023

---

# Executive summary

### Overview

|              |                                              |
| :----------- | :------------------------------------------- |
| Project Name | Brahma                                       |
| Repository   | https://github.com/code-423n4/2023-10-brahma |
| Website      | [Link](https://www.brahma.fi/)               |
| Twitter      | [Link](https://twitter.com/BrahmaFi)         |
| Methods      | Manual Review                                |
| Total nSLOC  | 833 over 16 contracts                        |

---

---

# Findings Summary

| ID              | Title                                                                                              | Instances | Severity       |
| --------------- | -------------------------------------------------------------------------------------------------- | --------- | -------------- |
| [L-01](#L-01)   | Bypassing of check in `_ensureAddressProvider()` function                                          | -         | _Low_          |
| [L-02](#L-02)   | Console Account Creation Without Policies Leads to Enforcement Challenges                          | -         | _Low_          |
| [NC-01](#NC-01) | Add `isWallet` check for provided wallet address in `WalletRegistry#registerSubAccount()` function | -         | _Non Critical_ |
| [NC-02](#NC-02) | Outdated Documentation                                                                             | -         | _Non Critical_ |
| [NC-03](#NC-03) | Consider to Implement Maximum `SubAccount` Limit                                                   | -         | _Non Critical_ |
| [NC-04](#NC-04) | Inconsistency in using `Enumerable.AddressSet` and `address[]`                                     | -         | _Non Critical_ |
| [NC-05](#NC-05) | Inconsistency in Checking Ownership in `ExecutorRegistry.sol`                                      | -         | _Non Critical_ |
| [NC-06](#NC-06) | Immutability of Registry Addresses in AddressProvider                                              | -         | _Non Critical_ |
| [NC-07](#NC-07) | Inability for Console Account to Remove `policyCommit` for his `SubAccounts`                       | -         | _Non Critical_ |
| [NC-08](#NC-08) | Adding of Additional Checks in `After Execution Functions` in the Protocol                         | -         | _Non Critical_ |

---

---

# <a name="L-01">Bypassing of check in `_ensureAddressProvider()` function</a>[L-01]

## GitHub Links

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L77-L90
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L97-L105

## Impact

- Bypassing of Validation Check during Important Functions execution.
- Potentially allowing unauthorized addresses to be added to the `authorizedAddresses` mapping, gaining access to functionalities within the protocol that they should not have.. (kind of centralization risk)

## Explanation

The `AddressProvider` contract serves as a central source for resolving addresses of core components and external contracts within the Brahma DeFi Protocol. It provides governance-controlled mechanisms for setting authorized addresses and registry addresses used within the protocol.
The `setRegistry()` and `setAuthorizedAddress()` functions in the `AddressProvider` contract serve important roles in managing the addresses of core components and authorized contracts within the Brahma DeFi Protocol.

The check in `_ensureAddressProvider()` function during `setRegistry()` and `setAuthorizedAddress()` functions execution in `AddressProvider.sol` contract can be bypassed.

## Proof of Concept

1. `setRegistry()` function allows the governance of the `AddressProvider` contract to set the address of a core component or registry by specifying a unique `_key` associated with that component. The `_key` is used as a reference to access the registered address later.

2. `setAuthorizedAddress()` function allows the governance to set an authorized address that can be used within the protocol. Authorized addresses are associated with a unique `_key` and are stored in the `authorizedAddresses` mapping.

Both functions utilize the `_ensureAddressProvider()` function, which ensures that the new address supports the `AddressProviderService` interface and is pointing to the `AddressProvider` contract.

However, this function doesn't provide adequate protection. The check within the `_ensureAddressProvider()` function can be bypassed by calling the `setRegistry()` function with the `_registry` parameter set to a contract that simply inherits `IAddressProviderService` and has implemented the `addressProviderTarget()` function, returning the address of the `AddressProvider` contract. The same applies to the `setAuthorizedAddress()` function with the `_authorizedAddress` parameter.

```jsx
    function setAuthorizedAddress(bytes32 _key, address _authorizedAddress, bool _overrideCheck) external {
        _onlyGov();
        _notNull(_authorizedAddress);

        /// @dev skips checks for supported `addressProvider()` if `_overrideCheck` is true
        if (!_overrideCheck) {
            /// @dev skips checks for supported `addressProvider()` if `_authorizedAddress` is an EOA
            if (_authorizedAddress.code.length != 0) _ensureAddressProvider(_authorizedAddress);
        }

        authorizedAddresses[_key] = _authorizedAddress;

        emit AuthorizedAddressInitialised(_authorizedAddress, _key);
    }
```

```jsx
    function setRegistry(bytes32 _key, address _registry) external {
        _onlyGov();
        _ensureAddressProvider(_registry);

        if (registries[_key] != address(0)) revert RegistryAlreadyExists();
        registries[_key] = _registry;

        emit RegistryInitialised(_registry, _key);
    }
```

### Coded Proof of Concept

```jsx
// Malicious contract attempting to bypass the _ensureAddressProvider check
contract MaliciousContract {
    function addressProviderTarget() external view returns (address) {
        // Return the AddressProvider contract address
        return address(0xAddressProviderContractAddress);
    }
}
```

### Tools Used

- Manual Code Review

## Recommended Mitigation Steps

Implement Additional Checks: Consider implementing additional checks or validation mechanisms to verify the authenticity and authorization of addresses being added as authorized addresses.

---

# <a name="L-02">Console Account Creation Without Policies Leads to Enforcement Challenges</a>[L-02]

## Summary:

Creating `Console Accounts` in the Brahma protocol with zero policies can lead to unexpected issues when attempting to enforce policies. The `SafeDeployer.sol` contract allows for the creation of `Console Accounts` without policies.

- [deployConsoleAccount()](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L56-L71) function:

```jsx
    function deployConsoleAccount(address[] calldata _owners, uint256 _threshold, bytes32 _policyCommit, bytes32 _salt)
        external
        nonReentrant
        returns (address _safe)
    {
        bool _policyHashValid = _policyCommit != bytes32(0);

        _safe = _createSafe(_owners, _setupConsoleAccount(_owners, _threshold, _policyHashValid), _salt);

        if (_policyHashValid) {
            PolicyRegistry(AddressProviderService._getRegistry(_POLICY_REGISTRY_HASH)).updatePolicy(
                _safe, _policyCommit
            );
        }
        emit ConsoleAccountDeployed(_safe);
    }
```

## Details:

The Brahma protocol relies on `Console Accounts`, which can be either EOA (Externally Owned Account) or `Safe Multisig Accounts`, to manage `Safe Sub Accounts` with configured policies. In the case of `Safe Multisig Accounts`, policies can be enforced on the `Console Account` using the `SafeModeratorOverridable` guard. However, the `SafeDeployer` contract permits the creation of `Safe Console Accounts` with zero policies, resulting in unexpected behavior.

When attempting to enforce a policy on a `Console Account` with zero policies, it's required to enable the `SafeModeratorOverridable` guard. This operation is performed as a `Safe Multisig Transaction` without interactions with the Brahma protocol. As a consequence, the `SafeModeratorOverridable` guard can be enabled on a Console with zero policies.

Policies are checked using an external trusted validator, and the signature of the validator is verified by the `PolicyValidator` contract. However, this contract reverts if the policy is not set (i.e., zero policy), leading to a denial of service for the console account.

```jsx
    function isPolicySignatureValid(address account, bytes32 transactionStructHash, bytes calldata signatures)
        public
        view
        returns (bool)
    {
        // Get policy hash from registry
        bytes32 policyHash =
            PolicyRegistry(AddressProviderService._getRegistry(_POLICY_REGISTRY_HASH)).commitments(account);
        if (policyHash == bytes32(0)) {
            revert NoPolicyCommit();
        }

        // ...more code...
```

Exploit Scenario:

1. Owners of a `Console Account` aim to enforce a new policy while the current policy is set to zero.
2. The owners enable the `SafeModeratorOverridable` guard.
3. They attempt to set the new policy, but the transaction reverts because changing a policy requires the signature of the trusted validator, which fails due to the zero policy.

Recommendation:
It is recommended to consider requiring a non-zero policy when creating a console account. Alternatively, provide a helper function that enables the SafeModeratorOverridable guard and requires a policy to be set. This would prevent the unexpected issues arising from `Console Accounts` with zero policies.

---

---

# <a name="NC-01">Add `isWallet` check for provided wallet address in `WalletRegistry#registerSubAccount()` function</a>[NC-01]

Currently the `registerSubAccount()` function in `WalletRegistry.sol` contract can be called only from `SafeDeployer.sol` contract. This is done in `SafeDeployer#deploySubAccount()` where the function check if the `msg.sender` is a registered wallet:

```jsx
    function deploySubAccount(
        address[] calldata _owners,
        uint256 _threshold,
        bytes32 _policyCommit,
        bytes32 _salt
    ) external nonReentrant returns (address _subAcc) {

        // ...code...

        // Check if msg.sender is a registered wallet
        WalletRegistry _walletRegistry = WalletRegistry(
            AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH)
        );
        if (!_walletRegistry.isWallet(msg.sender)) revert NotWallet();

        // ...code...

        // Register sub account to wallet
        _walletRegistry.registerSubAccount(msg.sender, _subAcc);

        // ...code...
}
```

Future the logic in `WalletRegistry#registerSubAccount()` function can be changed and maybe allow to be called not only from `SafeDeployer` contract.

So, It's recommended to implement check in `WalletRegistry#registerSubAccount()` function if the provided `wallet` address is registered wallet.

#### GitHub Links:

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L49-L55
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L82-L103

---

# <a name="NC-02">Outdated Documentation[NC-02]</a>

Update the documentation for the 'Console Guard Removal Transaction' section, which can be found at [link](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/docs/Architecture.md#console-guard-removal-transaction), to align with the latest code implementation of the Brahma Protocol.

Currently, the 'Console Guard Removal Transaction' section includes a schema illustrating the flow of the 'Console Guard Removal Transaction.' However, it references the `_isGuardBeingRemoved()` function, which no longer exists in the code. Instead, the code now implements the `_isConsoleBeingOverridden()` function with some logic changes.

Make the necessary updates to the documentation to reflect these changes accurately.

```jsx
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
            if (SafeHelper._GUARD_REMOVAL_CALLDATA_HASH == keccak256(_data)) {
                return true;
            } else if (SafeHelper._FALLBACK_REMOVAL_CALLDATA_HASH == keccak256(_data)) {
                return true;
            }
        }

        return false;
    }
```

Link: https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L149-L174

#####

Reference: L-01 issue from previous Brahma audit (https://github.com/Brahma-fi/brahma-security/blob/master/audits/brahma-fi-consolev2-audit-10-23-ackee.pdf).

---

# <a name="NC-03">Consider to Implement Maximum `SubAccount` Limit</a>[NC-03]

The Brahma Protocol currently lacks a maximum limit on the number of sub-accounts that a wallet can create. This omission raises concerns about scalability, complexity, resource consumption, and security. It is recommended to establish a reasonable maximum limit for sub-accounts to address these issues.

**Details:**

1. **Scalability Concerns:** The absence of a sub-account limit may hinder protocol scalability, impacting performance.

2. **Complexity:** Managing numerous sub-accounts can become complex and may lead to operational and security challenges.

3. **Resource Usage:** Excessive sub-accounts consume blockchain resources, potentially affecting overall efficiency.

4. **Security Risks:** A large number of sub-accounts increase the attack surface, potentially compromising security.

**Recommendation:**

Implement a well-defined maximum limit on sub-accounts for each wallet to enhance scalability, security, and user experience.

**Benefits:**

- **Improved Scalability:** Efficient management of sub-accounts enhances scalability.
- **Enhanced Security:** Limiting sub-accounts reduces security risks.
- **User-Friendly:** Simplifies account management for users.
- **Resource Efficiency:** Optimizes resource allocation.
- **Future-Proofing:** Prepares for future growth and challenges.

**Conclusion:**

## Introducing a maximum sub-account limit in Brahma Protocol is crucial for its scalability, security, and user-friendliness, ensuring readiness for future growth.

---

# <a name="NC-04">Inconsistency in using `Enumerable.AddressSet` and `address[]`</a>[NC-04]

**Links**

- ExecutorRegistry: https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol
- WalletRegistry: https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol

**Description:**

In the Brahma Protocol smart contracts, specifically in the `ExecutorRegistry.sol` and `WalletRegistry.sol` contracts, there is an inconsistency in the data structures used for storing related information.

**Examples:**

1. In the `ExecutorRegistry.sol` contract, the mapping `subAccountToExecutors` is used to associate sub-accounts with a set of executor addresses using `EnumerableSet.AddressSet`. This design choice ensures efficient membership checks and avoids duplicate entries.

   Example:

   ```jsx
   mapping(address subAccount => EnumerableSet.AddressSet) private subAccountToExecutors;
   ```

2. On the other hand, in the `WalletRegistry.sol` contract, the mapping `walletToSubAccountList` is used to map wallet addresses to an array of sub-account addresses. Here, a simple `address[]` array is used to store sub-account addresses.

   Example:

   ```jsx
   mapping(address wallet => address[] subAccountList) public walletToSubAccountList;
   ```

**Impact:**

This inconsistency in data structure usage may lead to differences in contract behavior and may not fully leverage the advantages of `Enumerable.AddressSet`. It could potentially impact contract efficiency, readability, and ease of use.

**Recommendation:**

For consistency and possibly better contract design, consider using `Enumerable.AddressSet` consistently in both contracts (`ExecutorRegistry.sol` and `WalletRegistry.sol`) if the functionality allows. This will maintain a uniform approach to handling sets of addresses, provide efficient membership checks, and ensure clear and predictable behavior throughout the protocol.

---

# <a name="NC-05">Inconsistency in Checking Ownership in `ExecutorRegistry.sol`</a>[NC-05]

**Links:**

- ExecutorRegistry: https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol

**Description:**

In the Brahma Protocol smart contract `ExecutorRegistry.sol`, there is an inconsistency in checking whether `msg.sender` is the owner of a sub-account. This inconsistency exists in the `registerExecutor()` and `deRegisterExecutor()` functions.

**Details:**

1. In the `registerExecutor()` function, the contract checks if `msg.sender` is the owner of the sub-account using the following code:

   ```solidity
   WalletRegistry _walletRegistry = WalletRegistry(
       AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH)
   );
   if (!_walletRegistry.isOwner(msg.sender, _subAccount))
       revert NotOwnerWallet();
   ```

   Here, it verifies ownership by calling `isOwner(msg.sender, _subAccount)` on the `WalletRegistry` contract.

2. In the `deRegisterExecutor()` function, the contract checks ownership using the following code:

   ```solidity
   WalletRegistry _walletRegistry = WalletRegistry(
       AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH)
   );
   if (_walletRegistry.subAccountToWallet(_subAccount) != msg.sender)
       revert NotOwnerWallet();
   ```

   In this case, it checks ownership by comparing `msg.sender` to `subAccountToWallet(_subAccount)` from the `WalletRegistry` contract.

**Impact:**

This inconsistency in checking ownership can lead to confusion and potential issues during contract interactions. Users may expect consistent ownership verification methods, but the contract uses different approaches in different functions.

**Recommendation:**

To maintain consistency and clarity in ownership checks, it is advisable to use a standardized method to verify ownership throughout the contract. You can choose one of the existing methods, such as `isOwner(msg.sender, _subAccount)`, and use it consistently in both the `registerExecutor()` and `deRegisterExecutor()` functions.

Updating the code to use a single ownership verification method will help prevent potential misunderstandings and make the contract more user-friendly. Additionally, ensure that this change is well-documented in the contract's comments or documentation to provide clear guidance to users.

---

# <a name="NC-06">Immutability of Registry Addresses in AddressProvider</a>[NC-06]

## Description

The AddressProvider contract enforces the immutability of registry addresses, meaning that once set, these addresses cannot be altered. This design choice has both positive and negative implications.

On the positive side, it enhances the trust model of the system as these addresses remain immutable, reducing the risk of malicious changes. However, in the unfortunate event of a disaster, this immutability becomes a limitation as it hinders potential recovery efforts for the protocol.

## Recommendation

Users should be made aware of the consequences of this design choice, particularly its impact on the trust model. It is essential to provide clear information about the trade-offs between immutability and disaster recovery to ensure informed decision-making.

#### Link:

- AddressProvider: https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L1-L150

---

# <a name="NC-07">Inability for Console Account to Remove `policyCommit` for his `SubAccounts`</a>[NC-07]

## Links:

- PolicyRegistry: https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L1-L70

## Description

The `PolicyRegistry` manages policy commits per account. The policy can be updated

- if the commit is zero and the msg.sender is `SafeDeployer`
- if the `msg.sender` is the owner of the account that is going to have an updated policy
- if the `msg.sender` is a wallet and is updating the policy for itself.

In the `PolicyRegistry` contract of the Brahma Protocol, there exists a limitation where the **`commitments` for SubAccounts cannot be set to zero (removed) if the `msg.sender is ConsoleAccount/Wallet` that is actually the owner of the SubAccount**. This behavior is due to a check in the `updatePolicy()` function that reverts if `policyCommit` is set to `bytes32(0)`. As a result, **Console Accounts cannot remove (set to zero) the `commitments` data for their SubAccounts**, which may not be the expected behavior.

## Contract Code

- [updatePolicy()](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L35-L59) function:

```jsx
function updatePolicy(address account, bytes32 policyCommit) external {
    if (policyCommit == bytes32(0)) {
        revert PolicyCommitInvalid();
    }

    // ... (other logic)

        } else if (walletRegistry.isOwner(msg.sender, account)) {
            //In case invoker is updating on behalf of sub account
        }

    // ... (other logic)


    _updatePolicy(account, policyCommit);
}
```

## Impact

The impact of this issue is that `commitments` data of SubAccounts cannot be removed or set to zero bytes data by the ConsoleAccount/Wallet. This means that once a `policyCommit` is set for a SubAccount, it cannot be effectively revoked or cleared. While this might be the intended behavior for some use cases, it limits the flexibility of Console Accounts to manage the policies of their SubAccounts.

## Recommendation

Consider to allow `bytes(0) for commitments[account]` to be set only if the `msg.sender` is the `Console Account/Wallet` that is going to change the `commitments` data for his `SubAccount`.

---

# <a name="NC-08">Adding of Additional Checks in `After Execution Functions` in the Protocol</a>[NC-08]

## Description

Consider to enhance security by adding additional checks in the `After Execution Functions` in the Protocol for various scenarios. These checks will help ensure the integrity and security of the protocol.

Example of validation checks that can be added in **After Execution Functions**:

- If the Console Account Owners are changed (this is especially important because the _`nonce` that is created during ConsoleAccount Deployment is calculated with the `Console Account Owners Hash`_ )
- If the Policy Signature is for ConsoleAccount or SubAccount is changed

## After Execution Functions

- **SafeModeratorOverridable.sol#checkAfterExecution()**: https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol#L76-L79
- **SafeModerator.sol#checkAfterExecution()**: https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModerator.sol#L70-L73
- **TransactionValidator.sol#validatePostTransactionOverridable()**: https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L81-L84
- **TransactionValidator.sol#validatePostTransaction()**: https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L105-L107
- **TransactionValidator.sol#validatePostExecutorTransaction()**: https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L132-L134

######

_Reference: Documentation Flow - https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/docs/Architecture.md_
