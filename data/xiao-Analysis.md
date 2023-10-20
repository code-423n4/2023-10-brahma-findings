# 🕵️‍♀️ Analysis - Brahma

## Summary

| List | Head                                            | Details                                                                                |
| :--- | :---------------------------------------------- | :------------------------------------------------------------------------------------- |
| a)   | The approach I followed when reviewing the code | Stages in my code review and analysis                                                  |
| b)   | Analysis of the code base                       | What is unique? How are the existing patterns used?                                    |
| c)   | Test analysis                                   | Test scope of the project and quality of tests                                         |
| d)   | Centralization risks                            | How was the risk of centralization handled in the project, what could be alternatives? |
| e)   | Systemic risks                                  | Potential systemic risks in the project                                                |
| f)   | Competition analysis                            | What are similar projects?                                                             |
| g)   | Security Approach of the Project                | Audit approach of the Project                                                          |
| h)   | Other Audit Reports and Automated Findings      | What are the previous Audit reports and their analysis                                 |
|      |

## a) The approach I followed when reviewing the code

Determine the scope of code audit:
https://github.com/code-423n4/2023-10-brahma

| Number | Stage                | Details                                                                                                 | Information                                                                      |
| :----- | :------------------- | :------------------------------------------------------------------------------------------------------ | :------------------------------------------------------------------------------- |
| 1      | Compile and Run Test | [Installation](https://github.com/code-423n4/2023-10-brahma#tests)                                      | Test and installation structure is simple, cleanly designed                      |
| 2      | Architecture Review  | https://docs.brahma.fi/product/brahma-console                                                           | Understand the functions of each module                                          |
| 3      | Graphical Analysis   | Graphical Analysis with [vscode-solidity-auditor](https://github.com/Consensys/vscode-solidity-auditor) | Solidity language support and visual security auditor for Visual Studio Code     |
| 4      | Test Suits           | [Tests](https://github.com/code-423n4/2023-08-shell/tree/main/src/test)                                 | In this section, the scope and content of the tests of the project are analyzed. |
| 5      | Code Review          | [Scope](https://github.com/code-423n4/2023-10-brahma#scope)                                             | Top-down analysis of codes according to architectural design, IDE used: VsCode   |
| 6      | What Is Brahma?      | [Brahma](https://docs.brahma.fi/)                                                                       |                                                                                  |
| 7      | Infographic          | [Excalidraw](https://excalidraw.com/)                                                                   | I made Visual drawings to understand the hard-to-understand mechanisms           |

## b) Analysis of the code base
![Analysis](https://raw.githubusercontent.com/0x7869616F/img/main/b.png)
https://raw.githubusercontent.com/0x7869616F/img/main/b.png
## c) Test analysis

### Fork 2023-10-brahma/contracts/

| Number | Head                     | Test Details                                                                                                                                                                                |
| :----- | :----------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1)     | AddressProvider          | Governance replacement, setting and obtaining addresses, and handling of various error situations                                                                                           |
| 2)     | AddressProviderService   | Other contracts can easily resolve the addresses of other services through the "AddressProvider" and include the necessary error handling and permission checking mechanisms.               |
| 3)     | ConsoleFallbackHandler   | Handles signature verification and execution operations                                                                                                                                     |
| 4)     | ConsoleOpBuilder         | This contract is designed to generate bytecode for multiple operations in order to perform those operations in the Brahma Console account                                                   |
| 5)     | Constants                | Provides a centralized place to manage and reuse constants for multiple contracts                                                                                                           |
| 6)     | ExecutorPlugin           | Allows executors with module permissions to execute transactions and provides a mechanism to verify the legitimacy of execution requests                                                    |
| 7)     | PolicyValidator          | Verify policy signatures for secure contract transactions, ensuring that only verified signatures can execute transactions, thereby increasing security                                     |
| 8)     | SafeDeployer             | Manage and create Brahma Console accounts and sub-accounts and ensure their initial setup                                                                                                   |
| 9)     | SafeEnabler              | Provides a way to initialize and configure Gnosis Safe's modules and daemons, allowing certain checks to be bypassed during initialization for setup                                        |
| 10)    | SafeModerator            | The "SafeModerator" contract acts as a guard for Safe accounts, validating transactions and ensuring they comply with policy requirements                                                   |
| 11)    | SafeModeratorOverridable | Act as a guardian for Safe accounts, validating transactions and ensuring they comply with policy requirements. But unlike "SafeModerator", it allows rewriting by removing guards          |
| 12)    | ExecutorRegistry         | It allows the owner of a sub-account to register and unregister executors and provides functionality to check if a specific executor has been registered as an executor for the sub-account |
| 13)    | PolicyRegistry           | The "PolicyRegistry" contract allows different roles to update an account's policy submission based on a set of conditions.                                                                 |
| 14)    | WalletRegistry           | The "WalletRegistry" contract is used to manage and track the relationship between wallets and their subaccounts, as well as record which addresses are marked as wallets                   |

## d) Centralization risks

Users cannot provide their own main wallet to call the function of registering a sub-wallet, and can only be assigned by the project party. This is a centralization risk.
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L50C91-L50C91

In the same way, ` ExecutorRegistry.sol` and `PolicyRegistry.sol` are also affected

## e) Systemic risks

Pay attention to version issues when deploying on some l2 chains
https://docs.arbitrum.io/solidity-support

## f) Competition analysis

[Beacon](https://degenscore.com/)

## g) Security Approach of the Project

It must be ensured that the project party does not interfere with the operation of the project, so that it will be safer.
The control of this type of smart contract project is more centralized, because the deployment of the contract and the setting of key parameters are usually performed by the owner or administrator of the project. The project owner or administrator has the authority to upgrade, maintain, and modify the contract.

## h)Other Audit Reports

**Automated Findings:**
https://github.com/code-423n4/2023-10-brahma/blob/main/bot-report.md

**Other Audit Reports:**
https://github.com/Brahma-fi/brahma-security/tree/master/audits

## Time spent

37 hours


### Time spent:
37 hours