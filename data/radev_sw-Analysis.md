# Brahma Audit Contest Analysis Report | 13 Oct 2023 - 20 Oct 2023

---

| #   | Topic                                                 |
| --- | ----------------------------------------------------- |
| 1   | Questions & Notes during the Audit                    |
| 2   | Codebase Explanation, Examples Executions & Scenarios |
| 3   | Code Complexity                                       |
| 4   | Systemic Risks and Possible Attacks                   |

---

# 1. Questions & Notes during the Audit

## 1.1 What Brahma is?

- Brahma is a non-custodial protocol that optimizes DeFi execution and provides tools for comprehensive asset management.
- Brahma’s products focus on improving the UX of performing DeFi interactions from self custody and mitigating security risks involved in the process.
- With Console, Brahma is specifically building for smart contract wallet custody. Console extends open dApp connectivity and interactions within a single-point interface without having to toggle between multiple executions. With intuitive transaction batching, access control and a seamless execution flow, Console extends a secure layer for granular asset management within the ecosystem.

## 1.2 Summary and Analysis of the Documentation

The Brahma DeFi Protocol, specifically Brahma Console v2, is an orchestration layer designed to improve the DeFi (Decentralized Finance) experience for users of smart contract wallets.

1. **Enhancing DeFi Experience**: Brahma Console v2 aims to enhance the DeFi experience by providing an orchestration layer. This layer likely simplifies and streamlines DeFi interactions for users, making it easier to engage with various DeFi protocols and applications.

2. **User-Configurable Automation/Strategies**: Brahma Console v2 allows users to configure and automate their DeFi strategies. This means users can set up specific actions or transactions to occur automatically based on their preferences. This feature can help users save time and optimize their DeFi activities.

3. **Low-Cost**: The protocol is designed to be cost-effective, ensuring that users can benefit from its features without incurring high fees. This is important for accessibility and affordability.

4. **Custody of Funds**: One significant advantage of Brahma Console is that it provides automation without requiring users to give up custody of their funds. This is a key selling point, as many users are concerned about the security of their assets in DeFi applications.

5. **SafeSub Accounts**: Users have access to SafeSub Accounts, which reduce their risk when interacting with the protocol. These Sub Accounts isolate interactions, adding an extra layer of security to the user's assets.

   5.1. **Console Account**: This is a standard Gnosis Safe owned by multiple users. It serves as a central account within the Brahma DeFi Protocol. "Console Account" serves as a primary or central point of control and coordination within the Brahma DeFi Protocol.

   5.2. **SubAccount**: SubAccounts are Gnosis Safes operated by delegatee accounts known as Operators. These Safes are owned by the Console Account, with Console Account acting as a safe module and SafeModerator as a safeguard. This structure allows for controlled access to specific transactions enabled by the Console Account.

   5.3. **Operator**: Operators are accounts delegated ownership of SubAccounts with restricted rights enforced by SafeModerator. The ConsoleAccount can update their rights. This hierarchy adds a layer of control and security to the system.

   5.4. **Executor**: Executors are authorized accounts capable of executing module transactions on a SubAccount using the ExecutorPlugin. ExecutorPlugin needs to be enabled as a module on the SubAccount. This feature likely enables specific actions to be performed on behalf of the SubAccount with appropriate authorization.

In summary, Brahma Console v2 is an advanced DeFi orchestration layer that prioritizes user security, control, and cost-effectiveness. It enables automation and strategies while allowing users to maintain custody of their funds. The architecture involves Console Accounts, SubAccounts, Operators, and Executors, creating a structured and secure environment for DeFi interactions.

## 1.3 Questions During the Audit

- How are Operator defined? Is it offchain? Are Operator and Trusted Validator the same?

  - Operators are “owners of sub account”. Their functionality is validated using a guard/transaction validator/delegated owners.
  - Trusted Validator is a trusted address in the system that allows transaction by signing them and the signature gets verified in policy validator contract.

- Is ConsoleAccount just GnosisSafe wallet? GnosisSafeProxy to be exact, which delegates the calls into GnosisSafe?

  - Yes.

- `SafeEnabler` is just used to set a guard and fallback handler on `Console` by making a delegate call to it.

- Is there 1 SafeEnabler for each Console?

  - No. 1 SafeEnabler for all.

- Can you explain a bit more about what the policy commits are used for? Is it mainly just an offchain mechanism so you can determine if the transaction was validated and signed by one policy versus another?

  - You can think of Policies as 32 byte offchain proof, allowing certain onchain actions.
  - Say if you(console account) wanna allow one of your sub account to be able to make swap transaction to uniswap, you can register corresponding policy on policy registry. Whenever you make a tx the trusted validator would sign that policy with tx offchain and the signature gets verified in PolicyValidator.
  - Therefore a policy/policy commit can include multiple allowed actions.

- Can `ExecutorPlugin` be used on `Console` account or it's only for a `SubAccount`?
  - It is intended to be used only for subAcc, tho it can be used w any safe being a module.

## 1.4 Additional Notes During the Audit

- Console Account: (primary point of control within the Brahma, owned by multiple users)
- SubAccount: (sub accounts, delegated by accounts known as Operators, Console Account can have multiple SubAccounts, Console Account acting as a safe module and SafeModerator as a safeguard)
- Operator: (accounts delegated ownership of SubAccounts with restricted rights enforced by SafeModerator (so one SubAccount can have multiple Operators with delegated ownership), ConsoleAccount can update their rights)
- Executor: (they are authorized accounts capable of executing module transactions on a SubAccount using the ExecutorPlugin, ExecutorPlugin needs to be enabled as a module on the SubAccount)

- msg.sender (SubAccount) owns a Wallet (one Wallet has one owner (the owner is SubAccount)) (one msg.sender can has only one registered Wallet)
- one Wallet has many SubAccounts list
- one SubAccount has many Executors

- Main Console and Sub-Account: The bedrock of Console's architecture is the Main Console (or Main Safe), which functions as the central account owner. Under the Main Console, Sub Accounts are created and owned. This strategic separation facilitates efficient activity isolation and risk management. Main Console signers hold administrative roles, enabling them to create new Sub Accounts, assign external operators to Sub Accounts, and establish transaction policies with granular permissions for each Sub Accounts.

- SubAccounts are effortlessly created and deployed through the Factory Safe Deployer, and each can be assigned one or more operators. All operator transactions are routed through the Console hook, ensuring robust security measures.
  Creating and managing Console Sub Accounts is not the same as just creating multiple Safes, as vanilla Safe owners all have the same permissions and ability to change ownership and/modules configuration of a Safe if the threshold is met. This makes it infeasible in most scenarios to keep a low signature threshold to perform quick DeFi operations, like 2 out ot x owners, as 2 owners could remove other owners and effectively take control of the Safe.
  In Console, every Sub-Account has an on-chain relationship with the Main Console Safe as its owner through the Console Hook, which ensures that Sub-Account operators can only execute transactions following the security policies set in place by the Main Console admins.

- User Flow
  The Flowchart below gives an overview of Console’s main automation components and their functions.
  A user onboards to Console by connecting an EOA (externally owned account) through a wallet like Metamask, Rabby, etc, and creating (or importing) a new Safe owned by the connected wallet in the onboarding flow.
  Once the new Safe is created or imported, users can either directly perform operations from their Main Console such as send, swaps, or connect to any dApp with wallet connect, or can create multiple subaccounts “Sub Accounts” to segregate risk, and assign operators and security policies to each.
  Users can also select automations, for example, DCA, which will be customizable to their preferred frequency and coin, review the transaction, simulate it and execute it to grant Console the permission to automatically call the swap function from user Safe to Cowswap at each selected time interval.
  Every time a new automation subscription happens, Console creates (or reuses) an automation Sub-Account for the user. This is a Safe that is owned by the user master Safe. This is done to silo smart contracts and approval risks of each automation. The funds always remain in the user’s Safe, and only the swap call is automated.
  In the DCA example, both the token spend approval as well as the approval to interact with the CoWswap contract is signed and given only on the newly created Subaccount Safe and only the exact token amount required for the DCA automation is transferred to the new Subaccount Safe.  
  The Main Console is where the admins can monitor and manage all of the fund flows, permissions and settings. Only admins have permission to perform these actions, while Sub-Account operators can only access their assigned Sub-Account and per

---

---

# 2. Codebase Explanation, Examples Executions & Scenarios

## ConsoleAccount execTransaction:

1. SafeModeratorOverridable: checkTransaction()
2. TransactionValidator: validatePreTransactionOverridable()

- `_isConsoleBeingOverriden()`: Check if guard or fallback handler is being removed, if yes, skip policy validation
- `_validatePolicySignature`: Validate policy otherwise

3. PolicyValidator: isPolicySignatureValid()

- Get policy hash from registry and revert if it is `bytes(0)`
- Ensure transaction has not expired
- Revert if TrustedValidator is an EOA and no trustedValidator signature is provided

4. Execute transaction
5. SafeModeratorOverridable: checkAfterExecution()

- no checks

6. TransactionValidator: validatePostTransactionOverridable()

- no checks

Note: Step 1 and 3 are only executed if SafeModeratorOverridable is enabled as guard on ConsoleAccount, which may not be enabled in the case where ConsoleAccount has not setup policy & guard as they are optional.

---

## ConsoleAccount (guard removal transaction)

1. SafeModeratorOverridable: checkTransaction()
2. TransactionValidator: validatePreTransactionOverridable()

- `_isConsoleBeingOverriden()`: Check if guard or fallback handler is being removed, if yes, skip policy validation
- `_validatePolicySignature()`: Validate policy otherwise

3. Execute transaction
4. SafeModeratorOverridable: checkAfterExecution()

- no checks

5. TransactionValidator: validatePostTransactionOverridable()

- no checks

Note: Step 1 and 3 are only executed if SafeModeratorOverridable is enabled as guard on ConsoleAccount, which may not be enabled in the case where ConsoleAccount has not setup policy & guard as they are optional.

---

## SubAccount execTransaction by operator:

1. SafeModerator: checkTransaction()
2. TransactionValidator: validatePreTransaction()

- `_validatePolicySignature()`: Validate policy signature for a safe txn

3.  PolicyValidator: isPolicySignatureValid()

- Get policy hash from registry and revert if it is `bytes(0)`
- Ensure transaction has not expired
- Revert if TrustedValidator is an EOA and no trustedValidator signature is provided

4.  Execute the desired calldata
5.  SafeModerator: checkAfterExecution()
6.  TransactionValidator: validatePostTransaction()

- `_checkSubAccountSecurityConfig`: Validate the module, guard and fallback handler for a subaccount

  - Ensure guard has not been disabled
  - Ensure fallback handler has not been altered
  - Ensure owner console as a module has not been disabled

---

## SubAccount execution via ConsoleAccount:

## SubAccount execution via executor plugin:

1. ExecutorPlugin: executeTransaction()
2. ExecutorPlugin: `_validateExecutionRequest()`

- Check if executor is valid for given account
- Empty Signature check for EOA executor
  - Executor is an EOA and no executor signature is provided
- `TransactionValidator#validatePreExecutorTransaction()`: Validate executor signature

3. TransactionValidator: validatePreExecutorTransaction()

- `_validatePolicySignature()`

4. PolicyValidator: isPolicySignatureValid()

- Get policy hash from registry and revert if it is `bytes(0)`
- Ensure transaction has not expired
- Revert if TrustedValidator is an EOA and no trustedValidator signature is provided

5. ExecutorPlugin: execTransactionFromModuleReturnData()
6. TransactionValidator: validatePostExecutorTransaction()

- `_checkSubAccountSecurityConfig()`: Validate the module, guard and fallback handler for a subaccount
  - Ensure guard has not been disabled
  - Ensure fallback handler has not been altered
  - Ensure owner console as a module has not been disabled

---

---

# 3. Code Complexity

1. **Building Blocks**: The code is like building with LEGO blocks. Different parts of the code do different jobs, like creating accounts, managing rules, and more. This makes it organized and easier to work with.

2. **Sharing Ideas**: Imagine sharing ideas within a family. In the code, ideas are shared between different parts, making things efficient. But it's like telling stories from one generation to another – you need to be careful that nothing gets lost or mixed up.

3. **Reusing Good Stuff**: Think of using your favorite recipe over and over. The code reuses good recipes (called libraries) to avoid repeating the same steps. This makes the code shorter and easier to understand.

4. **Making Choices**: Just like choosing what to wear depending on the weather, the code makes decisions. These decisions can be complex, so they need to be clear and tested well.

5. **Keeping Secrets**: The code keeps track of secrets, like hidden treasure maps. This needs to be done carefully to keep everything safe and secure.

6. **Talking to Friends**: Sometimes, the code talks to other people or computers. This is like making phone calls, and it has to be done in a way that's safe and reliable.

7. **Number Games**: The code plays games with numbers. It's like solving puzzles, but if not done right, it can lead to problems.

8. **Sharing News**: Imagine sending out announcements about an event. The code uses "events" to announce what it's doing. This helps keep track of everything.

9. **Dealing with Mistakes**: Just like people make mistakes, the code can too. Handling mistakes is crucial, so the code doesn't get confused.

10. **Saving Energy**: The code tries not to waste energy (in the form of gas fees). It's like driving a car efficiently to save fuel.

11. **Testing and Checking**: Like double-checking your homework, the code needs to be tested and reviewed by experts to make sure it's safe and reliable.

In simple terms, the Brahma Protocol's code is like building a complex structure with different parts, rules, and interactions. It's important to make sure everything works smoothly, just like in a well-organized family or a well-prepared recipe. Auditing and testing are like getting a second opinion to ensure everything is in order.

# 4. Systemic Risks and Possible Attacks

### Possible Attacks

Attack ideas (Where to look for bugs, Main focus of Audit)

- Access control, including `potential delegatecall and proxy issues`.
- Users can choose to import their own safe wallet in console which can be malicious. Exploring potential attack vectors there.
- DoS causing of validations during functions execution
- Are there any functions that should be called during functions flow execution
- Think for additional validations/checks during functions flow execution (example: changing of some state variables like wallet/ConsoleAccount owners and SubAccounts)
- address(0) situations
- When `updatePolicy()` and `currentCommit = bytes(0)` check if the function actually can be called from all supposed ways
- Wrong check in `registerWallet()`, `if (subAccountToWallet[msg.sender] != address(0)) revert IsSubAccount();`. When the SubAccount is registered assigning value to subAccountToWallet is not the `msg.sender`, but the account.
- Is the Guard enable when the Console policy get non-zero value???

Main invariants

- Main Console Account should always stay as a module enabled on any subaccount it owns (unless manually changed by Main Console)
- Subaccount should always have SafeModerator enabled as guard on it (unless manually changed by Main Console)
- Subaccount should always have ConsoleFallbackHandler enabled as the fallback handler on it (unless manually changed by Main Console)
- Main Console Account should always be able to remove SafeModeratorOverridable without validation from PolicyValidator
- Main Console Account should always be able to remove ConsoleFallbackHandler without validation from `PolicyValidator

---

---


### Time spent:
10 hours