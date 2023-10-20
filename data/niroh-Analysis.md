# Overview

Brahma is a smart contract framework enabling digital asset managers to use crypto funds in various DeFi activities while maintaining granular access control and operation policies to achieve a balance between ease of operation and risk mitigation. This is enabled through the use of multi-sig gnosis safes with enhanced functionality that is integrated through gnosis safe modules and guards.

Brahma achieves its goal mainly through four design constructs:

1. A hierarchical structure comprising Console Accounts and Sub Accounts, where the owners of each Console Account maintain full control of its Sub Accounts.

2. A modular Policy framework through which users can define specific limitations on account activity per account, validated by an external trusted validator.

3. An Executor framework enabling fund managers to grant actors (Executors) limited execution permissions to run selected  DeFi activities, while imposing a strict policy on the actions these actors can perform.

4. Additional modules that simplify and automate common DeFi activities such as trading, staking, cost averaging etc. (out of scope for this audit).

Brahma's solution is unique in the landscape of DeFi fund management solutions in that asset managers maintain full custody of their funds throughout the usage of the system. This is achieved through the extensive use of Gnosis Safes as the underlying multisig wallet framework.

# Scope

**src/libraries/TypeHashHelper.sol** - Helper contract that creates an EIP712 compatible struct hash for a transaction struct (used to sign transaction data) and validation struct (used for trusted validator signature)

**src/libraries/SafeHelper.sol** - Helper contract for interaction with Gnosis Safe functions, such as packing multiple transactions for the Multisend interface.

**src/core/TransactionValidator.sol** - Called by Brahma Guards and ExecutionPlugin to perform validations before and after running transactions on a Brahma Gnosis Safe. 

**src/core/SafeModeratorOverridable.sol** - A Gnosis Safe Guard (Hook that performs custom validations) applied to Console Accounts that can be disabled or enabled by safe owners.

**src/core/SafeModerator.sol** - Similar to SafeModeratorOverridable but used with Sub Accounts and does not enable turning on or off (Sub Accounts must have the guard enabled at all times).

**src/core/SafeEnabler.sol** - A workaround contract to enable setting a module to a gnosis safe while it's being created (typically not possible as only the safe contract can set a module to itself). This is worked around by delegating a call to this contract which adds the module directly to the safe state using the same structure as the original function.

**src/core/Constants.sol** - Constants used throughout the system. Mainly hashes that are used for mapping the different core contracts on the Address Provider.

**src/core/ConsoleFallbackHandler.sol** - Used for Console safes that have a policy enabled - replaces the out-of-the-box Gnosis Safe compatibility handler, adding policy validation enforcement.

**src/core/AddressProvider.sol** - A global mechanism that controls the core contracts of the system. Maps core contracts to fixed hashes. Enable a single governance address to set or update system core contracts. Governance can be handed over to a different address by the current governance (pending acceptance by the new governance).

**src/core/PolicyValidator.sol** - Used by Brahma Guards to validate policy where one is applied. Checks that the policy signature structure is valid and signed by the authorized validator, that the correct policy hash was signed and that the signature hasn't expired.

**src/core/registries/PolicyRegistry.sol** - A registry mapping policy hashes to accounts. Verifies that only authorized addresses can set policies (either the SafeDeployer zeroing a policy, a Console Account setting its own policy or a Console Account setting a policy for a Sub Account).

**src/core/registries/ExecutorRegistry.sol** - A registry of the authorized Executors of each Sub Account. Enables Owner Console Accounts to add/remove Executors to a Sub Account.

**src/core/registries/WalletRegistry.sol** - Registery for wallets (Console Accounts) and their list of Sub Accounts. Anyone can register an address as a wallet as long as its not already registered. Only the SafeDeployer can register a Sub Account.

**src/core/AddressProviderService.sol** - An interface enabling read only access to the core contracts registered by the AddressProvider.

**src/core/SafeDeployer.sol** - Helper contract enabling deployment of Console Accounts and Sub Accounts while handling all configurations required by Brahma (i.e. Enabling Guards, enforcing policies if provided, enabling Console Accounts control of Sub Accounts through the use of Safe Modules etc.)

**src/core/ExecutorPlugin.sol** - A Gnosis Safe Module that can be applied to Sub Accounts to enable granting execution permissions to registered Executors of a Sub Account.

# Approach Taken in evaluating the codebase

In analyzing the codebase we took the following steps:

**1. Architecture Review**  
    Reading available documentation to understand the architecture and main functionality of Brahma Fi.

**2. Compiling code and running provided tests**

**3. Detailed Code Review**  
    A thorough ode review of the contracts in scope as well as relevant parts of third party contracts (mainly Gnosis Safe contracts).

**4. Security Analysis**  
    Defining the main attack surfaces of the codebase i.e. Safe funds exfiltration, unauthorized transaction executions etc.

**5. Additional Testing**  
    Adding an array of additional tests derived from the potential attack surfaces outlined in the security analysis conducted in the previous step.    


# Mechanism Review

Brahma's architecture includes several mechanisms working in unison:

## 1. Core Contract Addresses Registry
This mechanism enables a single governance address to set and update the core contracts used throughout Brahma framework. This registry is managed by the AddressProvider contract. Any system component that needs access to these contracts implements the AddressProviderService interface and is initialized with the (single) AddressProvider contract controling the system.

![Address Provider Diagram](https://raw.githubusercontent.com/nirohgo/images/d42c6c1985d00f6b8647038e3fe7c4b18ebb92c2/AddressRegistry.png "Address Provider Diagram")


## 2. End User Registries
These regitries keep track of Console and Sub Account wallets and their associated relations, security policies and permitted Executors. Three registries exist in the system:  
        **1. Wallet Registry**  
        Keeps track of Console (main) Accounts, Sub Accounts and their relations. Any address can register as a wallet (Console Account). Sub Accounts can be registered only through the SafeDeployer contract.  
        **2. Policy Registry**  
        Keeps track of Policy Hashed applied to accounts. Policies can be set by Console Accounts to themselves or to their Sub Accounts.  
        **3. Executor Registry**  
        For Sub Accounts that use the ExecutorPlugin, keeps track of the list of Executor addresses approved to run transactions on each Sub Account.

## 3. Access control
Access control forms the heart of the Brahma protocol. On Brahma there are four mechanisms controling access that may be combined in different ways:
1. The built in Gnosis Safe multisignatures/threshold system controling Safes.
2. A policy system through which custom transaction execution policies can be applied per account. Sub Accounts must have a policy assigned to them, while Console Account owners can choose weather or not to enforce a policy. Policies are validated by en external entity (Trusted Validator) whos address is set through the AddressProvider. transactions send to Accounts with a policy assigned to them must be signed by the trusted validator along with a hash of the specific policy applied. 
3. An Executor Plugin enabling the Console Account controling a Sub Account to set specific addresses as approved Executors. These addresses can then send transactions to be executed by the Sub Account. These transactions must be signed by both the Executor and the Trusted Validator to be executed.
 4. A Console Account can always run transactions on Sub Accounts it controls, bypassing any policies applied to them (but still requiring the Console Account multisig).

Brahma's ACL system is implemented with the help of two Gnosis Safe features: Guards (hooks that enable attaching custom validations to Safes) and Modules (addresses that when registered on a Safe as Modules are allowed to bypass the regular multisignature path and execute transactions on the Safe directly).

The following diagram ilustrates how access control works for Brahma Safes.
      
![Access Control Diagram](https://raw.githubusercontent.com/nirohgo/images/b877d08b4428209340dfdd5e57fdc3b091f30c42/ACL.jpg "Access Control Diagram")


## 4. Helper Contracts
A set of contracts implementing common functionalities to simplify working with the system. These include:
- SafeDeployer: Implements the deployment of Console Accounts and Sub Acccounts while taking care of all nesessary configurations and setups.

- SafeHelper: Helper functions for working with Safes (such as running a transaction on a safe, packing several transaction for Safe Multisend etc)
- TypeHashHelper: Helper library containing functions to build EIP712 struct and type hashes 
- ConsoleOpBuilder: creates bytecode for common operations on Console Accounts: Enabling/Disabling a policy on a Console, Enabling/Disabling the Executor plugin on a Sub Account 
![SafeDeployer Diagram](https://raw.githubusercontent.com/nirohgo/images/a5750ce837538372ee81d1b6b7398b2553e9bea7/HelperContracts.png "SafeDeployer Diagram")



# Systemic Risks

As with most blockchain frameworks, the use of Brahma for fund management entails several risks that are inherent to the system:  
### AddressProvider System
The reliance on the AddressProvider registry with governance ability to replace any core contract in the system poses several risks:  
1. **Security Risk** - One of Brahma's main selling points is the fact that users maintain full custody of their assets. This assurance however can be easily broken if the governance address (which according to the docs is currently controled by Barhma team only) decided to change some core contracts maliciously. For example, the ExecutorRegistry and the PolicyRegistry can be replaced with malicious versions that "register" a malicious address as an Executor and an all allowing policy on user Sub Accounts granting them full control on the Sub Account.
2. **Stability Risk** - The ability to replacing key contracts in the system without any restrictions on interface or structure poses the risk of code breaking due to interface or structural changes that are incompatible with other contracts using the changed contracts.
### External/Third Party protocol risk
As with any smart contract system, the reliance on third party code and packages might incure additional risks residing in these packages. With regards to Brahma there is a significant dependance on Gnosis Safe as well as OpenZeppelin and Solady.
### Offchain/External elements
Brahma's security approach places significant weight on its Policy system which relies on an external validator to sign policy validations thus enforcing user policies. Atention should be paid to serveral aspects of this mechanism with regards to security:  
**1. Transparency** - As of the time for writing we could not find any documentation of the policy enforcing mechanism, its code, available policy options or level of security. These are crucial to increase trust in the system.  
**2. Availability** - Since the trusted validator is a crucial part of the system (without which users will have limited access to their safes) the availability guarantees of this component need to be made clear.
 

# Centralization Risks

### Governance  
As mentioned above, the system and its security relies entirly on the governance address (currently fully controled by Barhma team). To increase trust and security it is advisable to expand the spread of control, possible by adding tiered governance where for critical actions an additional governance address is required, including trusted entities external to Brahma, or even a community DAO.

# Recommendations

Based on out review and the above mentioned risks we compiled the following list of recommendations that might address some of these risks:

### Using Zero Knowledge Proofs for policy validation
In the current system, trust must be put in an external Trusted Validator to validate Policies which are central to Brahma's security framework. While the validator includes a relevant Policy hash in its signature to specify which policy it validated, there is no guarantee that the signed policy was indeed enforced. If the trusted validator is compromized, anyone can provide validity signatures without enforcing the policy. In the future this trust assumption may be alliviated if a mechanism is integrated where policy validations are precompiled as ZKPs and are submitted together with a proof that the right policies have been enforced.


### Employing a more robust and detailed governance system
As per current documentation, Brahma's governance address is a multisig of 10 addresses from within the team. In the future it may be advisable to further decentralize governance by adding external signers (such as independant trusted industry figures) or even a DAO.

### Limiting Wallet Registration to Brahma Enabled Console Safes
Currently the system ebables any address to be registered as a wallet and deploy Sub Accounts. It is recommended to limit this option to Brahma console accounts only. This is more aligned with Brahma protocol semantics and will prevent the option that users will grant a single EUA address full control of a sub safe by mistake.

### Replacing the Current AddressProvider System With a More Standard Contract Updrage Mechanism
With the current system, core system contracts can be replaced at will without any backward compatibility checks, version controls, interface support or an option to set multiple new contracts as a batch.This might result in code breaks or unecpected system behaviour which in turn can result in security breaches or denial of service to safes. In the future it may be beneficial to change this system (for exmaple with a standard contractUpgradability framework) or enhance it with more capabilities preventing the above mentioned errors.


### Time spent:
15 hours