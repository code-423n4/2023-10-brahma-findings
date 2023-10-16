# Brahma.fi
Audit contest: `2023-10-brahma`

## Table of Contents

- 1. Executive Summary
- 2. Code Audit Approach
  - 2.1 Audit Documentation and Scope
  - 2.2 Code review
  - 2.3 Threat Modelling
  - 2.4 Exploitation and Proofs of Concept
  - 2.5 Report Issues
- 3. Architecture overview
  - 3.1 Overview of AddressProvider in Brahma.fi
  - 3.2  The protocol's essential registries
  - 3.3 Configuration of Safe accounts
  - 3.4 Validation of transactions
  - 3.5 Basic summary of the system
- 4. Implementation Notes
  - 4.1 General Impressions
  - 4.2 Composition over Inheritance
  - 4.3 Comments
  - 4.4 Solidity Versions
- 5. Conclusion

# 1. Executive Summary

In focusing on the ongoing audit contest 2023-10-brahma, my analysis starts by delineating the code audit methodology applied to the contracts within the defined scope. Subsequently, I provide insights into the architectural aspects, offering my perspective. Finally, I offer observations pertaining to the code implementation.


I want to emphasize that unless expressly specified, any potential architectural risks or implementation concerns discussed in this document should not be construed as vulnerabilities or suggestions to modify the architecture or code based solely on this analysis. As an auditor, I recognize the necessity of a comprehensive evaluation of design choices in intricate projects, considering risks as only one component of a larger evaluative process. It's essential to acknowledge that the project team may have already evaluated these risks and established the most appropriate approach to mitigate or coexist with them.

# 2. Code Audit Approach

Time spent: 18 hours

## 2.1 Audit Documentation and Scope
Commencing the analysis, the first phase entailed a thorough review of the https://github.com/code-423n4/2023-10-brahma to fully grasp the fundamental concepts and limitations of the audit. This initial step was crucial in guiding the prioritization of my audit efforts. Notably, the README associated with this audit contest stands out for its excellent quality, offering valuable insights and actionable guidance that significantly streamline the onboarding process for auditors.


## 2.2 Code review


Initiating the code review, the starting point involved gaining a comprehensive understanding of "AddressProvider.sol," The AddressProvider contract plays a pivotal role in overseeing and modifying addresses essential for various contracts within the system, including registries and authorized contracts. Governance is responsible for managing the AddressProvider contract, while other contracts interact with it through inheritance from the AddressProviderService contract.

## 2.3 Threat Modelling

The initial step involved crafting precise assumptions that, if breached, could present notable security risks to the system. This approach serves to guide the identification of optimal exploitation strategies. Although not an exhaustive threat modeling exercise, it closely aligns with the essence of such an analysis.

## 2.4 Exploitation and Proofs of Concept

Progressing from this juncture, the primary methodology took the form of a cyclic process, conditionally encompassing steps 2.2, 2.3, and 2.4. This involved iterative attempts at exploitation and the subsequent creation of proofs of concept, occasionally aided by available documentation or the helpful community on Discord. The key focus during this phase was to challenge fundamental assumptions, generate novel ones in the process, and refine the approach by utilizing coded proofs of concept to hasten the development of successful exploits.

## 2.5 Report Issues

While this particular stage might initially appear straightforward, it harbors subtleties worth considering. Hastily reporting vulnerabilities and subsequently overlooking them is not a prudent course of action. The optimal approach to augment the value delivered to sponsors (and ideally, to auditors as well) entails thoroughly documenting the potential gains from exploiting each vulnerability. This comprehensive assessment aids in the determination of whether these exploits could be strategically amalgamated to generate a more substantial impact on the system's security. It's important to recognize that seemingly minor and moderate issues, when skillfully leveraged, can compound into a critical vulnerability. This assessment must be weighed against the risks that users might encounter. Within the realm of Code4rena audit contests, a heightened level of caution and an expedited reporting channel are accorded to zero-day vulnerabilities or highly sensitive bugs impacting deployed contracts.

# 3. Architecture overview


## 3.1 Overview of AddressProvider in Brahma.fi
The `AddressProvider` contract is vital for managing and updating addresses used by other contracts in the system, particularly for registries and authorized contracts. Governance oversees this contract, and other contracts interact with it through inheritance from the `AddressProviderService` contract.



## 3.2 The protocol's essential registries:

### `ExecutorRegistry` manages allowed executors for transaction execution on subaccounts.
### `PolicyRegistry` handles policy commits per account, allowing updates under specific conditions.
### `WalletRegistry` facilitates registration of wallets and subaccounts.

## 3.3 Configuration of Safe accounts:

The `SafeDeployer` contract governs the deployment and configuration of Safe accounts, adapting them for console or subaccounts as needed.

## 3.4 Validation of transactions:

`SafeModerator` acts as a safeguard validating transactions that must be executed on subaccounts, as defined by the `TransactionValidator.`

`SafeModeratorOverridable` is a variant of `SafeModerator` tailored for console accounts within the system.

The `TransactionValidator` contract offers validation hooks, examining policy commits and signatures among other validation subjects.

`PolicyValidator` contract validates validator signatures against policy commits for respective accounts, ensuring non-expiration.

`ExecutorPlugin` contract functions as a Safe module, validating executor signatures and their validity for associated accounts.

## 3.5 Basic summary of the system:

The system involves various actors with distinct roles:

`Safe` architecture serves as the business logic for the protocol.

`Console` represents a Safe account managing subaccounts, capable of functioning independently or with additional logic.

`Subaccount` is a Safe account with an enabled guard, allowing for fine-grained execution control through the `ExecutorPlugin.`

`Executor` refers to an account with permissions to utilize the `ExecutorPlugin.`

`Brahma governance` is an account with the authority to add authorized addresses or registries, although it cannot update existing ones.


# 4. Implementation Notes

During the course of the audit, several noteworthy implementation details were identified, and among these, a significant subset holds potential value for the ongoing analysis.

## 4.1 General Impressions

Brahma Console v2 stands as a sophisticated orchestration layer meticulously designed to elevate the DeFi user experience within smart contract wallets. This innovative layer, constructed atop the robust Safe framework, empowers users with customizable automation and tailored strategies for seamless and frequent engagements within the DeFi landscape. Leveraging the force of Brahma, this solution is cost-effective and amplifies DeFi accessibility.

At its core, Brahma Console extends automation capabilities to users, presenting an avenue that does not necessitate relinquishing control of their funds. It seamlessly integrates into their wallet environment, providing a harmonious and secure platform for diverse interactions in the DeFi realm.

Within this ecosystem, users gain the valuable advantage of SafeSub-accounts, a feature meticulously crafted to mitigate risks associated with the protocol. These sub-accounts operate in isolation, effectively compartmentalizing and minimizing potential exposure.

A Console Account within this framework signifies a standard, readily available Gnosis Safe, collectively owned by multiple users. In contrast, a SubAccount embodies a Gnosis Safe intricately operated by designated delegatee accounts known as Operators. The ownership of this subAccount resides with the Console Account, which, as the supreme authority, configures it as a safe module and engages SafeModerator as a safeguard. Operators possess specific transaction execution rights granted by the Console Account (Owner), albeit moderated and restricted by SafeModerator. Importantly, the Console Account retains the ultimate authority over the subAccount, ensuring comprehensive control and oversight.

An Operator, on the other hand, represents an individual account entrusted with delegated ownership of the subAccount. The extent of their rights is carefully governed by the constraints imposed by SafeModerator. Notably, these rights can be dynamically updated and adjusted by the Console Account.

Further enriching the ecosystem, Executors are designated accounts endowed with the authorization to execute module transactions on a subAccount, facilitated through the ExecutorPlugin. To avail this capability, the ExecutorPlugin must be effectively enabled as a module within the subAccount, encapsulating an additional layer of security and transactional control.



## 4.2 Composition over Inheritance

Brahma is using composition over inheritance which is helpful in software design and  provides flexibility and adaptability by enabling the dynamic arrangement of components, allowing for independent modification and substitution. It promotes code reusability through versatile component combinations and reduces tight coupling, facilitating easier maintenance and testing. Additionally, composition encourages scalability and interface-based programming, addressing potential issues like the diamond inheritance problem and promoting a clear understanding of the codebase. Overall, composition empowers developers to create more modular, flexible, and maintainable code, making it a valuable paradigm for building robust and evolving software systems.


## 4.3 Comments

### Importance of Comments for Clarity:

Comments serve a pivotal role in enhancing the understandability of the codebase. While the code is generally clean and logically structured, judiciously placed comments can provide valuable insights into the functionalities and intentions behind the code. They contribute to a better comprehension of the code's purpose, especially for auditors and developers involved in the analysis.

### Strategic Comment Placement:

The codebase would greatly benefit from an increased presence of comments, particularly within the AddressProvider.sol contract. Strategic placement of comments within this essential contract can significantly aid auditors in comprehending the implementation details. These comments should elucidate the logic, processes, and methodologies employed, promoting a seamless audit experience.

### Facilitating Audits and Code Readability:

Comprehensive comments in the AddressProvider.sol and other contracts can expedite the auditing process by allowing auditors to swiftly grasp the intended functionality of the code. This, in turn, enhances the readability of the functions and methods, making it easier for auditors to identify any inconsistencies or deviations between the documented intentions and the actual code implementation.

### Detecting Discrepancies in Code Intentions:

An important aspect of code review is discerning any mismatches between the documented intentions in the comments and the actual code implementation. Comments that accurately reflect the code's purpose are crucial for auditors, as discrepancies between the two can be indicators of potential vulnerabilities or errors. The act of aligning comments with the true code behavior is fundamental to ensuring the reliability and security of the smart contract.


## 4.4 Solidity Versions

Though there are valid arguments both in support of and against adopting the latest Solidity version, I find that this discussion bears little significance for the current state of the project. Without a doubt, choosing the most up-to-date version is a far superior decision when compared to the potential risks associated with outdated versions.


# 5. Conclusion

### Positive Audit Experience:

The process of auditing this codebase and evaluating its architectural choices has been thoroughly enjoyable and enriching. Navigating through the intricacies and nuances of the project has been enlightening, presenting an opportunity to delve into the complexities of the system.

### Strategic Simplifications for Complexity Management:

Inherent complexity is a characteristic of many systems, especially those in the realm of blockchain and smart contracts. The strategic introduction of simplifications within this project has proven to be a valuable approach. These simplifications are well-thought-out and strategically implemented, demonstrating an understanding of how to manage complexity effectively.

### Achieving a Harmonious Balance:

A notable achievement of this project is striking a harmonious balance between the imperative for simplicity and the challenge of managing inherent complexity. This equilibrium is crucial in ensuring that the codebase remains comprehensible, maintainable, and scalable, even as the system becomes more intricate.

### Importance of Methodology Overview:

The overview provided regarding the methodology employed during the audit of the contracts within the defined scope is invaluable. It offers a clear and structured insight into the analytical approach undertaken, shedding light on the depth and rigor of the evaluation process.

### Relevance for Project Team and Stakeholders:

The insights presented are not only beneficial for the project team but also extend to any party with an interest in analyzing this codebase. The detailed observations, considerations, and recommendations have the potential to guide and inform decision-making, contributing to the project's overall improvement and security.


### Time spent:
18 hours


### Time spent:
18 hours