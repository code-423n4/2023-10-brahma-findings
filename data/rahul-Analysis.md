## Analysis summary
The Brahma Protocol (`Consolev2` release) was analysed for common design flaws, such as access control, arithmetic issues, front-running, denial of service, race conditions, and protocol manipulations. 

Special attention was paid to custom signature packing, possibility of transaction replay across chains, signature malleability, authorization bypass, and privilege escalations. 

## Protocol Overview
Brahma protocol aims to be a non-custodial abstraction over Safe protocol. The protocol deploy modular, and secure components to User's safe wallet which can offer varied levels of automation of common DeFi tasks. This approach avoids the need for users to roll out their own, potentially insecure Safe modules. 

A background of the Safe protocol is provided in Appendix A.
## Architecture overview

An excellent overview of the contracts / modules [is provided as part the documentation](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/docs/Core.md). For the sake of brevity, it is not intentionally covered again here. 

![Architecture](https://i.imgur.com/bn0V01s.png "Architecture")

### Authorization matrix for transaction types

| Transaction type                          | Console owner(s) signature | Operator(s) signature | Trusted validator signature | Executor plugin | Executor signature |
| ----------------------------------------- | -------------------------- | --------------------- | --------------------------- | --------------- | ------------------ |
| Console account via owner(s)              | Yes                        | \-                    | No                          | \-              | \-                 |
| Console account w/ policy                 | Yes                        | \-                    | Yes (1x)                    | \-              | \-                 |
| Sub-account via operator(s)               | \-                         | Yes                   | Yes (1x)                    | \-              | \-                 |
| Sub-account via executor plugin           | \-                         | \-                    |                             | Yes             | Yes                |
| Sub-account via Console account           | Yes                        | No                    | Yes (1x)                    | \-              | No                 |
| Sub-account via Console account w/ policy | Yes                        | No                    | Yes (2x)                    | \-              | No                 |

Policy commitements for the corresponding transaction types:

| Transaction type                          | Console policy enabled? | Sub-account policy enabled? |
| ----------------------------------------- | ----------------------- | --------------------------- |
| Console account via owner(s)              | No                      | \-                          |
| Console account w/ policy                 | Yes                     | \-                          |
| Sub-account via operator(s)               | \-                      | Yes                         |
| Sub-account via executor plugin           | \-                      | Yes                         |
| Sub-account via Console account           | No                      | Yes                         |
| Sub-account via Console account w/ policy | Yes                     | Yes                         |

## Comments on code

The codebase exhibits a high level of maturity in terms of quality and documentation. The test suite is comprehensive, with test coverage approaching an ideal level. Notably the test practices demonstrates exceptional use of [Paul Berg's Branch Tree Testing standard](https://www.youtube.com/watch?v=V6KBy8QQnCo).

Following comments highlight scope of suggested improvements:
### [A-01] Incorrect error message during policy signature verification
When the `execTransaction` function on a console account with a valid policy commit is invoked without a validator signature, the behavior is not consistent with expectations. Instead of throwing an `InvalidSignatures` exception, `PolicyValidator.sol` throws a `TxnExpired` exception.
#### POC test case 
```solidity
    function test_Transaction_ConsoleAccount_WithPolicy() public {
        address consoleAccount =
            safeDeployer.deployConsoleAccount(_getConsoleOwners(), 1, hex"C0FFEE", keccak256("salt"));

        IGnosisSafe(consoleAccount).execTransaction(
            address(mockProtocol),
            500,
            abi.encodeWithSelector(mockProtocol.deposit.selector),
            Enum.Operation.Call,
            0,
            0,
            0,
            address(0),
            payable(0),
            SafeHelper._generateSingleThresholdSignature(_getConsoleOwners()[0])
        );
    }
```
#### Recommendation
A mechanism needs to be devised to check if validator signature is appended to `_signatures`. One possible solution could be to append a magic value to `_signatures` after `expiryEpoch` indicating that appended signature belongs to a trusted validator.

```diff
	Source: src/core/PolicyValidator.sol:156
	
    function _decompileSignatures(bytes calldata _signatures)
        internal
        pure
        returns (uint32 expiryEpoch, bytes memory validatorSignature)
    {
+       bytes4 VALIDATOR_SIG_MAGIC_VALUE = hex"ffffffff"; 
  
        uint256 length = _signatures.length;
        if (length < 8) revert InvalidSignatures();

+       bytes4 magicValue = bytes4(_signatures[length - 4:length]);
+       if (magicValue != VALIDATOR_SIG_MAGIC_VALUE) revert InvalidSignatures();
	    
		 __SNIP__
    }

```

[Source file on Github: PolicyValidator.sol#L156-L167](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/PolicyValidator.sol#L156-L167)
### [A-02] Incorrect decompilation of validator signature
Issue [A-01] has a ripple effect that extends beyond simply triggering an incorrect custom error message. The problem arises from the fact that `_decompileSignature` fails to revert when a validator signature is absent. Consequently, the function incorrectly interprets a "Safe" signature as a "Validator" signature. This issue introduces a vulnerability that can affect the proper functioning of all references to this function, and it remains a potential problem even in future releases.

Addressing this issue is strongly advised in order to prevent unintended behavior.
### [A-03] Incorrect error message during executor signature verification

If the signature provided by a registered executor is incorrect, `ExecutorPlugin` throws an `InvalidExecutor` exception rather than the expected `InvalidSignature` exception. This deviation from the anticipated behavior should to be addressed.
#### Recommendation
Replace with the correct custom error:

```diff
	Source: src/core/ExecutorPlugin.sol:139
	
        if (
            !SignatureCheckerLib.isValidSignatureNow(
                execRequest.executor, _transactionDigest, execRequest.executorSignature
            )
        ) {
-            revert InvalidExecutor();
+            revert InvalidSignature()
        }
```

[Source file on Github: ExecutorPlugin.sol#L139-L145](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ExecutorPlugin.sol#L139-L145)

### [A-04] Use of context appropriate functional selector for setting guards

As stated in the documentation and comments, `SafeEnabler` is designed to enable modules and guard for a Safe. 

``` Solidity
/**
__SNIP__
 * @notice Contract which holds bytecode to enable modules and guards for a Gnosis Safe
__SNIP__
contract SafeEnabler is GnosisSafeStorage {

    /**
     * @notice Sets the guard for a safe
     * __SNIP__
     */
    function setGuard(address guard) public {
	     __SNIP__
    }
}

```

Yet, `SafeDeployer` uses selector from `IGnosis` interface to create guard. Consider using selector from `SafeEnabler` instead to make intent clear and safe guard against changes to `Gnosis` interface in a future release.
#### Recommendation
```diff
Source: src/core/SafeDeployer.sol:(128,189)
    function _setupConsoleAccount(address[] memory _owners, uint256 _threshold, bool _policyHashValid)
        private
        view
        returns (bytes memory)
    {
		__SNIP__
                data: abi.encodeCall(
-                   IGnosisSafe.setGuard,   (AddressProviderService._getAuthorizedAddress(_SAFE_MODERATOR_OVERRIDABLE_HASH)
+                   SafeEnabler.setGuard,   (AddressProviderService._getAuthorizedAddress(_SAFE_MODERATOR_OVERRIDABLE_HASH))
                    )
	     __SNIP__
    }
    
    __SNIP__
    
        function _setupSubAccount(address[] memory _owners, uint256 _threshold, address _consoleAccount)
        private
        view
        returns (bytes memory)
    {

    __SNIP__
-           data: abi.encodeCall(IGnosisSafe.setGuard, (AddressProviderService._getAuthorizedAddress(_SAFE_MODERATOR_HASH)))
+           data: abi.encodeCall(SafeEnabler.setGuard, (AddressProviderService._getAuthorizedAddress(_SAFE_MODERATOR_HASH)))
        });
    __SNIP__
    }
    
 __SNIP__
```
- [Source file on Github: SafeDeployer.sol#L128](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeDeployer.sol#L128) 
- [Source file on Github: SafeDeployer.sol#L189](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeDeployer.sol#L189) 

### [A-05] Consider creating an Index of addresses implementing `IAddressProviderService`

In the `AddressProvider.sol` contract, the `setAuthorizedAddress` function currently relies on the `_overrideCheck` flag to determine whether the provided address requires verification for implementing the `IAddressProviderService`. This approach has a potential risk, as governance might unintentionally skip checking important addresses by providing the wrong flag.

An alternative solution is to maintain a mapping of `_key` to `_overrideCheck`, which can be updated via governance. This adjustment would eliminate the need for the flag, simplifying the process of changing interface implementations and streamlining the setup for testing.

### [A-06] Inconsistent use of variable names and components

There is some variability in the naming conventions for function and event parameter names. Some of them begin with an underscore, while others do not. While both approaches are technically valid, it is advisable to adhere to a consistent coding style and to thoroughly document this style for the benefit of the entire team.

Additionally, within the project, there is occasional interchangeability between terms like "Wallet," "Safe/Console Account," and "Sub Safe/Sub-account." To ensure clarity and prevent confusion, it is recommended to select a suitable terminology and maintain consistent usage throughout the project.
## Centralization risk
The role of the trusted validator serves as a secondary layer of authorization for accounts with policy commits enabled. However, considering that the trusted validator is governed centrally and is a shared entity for all accounts, it raises concerns about potential centralization risks.

In the event of a denial of service caused by trusted validator, the consequences could be notably disruptive, particularly since all sub-accounts are mandated to have policy commitments. Such an interruption could lead to transaction delays or denial of transactions altogether.
## Systemic risk due to dependency on Safe Protocol

Although the project aims to be a non-custodial platform User's safe wallet. It should be noted that operational disruptions are ultimately a significant worry. The unavailability or compromise of the Safe protocol could lead to transaction delays, complications in asset withdrawals, and other essential functions, ultimately eroding user confidence and potentially impacting the project's reputation and revenue. 

To mitigate these risks effectively, the project should establish contingency plans if Safe Protocol becomes unavailable, and maintain transparent communication with users regarding potential risks and the steps taken to address them.

## Appendix A:  Background of Safe Protocol
### Safe wallet
Ethereum transactions are signed by a single Externally-owned account (EOA). This can become a challenge when transaction execution requires approval from multiple stake holders. An example could be a governance proposal execution that requires approvals from core team members.

Safe wallet is smart contract that emulates a wallet that requires multiple signatures. It allows users to configure a minimum number of signers required, known as threshold, to execute a transaction.
### Executing transactions
There are two methods available to execute a transaction:
1. `executeTransaction` executes transaction using signers.
2. `executeTransactionFromModule` executes transaction using Safe module. This function checks that the `msg.sender` is an enabled Safe module for this wallet.
### Safe modules
Safe Modules are smart contract extensions that are added to a Safe wallet by its signers. Modules have root access to the safe wallet - which means they can execute arbitrary transactions without approval of the signers. Safe modules can  be used as a recovery mechanism to replace the original signers of the safe wallet if their private keys are lost. 

>[!Warning]
>Safe Modules can be a security risk since they can execute arbitrary transactions. Only add trusted and audited modules to a Safe. A malicious module can take over a Safe.

A basic Safe does not require any modules. Safe modules extend the functionality of a safe wallet. They can be called by third party Externally-owned account (EOA) to execute transactions from safe wallet.
### Safe guards
Safe guards are hooks that fire before and after the execution of transaction from a Safe wallet.This allows for programmatic checking of pre-conditions or verification of transaction execution. If any of the check fails, the transaction is reverted.
## Appendix B: Analysis methodology
The evaluation process involves two key phases: **Understanding** and **Review**.

In the **Understanding** phase, significant effort is dedicated to comprehending the project's background. This involves in-depth analysis of project documentation, prior releases, and similar protocols to grasp the project's core value proposition. Furthermore, this phase strives to identify critical risks, assess user journeys, and evaluate the project's security posture, including factors such as test coverage.

The **Review** phase involves a systematic assessment of the project's architecture. It aims to pinpoint code duplications, highlight inconsistencies, recommend the implementation of best practices, and identify areas for potential optimization. This comprehensive approach to project evaluation ensures that not only potential security risks are addressed but also that the project's overall structure and code quality are scrutinized for enhancements.



### Time spent:
43 hours