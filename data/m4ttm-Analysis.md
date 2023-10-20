## 1. Audit Approach

|     |     |     |
| --- | --- | --- |
| Step | Task | Details |
| 1   | Run Tests | Tests run successfully |
| 2   | Coverage | 100% test coverage for contracts in audit scope |
| 3   | Slither | Reviewed Slither results, no vulnerabilities discovered |
| 4   | Surya | Generate graphs to understand the overall project structure. Provided an initial insight to the contract inheritance and function call flow |
| 5   | Solidity Metrics | Generate metrics reports to obtain initial insight on the codebase, noting areas of potential concern |
| 6   | Code Review | Line by line code review |
| 7   | Test Review | Review of each test and it's purpose |

## 2. Mechanism Summary

Brahma aims to allow users to automate DeFi actions without giving up custody of their funds by using preconfigured Gnosis Safe accounts. Through the use of sub accounts, users are able to isolate their interactions and reduce their overall exposure to risk. `ExecutorPlugin` is a contract which is authorised to make transactions on sub accounts via the use of modules. Transactions are validated through multiple stages, initially via `SafeModerator` which uses `TransactionValidator` to verify transactions before and after exection, and `PolicyValidator` to ensure policy signature validity. Three registry contracts are used, `ExecutorRegistry`, `PolicyRegistry` and `WalletRegistry` and a central source of truth for addresses is given by `AddressProvider`. This is accessed via an abstract contract called `AddressProviderService`.

## 3. Centralisation Risks

### 3.1 Governance Can Change Authorized Addresses in AddressProvider

The Governance has the ability to change the addresses authorised to use each policy. In the event of a Governance attack this could result in the addresses being set to malicious contracts, resulting in the loss of user funds.

### 3.2 Guardian Is Supposed To Handle Emergency Actions

According to the [documentation](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/docs/RegistriesRoles.md), the guardian is supposed to handle emergency actions such as pausing and conducting emergency withdrawals, however it is unclear where this functionality is implemented or who controls this address. This could potentially place trust in a single centralised entity, allowing them to freeze the protocol at will or steal users funds.

## 4. Quality Analysis

### 4.1 Codebase

The code is well structured and organised into separate contracts, each with a clear purpose. In depth functionality is logically separated into libraries. NatSpec comments are well used throughout the codebase. The [automated findings](https://github.com/code-423n4/2023-09-delegate/blob/main/bot-report.md) show that some styling conventions such as line lengths, custom error descriptions and exposing interfaces are not adhered to

### 4.2 Documentation

NatSpec comments are present throughout the code, although are incomplete in places. In depth details of how the contracts work can be found in the [documentation](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/docs). There are diagrams showing the main flows of execution but function names are incorrect in places.

### 4.3 Tests

Full test coverage is achieved for the contracts in scope, making use of both Hardhat and Foundry. Tests are well written and clearly structured. Testing could be further improved with the use of fuzz testing, or formal verification to give users the extra assurance that their funds are safe.

## 5. Architecture Improvements

### 5.1 Combine Registries and AddressProvider into a single contract

Multiple contracts are deployed for each registry and another for the address provider. `ExecutorRegistry`, `PolicyRegistry`, `WalletRegistry` and `AddressProvider` can be combined. Using a single deployed contract simplifies the system and makes this easier to read when using block explorers and tools using live data.

### 5.2 Use an Interface for AddressProvider instead of inheriting from AddressProviderService

Contracts inherit from AddressProviderService to obtain the addresses from AddressProvider. Using an interface to expose its methods directly and setting the address for this in the constructor would be the cleaner and more conventional approach.

### 5.3 Use Modifiers Instead of Internal Functions to Add Functionality to Multiple Functions

Internal functions such as `_onlyGov` and `_ensureAddressProvider` are implemented for access control where modifiers are the usual convention. The use of modifiers cleans the actual function body and makes it easier to read the code, obtaining key details before you dive into the function body.

### Time spent:
20 hours