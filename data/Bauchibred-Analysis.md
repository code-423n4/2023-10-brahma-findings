# Analysis

## Table of Contents

- [Approach](#approach)
- [Contracts Architecture Overview](#contracts-architecture-overview)
- [Centralization Risks](#centralization-risks)
- [Testability](#testability)
- [Other Recommendations/Feedback](#other-recommendationsfeedback)
- [Security Researcher Logistics](#security-researcher-logistics)
- [Conclusion](#conclusion)

## Approach

The audit initiated with a high-level overview of the Brahma ecosystem. Steps included referencing the [previous audit by Ackee](# Analysis

## Table of Contents

- [Approach](#approach)
- [Contracts Architecture Overview](#contracts-architecture-overview)
- [Centralization Risks](#centralization-risks)
- [Testability](#testability)
- [Other Recommendations/Feedback](#other-recommendationsfeedback)
- [Security Researcher Logistics](#security-researcher-logistics)
- [Conclusion](#conclusion)

## Approach

The audit initiated with a high-level overview of the Brahma ecosystem. Steps included referencing the [previous audit by Ackee](), leveraging static analysis tools, and manually reviewing the 800+ sLOC. The final phase consisted of interactive developer sessions on Discord, deepening comprehension of unique implementations and promoting discussions around potential vulnerabilities and significant observations.

## Contracts Architecture Overview

- **AddressProvider**:
  Manages and updates addresses used by other system contracts. Specifically, it handles registries and authorized contracts. Managed by an address representing governance. Interaction with this contract is facilitated through the AddressProviderService contract.

- **Registries**:
  The protocol employs three registries:

  - **ExecutorRegistry**: Manages authorized executors for transaction execution on subaccounts.
  - **PolicyRegistry**: Handles policy commits per account. The policy can be updated under specific conditions.
  - **WalletRegistry**: Manages the registration of wallets and subaccounts.

- **SafeDeployer**:
  Oversees Safe account deployment and configuration.

- **SafeModerator & SafeModeratorOverridable**:
  Validates transactions intended for subaccounts. The latter is designed for console accounts.

- **TransactionValidator & PolicyValidator**:
  Offer validation hooks, examining policies, and signatures.

- **ExecutorPlugin**:
  Represents a Safe module and additional execution capabilities.

- **Actors**:
  Outlines the system's roles and permissions.

- **Safe, Console & Subaccount**:
  Detail the architecture and functionalities of each.

- **Executor**:
  Defines accounts permitted to use the ExecutorPlugin.

- **Brahma Governance**:
  Describes the account that can add, but not update, authorized addresses or registries.

## Centralization Risks

- Upholding trust assumptions is pivotal for the project's longevity just as any other web3 protocol, NB: the protocol is well-designed and leverages best practices of trustless protocols. However, the governance is in charge of the authorized addresses registry which can affect users positively & also negatively.

- Another tricky one is the fact that a trust notion is placed on validators and assumption is made that they  always will validate the transactions correctly with the policy, and the policy will always prevent the sub account from doing unintended transactions, seems like a far stretch but it still entails a bit of centralization risk in it's ranks.

## Testability

The level of testability exhibited by the system is both extensive and praiseworthy. Achieving 100% test coverage is an admirable accomplishment. However, it's recommended to diversify test scenarios, particularly from the user's perspective. For instance, the team should explore potential outcomes when a user attempts to integrate with the system using a counterfactual account. It would be beneficial to understand how the current `isValidSignature` implementation interacts in such contexts among other user-centric scenarios.

## Other Recommendations/Feedback

### **Expand Auditing Tools Integration**

Many security experts prefer using Visual Studio Code augmented with specific plugins. While the popular [Solidity Visual Developer](https://marketplace.visualstudio.com/items?itemName=tintinweb.solidity-visual-auditor) has already been integrated with the protocol, there's room for incorporating other beneficial tools.

### **Deepen Test Coverage**

Although the report commends the 100% test coverage, it's imperative to understand that it doesn't guarantee a bug-free system. The team should consider adding more tests, especially those reflecting real-world user interactions with the protocol.

### **Refine Event Monitoring**

The current approach to event tracking could be optimized. Notable gaps include missing events or those that appear inconsistently placed. It's advised to employ a more deliberate and structured approach to event utilization.

### **Onboard More Developers**

Having multiple eyes scrutinizing a protocol can be invaluable. More contributors can significantly reduce potential risks and oversights.

### **Elevate Documentation Quality**

Though the current code documentation meets an acceptable standard, there's room for enhancement. Comprehensive documentation can aid in clarity and ease of onboarding for new developers.

### **Reconsider Library Usage**

While the extensive use of factories and libraries introduces additional complexity, it's recognized that such intricacies might be unavoidable, especially given the integration with platforms like Gnosis's Safe wallets.

### **Refine Naming Conventions**

There's a need to improve the naming conventions for contracts, functions, and variables. In several instances, the names don't resonate with their respective functionalities, leading to potential confusion.

## **Security Researcher Logistics**

My attempt on reviewing the Brahma spanned 15 hours distributed over 3 days:

- 1 hour dedicated to writing this analysis.
- 1.5 hours exploring the previous audit report, viewable [here](https://github.com/Brahma-fi/brahma-security/blob/master/audits/brahma-fi-consolev2-audit-10-23-ackee.pdf).
- 0.5 hours glancing through the bot race report, viewable [here](https://github.com/code-423n4/2023-10-brahma/blob/main/bot-report.md).
- 1.5 hours was allocated for discussions with sponsors on the private discord group regarding potential vulnerabilities.
- The remaining time on finding issues and writing the report for each of them on the spot, later on _editing a few with more knowledge gained on the protocol as a whole, or downgrading them to QA reports_

## Conclusion

The codebase was a very great learning experience, though it was a pretty hard nut to crack, being that it's like an update contest since bugs have already been spotted and mitigated from the previous contest.

During the course of my security review, I encountered a few interesting reports regarding how things would go wrong for users who sign a transaction from their accounts while these accounts exist counterfactually and how this wouldn't work. Additionally another interesting one is centred around how the magic value being used for the EIP 127! return value is different from what's been specified in the EIP and would lead to compatibility issues, since even correct signatures based on the EIP would be deemed invalid.


), leveraging static analysis tools, and manually reviewing the 800+ sLOC. The final phase consisted of interactive developer sessions on Discord, deepening comprehension of unique implementations and promoting discussions around potential vulnerabilities and significant observations.

## Contracts Architecture Overview

- **AddressProvider**:
  Manages and updates addresses used by other system contracts. Specifically, it handles registries and authorized contracts. Managed by an address representing governance. Interaction with this contract is facilitated through the AddressProviderService contract.

- **Registries**:
  The protocol employs three registries:

  - **ExecutorRegistry**: Manages authorized executors for transaction execution on subaccounts.
  - **PolicyRegistry**: Handles policy commits per account. The policy can be updated under specific conditions.
  - **WalletRegistry**: Manages the registration of wallets and subaccounts.

- **SafeDeployer**:
  Oversees Safe account deployment and configuration.

- **SafeModerator & SafeModeratorOverridable**:
  Validates transactions intended for subaccounts. The latter is designed for console accounts.

- **TransactionValidator & PolicyValidator**:
  Offer validation hooks, examining policies, and signatures.

- **ExecutorPlugin**:
  Represents a Safe module and additional execution capabilities.

- **Actors**:
  Outlines the system's roles and permissions.

- **Safe, Console & Subaccount**:
  Detail the architecture and functionalities of each.

- **Executor**:
  Defines accounts permitted to use the ExecutorPlugin.

- **Brahma Governance**:
  Describes the account that can add, but not update, authorized addresses or registries.

## Centralization Risks

- Upholding trust assumptions is pivotal for the project's longevity just as any other web3 protocol, NB: the protocol is well-designed and leverages best practices of trustless protocols. However, the governance is in charge of the authorized addresses registry which can affect users positively & also negatively.

- Another tricky one is the fact that a trust notion is placed on validators and assumption is made that they  always will validate the transactions correctly with the policy, and the policy will always prevent the sub account from doing unintended transactions, seems like a far stretch but it still entails a bit of centralization risk in it's ranks.

## Testability

The level of testability exhibited by the system is both extensive and praiseworthy. Achieving 100% test coverage is an admirable accomplishment. However, it's recommended to diversify test scenarios, particularly from the user's perspective. For instance, the team should explore potential outcomes when a user attempts to integrate with the system using a counterfactual account. It would be beneficial to understand how the current `isValidSignature` implementation interacts in such contexts among other user-centric scenarios.

## Other Recommendations/Feedback

### **Expand Auditing Tools Integration**

Many security experts prefer using Visual Studio Code augmented with specific plugins. While the popular [Solidity Visual Developer](https://marketplace.visualstudio.com/items?itemName=tintinweb.solidity-visual-auditor) has already been integrated with the protocol, there's room for incorporating other beneficial tools.

### **Deepen Test Coverage**

Although the report commends the 100% test coverage, it's imperative to understand that it doesn't guarantee a bug-free system. The team should consider adding more tests, especially those reflecting real-world user interactions with the protocol.

### **Refine Event Monitoring**

The current approach to event tracking could be optimized. Notable gaps include missing events or those that appear inconsistently placed. It's advised to employ a more deliberate and structured approach to event utilization.

### **Onboard More Developers**

Having multiple eyes scrutinizing a protocol can be invaluable. More contributors can significantly reduce potential risks and oversights.

### **Elevate Documentation Quality**

Though the current code documentation meets an acceptable standard, there's room for enhancement. Comprehensive documentation can aid in clarity and ease of onboarding for new developers.

### **Reconsider Library Usage**

While the extensive use of factories and libraries introduces additional complexity, it's recognized that such intricacies might be unavoidable, especially given the integration with platforms like Gnosis's Safe wallets.

### **Refine Naming Conventions**

There's a need to improve the naming conventions for contracts, functions, and variables. In several instances, the names don't resonate with their respective functionalities, leading to potential confusion.

## **Security Researcher Logistics**

My attempt on reviewing the Brahma spanned 15 hours distributed over 3 days:

- 1 hour dedicated to writing this analysis.
- 1.5 hours exploring the previous audit report, viewable [here](https://github.com/Brahma-fi/brahma-security/blob/master/audits/brahma-fi-consolev2-audit-10-23-ackee.pdf).
- 0.5 hours glancing through the bot race report, viewable [here](https://github.com/code-423n4/2023-10-brahma/blob/main/bot-report.md).
- 1.5 hours was allocated for discussions with sponsors on the private discord group regarding potential vulnerabilities.
- The remaining time on finding issues and writing the report for each of them on the spot, later on _editing a few with more knowledge gained on the protocol as a whole, or downgrading them to QA reports_

## Conclusion

The codebase was a very great learning experience, though it was a pretty hard nut to crack, being that it's like an update contest since bugs have already been spotted and mitigated from the previous contest.

During the course of my security review, I encountered a few interesting reports regarding how things would go wrong for users who sign a transaction from their accounts while these accounts exist counterfactually and how this wouldn't work. Additionally another interesting one is centred around how the magic value being used for the EIP 127! return value is different from what's been specified in the EIP and would lead to compatibility issues, since even correct signatures based on the EIP would be deemed invalid.




### Time spent:
15 hours