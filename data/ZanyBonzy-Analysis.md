# **Brahma Console**
Brahma Console v2 is a platform that helps people use decentralized finance (DeFi) more easily and safely. It does this by providing a layer of automation and risk management on top of smart contract wallets.  It provides Sub-accounts, which isolate user interactions to reduce risk. The Console is built on the principle of non-custodiality which means that users should't have to give up control of their funds to execute transactions.

It consists of four major account types:
1. Console account - a gnosis safe that is owned by a number of users, and can be safeguarded by a `safeModeratorOverridable` if the owners want to;
2. Sub-account - also a gnosis safe owned by console acct, controlled by operators, has the console account as a safe module and a `SafeModerator` as safeguard;
3. Operator account - account acts as a delegate of a sub-account. Its rights are restricted by `safeModerator` and controlled by the console account;
4. Executor account - an external account that makes module transactions on the sub-account via an executorplugin which needs to be enabled before use.

***

## **System Overview**
### Scope
#### Core Contracts

1. [AddressProvider.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol) - is the gatekeeper of the system's authorized contracts and registries. It ensures that only the right addresses are allowed to interact with the system, and that all changes to the addresses are properly authorized and implemented;
2. [AddressProvideService.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol) - is an abstract contract that provides a common interface for all core contracts to interact with the `AddressProvider` contract. It does this by providing a host of helper functions to help in these interactions;
3. [SafeDeployer.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol) - does the actual work of deploying and registering console accounts(with or without policies) and sub-accounts with policies. It sets up the required states - safeguards, fallback handler and module for these accounts;
4. [SafeModerator.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModerator.sol) - is the safeguard for a sub-account. It validates transaction before and after execution by making sure that the sub-account's state remain as is and is policy compliant. It does this through the `TransactionValidator` contract; 
5. [SafeModeratorOverridable.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol) - is the safeguard for a console account that has safeguard enabled (Its not compulsory). It has the same functions as the `safeModerator` contract. On top of this, it also does a check to confirm if an owner decides override the it as a safeguard and/or the `ConsoleFallbackHandler` as the fallback handler. 
6. [TransactionValidator.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol) - is the contract called by the safeguards to validate state/policies before and after executions. It is also called by an executor through the `executorPlugin` for sub acoounts module executions.
7. [PolicyValidator.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol) - validates the Trusted Validator's signature against the policies. It ensures policy compliance based on EIP712 sugnatures before executing transactions.
8. [ExecutorPlugin](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol) - acts as a gatekeeper that ensures that only authorized executors can submit valid execution requests on console accounts. It also verifies that the requests are valid and that the executor has the necessary permissions to execute them.
9. [SafeEnabler.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol) - makes delegatecalls to the `ModuleManager` and `GuardManager` to set up modules and safeguard states. This helps the accounts bypass the selfAuthorized checks that exist on the actual Safe.
10. [Consolefallbackenabler.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol) - is a fallback handler for Safe accounts. It performs the same functions as the Safe's `CompatibilityFallbackHandler`, but it also ensures that policy validation is performed for ConsoleAccounts and sub-accounts that have policy validation enabled;
    
#### Registry contracts
11. [WalletRegistry.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol) - manages registrations and ownership relationships of/between wallets and their sub-accounts
12. [PolicyRegistry.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol) - allows authorized entities to set and update policy commitments for specific accounts.
13. [ExecutorRegistry.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol) - manages the registration and removal of executor addresses by owners for their desired sub-accounts.

#### Library contracts
14. [SafeHelper.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol) - is a helper library that provides essential functions for interacting with Safe. This includes executing transactions, generating calldata, obtaining necessary storage slots, and parsing data.
15. [TypeHasHelper.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/TypeHashHelper.sol) -  builds struct hashes for generating EIP712 digests which are required for signature validations.

#### Roles
16. Trusted Validator - is a trusted third party account that can ensures that transactions on sub-accounts and console accounts are valid. It's signature needs to be verified in PolicyValidator before execution.
17. Guardian - is in charge of all emergency actions to ensure contract security.
18. Governace - is brahma owned, configures addresses, funds accounts and performs a host of other privileged functions.
***
## **Architecture review**
#### Console and Sub account Transactions

1. `execTransaction` is called by a console account without policies and safeguard in the [GnosisSafe.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/lib/safe-contracts/contracts/GnosisSafe.sol) contract. The desired call datas execute without any safety checks;
2. `execTransaction` is called by a console account with policies and safeguard. Here, the `execTransaction` function performs the needed safety checks before and after executing the transactions;
   - The `execTransaction` calls `checkTransaction` function in the [SafeModeratorOverridable.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol);
   - The [SafeModeratorOverridable.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol)'s `checkTransaction` function calls the `validatePreTransactionOverridable` function in the [TransactionValidator.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol);
   - In the [TransactionValidator.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol) contract, `validatePreTransactionOverridable` checks if the guard isn't being removed with the help of the `_isConsoleBeingOverriden`. This checks to see if the required guard removal conditions are met. As, the guard isn't being removed, the `_validatePolicySignature` is called.
   - The `_validatePolicySignature` calls the `isPolicySignatureValid` in the [PolicyValidator.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol) contract. This validates the policyHash and the Trusted Validator Signature.
   - After the check, the desired calldata is executed. 
   - After the calldata is successfully executed, `execTransaction` function calls the `checkAfterExecution` function in [SafeModeratorOverridable.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol) contract. This calls the `validatePostTransactionOverridable` in the [TransactionValidator.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol) contract.

    ###### **Transaction function flow**
    execTransaction &#8594; checkTransaction &#8594; validatePreTransactionOverridable ==> _isConsoleBeingOverriden (No) ==> _validatePolicySignature &#8594; isPolicySignatureValid &#8594; execTransaction (desired calldatas are executed) &#8594; checkAfterExecution &#8594; validatePostTransactionOverridable
    ###### **Transaction contract flow**
    GnosisSafe &#8594; SafeModeratorOverridable &#8594; TransactionValidator <==> TransactionValidator <==> TransactionValidator &#8594; PolicyValidator &#8594; GnosisSafe &#8594; SafeModeratorOverridable &#8594; TransactionValidator

3. `execTransaction` is called by a console account with policies and safeguard, while also disabling the safeguard - This occurs almost as above with the added caveat that the safeguard is overriden. The action of overriding the safeguard skips the policy validation, and disables the guard by setting guard address to 0. However unlike stated in the readme, after guard is disabled. It doesn't seem to perform the checkAfterExecution. This is because before performing checkAfterExecution, there's a [check](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/lib/safe-contracts/contracts/GnosisSafe.sol#L190) to see if guard address isn't 0. If 0, the checkAfterExecution is skipped.     


      ###### ***Transaction function flow***
      execTransaction &#8594; checkTransaction &#8594; validatePreTransactionOverridable ==> _isConsoleBeingOverriden (Yes) ~~== _validatePolicySignature =&#8594; isPolicySignatureValid~~ &#8594; execTransaction (desired calldatas are executed, setGuard(address(0)))
      ###### ***Transaction contract flow***
      GnosisSafe &#8594; SafeModeratorOverridable &#8594; TransactionValidator <==> TransactionValidator ~~&#8594; TransactionValidator =&#8594; PolicyValidator~~ &#8594; GnosisSafe

5.  `execTransactionFromModuleReturnData` is called by the console account via the sub accouunt using the [ModuleManager.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/lib/safe-contracts/contracts/base/ModuleManager.sol). The function executes the desired calldata and returns returnData to ConsoleAccount.

6. `execTransaction` is called by operator through the sub-account - This tranaction occurs just like console account exectransaction (see №2) above. The only difference is that instead of the checks being conducted by the [SafeModeratorOverridable.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol) contract, it is conducted by the [SafeModerator.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModerator.sol) instead.
   - The `execTransaction` calls `checkTransaction` function in the [SafeModerator.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModerator.sol);
   - The [SafeModerator.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModerator.sol) `checkTransaction` function calls the `validatePreTransaction` function in the [TransactionValidator.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol);
   - The [TransactionValidator.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol) contract validates the policy signature through the `_validatePolicySignature`. which calls the `isPolicySignatureValid` in the [PolicyValidator.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol) contract. This validates the policyHash and the Trusted Validator Signature.
   - After the check, the desired calldata is executed. 
   - After the calldata is successfully executed, `execTransaction` function calls the `checkAfterExecution` function in[SafeModerator.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModerator.sol) contract. This calls the `validatePostTransaction` in the [TransactionValidator.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol) contract. This checks the security config by calling the `_checkSubAccountSecurityConfig` function. It makes sure guard, fallbackhandlers and the console has not be disabled as module.    
    ###### ***Transaction function flow***
    execTransaction &#8594; checkTransaction &#8594; validatePreTransaction ==> _validatePolicySignature &#8594; isPolicySignatureValid &#8594; execTransaction (desired calldatas are executed) &#8594; checkAfterExecution &#8594; validatePostTransaction ==> _checkSubAccountSecurityConfig
    ###### ***Transaction contract flow***
    GnosisSafe &#8594; SafeModerator &#8594; TransactionValidator <==> TransactionValidator &#8594; PolicyValidator &#8594; GnosisSafe &#8594; SafeModerator &#8594; TransactionValidator <==> TransactionValidator

7. `executeTransaction` is called the executor on the [ExecutorPlugin.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol) enabled as module on the sub-account.
   - The Executor calls `executeTransaction`, this validates the signer executor by calling the `_validateExecutionRequest`, where it's determined if the executor is valid for the sub-account by checking with the [ExecutorRegistry.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol), validates the executor's signature and validates the sub-account's policy by calling the `validatePreExecutorTransaction` function in the [TransactionValidator.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol) function.
   - Through the `validatePreExecutorTransaction` function, the policy signature is validated by the `_validatePolicySignature`. which calls the `isPolicySignatureValid` in the [PolicyValidator.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol) contract.
   - After a successful check, the [ExecutorPlugin.sol](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol) executes transaction on the sub-account by calling `_executeTxnAsModule` function.
   - After execution, the ExecutorPlugin calls `validatePostExecutorTransaction` function on TransactionValidator. to check that SafeModerator hasn't been removed as guard on SubAccount and ConsoleAccount hasn't been removed as module on SubAccount.

    ###### ***Transaction function flow***
    executeTransaction &#8594; _validateExecutionRequest &#8594; validatePreExecutorTransaction ==> _validatePolicySignature &#8594; isPolicySignatureValid &#8594; executeTransaction (desired calldatas are executed)(_executeTxnAsModule) &#8594; validatePostExecutorTransaction &#8594; _checkSubAccountSecurityConfig
    ###### ***Transaction contract flow***
    ExecutorPlugin <==> ExecutorPlugin (checks with ExecutorRegistry) &#8594; TransactionValidator <==> TransactionValidator &#8594; PolicyValidator &#8594; ExecutorPlugin &#8594; TransactionValidator <==> TransactionValidator

## **Codebase quality analysis**
The codebase is of high quality, well structured and would-be large functions are carefully divided into seperate contracts. Required functions are either directly called or delegatecalled when needed. The codebase is well commented and its documentations are easy to follow. There are also unit tests for the contracts which is very commendable.
Error handling is fine. The codebase uses a mixture of custom, require and reverts errors. In a host of cases, the errors weren't very descriptive, some without messages, some with cryptic messages like "GS102". An error also appears to be unused.
Events were also well handled, some events didn't fully emit needed parameters, some didn't have a lot of declarations or well handled. 
In, general NatSpec and Style guide were followed, not a 100%, but just enough to make have the contract look well organized.
We recommend using linters and static analysis tools can help note and fix these issues before deployment.

## **Systemic risks & Centralization risks**
The risk of centralization seems to be limited to the brahma owned governace and guardian. Any malicious actions (intended or not) can pose a risk to the protocol and its users. 
Some contract inherits thrid party contracts like openzeppelin, solady, gnosissafe contracts. Any issue found in these inheritancies might pose a threat to the protocol.
One thing to also note is that although wallets and sub accounts can be added, they cannot be removed. Not sure if that is intended by the sponsor, but it might cause a potential issue in case of some problem to keep console secure.

## **Conclusion**
We started the audit process by thoroughly reviewing provided documentations, tests noting various important parts and asked the sponsors questions. We used static analysis tool and linters to audit the codebase, noted the bot reports and went about with the manual code inspection. We carefully inspected each section of the codebase, we tested possible attack vectors, focusing both on the sponsor's interested areas of attack (through imported wallets) and areas we deemed important. We took notes of our findings, tested them out and prepared our analysis. Overall, the console contract looks to be well written and  doesn't have a lot of serious issues. We recommend the team goes through these, make appropriate modifications where needed before deployment.





### Time spent:
36 hours