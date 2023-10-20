# 🛠️ Analysis - Brahma.fi Project 
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? "Solidity-metrics" was used  |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Security Approach of the Project | Audit approach of the Project |
|e) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|f) |Packages and Dependencies Analysis | Details about the project Packages |
|g) |Other recommendations | What is unique? How are the existing patterns used? |
|h) |New insights and learning from this audit | Things learned from the project |



## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2023-10-brahma

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2023-10-brahma#building-and-running)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review| [Brahma.fi](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/docs) |Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/code-423n4/2023-10-brahma#building-slither-security-report)| The project does not currently have a slither result, a slither control was created from scratch |
|5|Test Suits|[Tests](https://github.com/code-423n4/2023-10-brahma#tests)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2023-10-brahma#scope)||
|7|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|
|8|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2023-10-brahma#main-invariants)||

## b) Analysis of the code base

The most important summary in analyzing the code base is the stacking of codes to be analyzed.
In this way, many predictions can be made, including the difficulty levels of the contracts, which one is more important for the auditor, the features they contain that are important for security (payable functions, uses assembly, etc.), the audit cost of the project, and the time to be allocated to the audit;
Uses Consensys Solidity Metrics
-  **Lines:** total lines of the source unit
-  **nLines:** normalized lines of the source unit (e.g. normalizes functions spanning multiple lines)
-  **nSLOC:** normalized source lines of code (only source-code lines; no comments, no blank lines)
-  **Comment Lines:** lines containing single or block comments
-  **Complexity Score:** a custom complexity score derived from code statements that are known to introduce code complexity (branches, loops, calls, external interfaces, ...)

![image](https://github.com/code-423n4/2023-10-brahma/assets/104318932/14a95f4f-d5c9-4721-8ea1-925e5a161af6)


</br>
</br>

## Code Entry Point

What is orchestration layer?
An orchestration layer is a component or framework that coordinates and manages the execution of various tasks or services within a larger system. It acts as an intermediary between different components or services to ensure they work together.

![image](https://github.com/code-423n4/2023-10-brahma/assets/104318932/7d76fd3d-8dcc-45b9-bdd1-0dace9fdc366)
![image](https://github.com/code-423n4/2023-10-brahma/assets/104318932/821a13b0-749f-44f3-84e4-73dd3693be79)
![image](https://github.com/code-423n4/2023-10-brahma/assets/104318932/2bdb2910-ab1f-41cc-bab6-47c2a6438e59)

</br>
</br>

![image](https://github.com/code-423n4/2023-10-brahma/assets/104318932/cdfdd085-b10a-4357-8fce-e5b66458646a)

</br>
</br>

###  SafeDeployer.sol

![image](https://github.com/code-423n4/2023-10-brahma/assets/104318932/8d41f41c-1847-4e2c-b128-633f2ddf1a0c)
</br>
![image](https://github.com/code-423n4/2023-10-brahma/assets/104318932/5f3cd703-c894-4049-b67d-b0fcd0ee37ad)



## c) Test analysis
### What did the project do differently? ;
-   1) In each test file, vulnerability vectors were determined and these were specified in a separate file. Test files were created according to these vectors. This method ensured that the tests were of higher quality. The team did a very high quality job here. For example, a test file consists of 2 files like this.

![image](https://github.com/code-423n4/2023-10-brahma/assets/104318932/0791ee67-de5b-4e8f-8c4f-728b6e59bd27)


### What could they have done better?
- 1) Test suites do not test for re-entrancy
Test teams are testing many functions and variables, but recently, due to the vulnerability in the Vyper Compiler, the hacking of the projects using certain Vyper compiler and losing 50 million $ has revealed the security weakness here. https://cointelegraph.com/news/curve-finance-pools-exploited-over-24-reentrancy-vulnerability

```solidity
lock modifier functions : 3 files

contracts\src\core\ExecutorPlugin.sol:
  68:     function executeTransaction(ExecutionRequest calldata execRequest) external nonReentrant returns (bytes memory) {
  69          _validateExecutionRequest(execRequest);

contracts\src\core\SafeDeployer.sol:
  56:     function deployConsoleAccount(address[] calldata _owners, uint256 _threshold, bytes32 _policyCommit, bytes32 _salt)
  57:         external
  58:         nonReentrant
  59:         returns (address _safe)
  60:     {

  82:     function deploySubAccount(address[] calldata _owners, uint256 _threshold, bytes32 _policyCommit, bytes32 _salt)
  83:         external
  84:         nonReentrant
  85:         returns (address _subAcc)
  86:     {


```
The accuracy of the functions has been tested, but it has not been tested whether the `nonReentrant` modifier in the function works correctly or not. For this, a test must be written that tries to reentrancy and was observed to fail.

Let's take the depositToPor function from the project as an example, we can write any test of this function, it has already been written by the project teams in the test files, but the risk of reentrancy with the lock modifier in this function has not been tested, a test file can be added as follows;

Project File:
```solidity
contracts\src\core\ExecutorPlugin.sol:
  68:     function executeTransaction(ExecutionRequest calldata execRequest) external nonReentrant returns (bytes memory) {
  69          _validateExecutionRequest(execRequest);
```
</br>

Reentrancy Test  File:
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {DSTest} from "forge-std//test.sol"
import {console2} from "forge-std/console2.sol";

import "./ExecutorPlugin.sol"; 

contract ReentrancyAttack {
    ExecutorPlugin victim;

    constructor(address _victim) {
        victim = ExecutorPlugin(_victim);
    }

    // Fallback function to simulate reentrancy
    fallback() external payable {
        victim.depositToPort(address(0x0), 1 ether); // This should fail due to the lock modifier
    }

    function attack() external payable {
        victim.depositToPort(address(this), 1 ether); // This will trigger the fallback function
    }
}

contract TestReentrancyProtection is DSTest {
    ExecutorPlugin ExecutorPluginInstance;
    ReentrancyAttack attacker;

    function setUp() public {
        ExecutorPluginInstance = new ExecutorPlugin();
        attacker = new ReentrancyAttack(address(ExecutorPluginInstance));
    }

    function testReentrancyAttack() public {
        // Fund the attacker contract
        address(attacker).transfer(2 ether);

        // Try the reentrancy attack
        bool success = address(attacker).call(abi.encodeWithSignature("attack()"));

        // The reentrancy attack should fail due to the lock modifier
        assertFalse(success, "Reentrancy attack succeeded!");
    }
}

```

</br>





## d) Security Approach of the Project

### Successful current security understanding of the project;

1 - First they did the main audit from a reputable auditing organization like Ackee/Trust  and resolved all the security concerns in the report

2- They manage the 2nd audit process with an innovative audit such as Code4rena, in which many auditors examine the codes.


### What the project should add in the understanding of Security;

1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)

2- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)

3- Add On-Chain Monitoring System; If On-Chain Monitoring systems such as Forta are added to the project, its security will increase.

For example ; This bot tracks any DEFI transactions in which wrapping, unwrapping, swapping, depositing, or withdrawals occur over a threshold amount. If transactions occur with unusually high token amounts, the bot sends out an alert. https://app.forta.network/bot/0x7f9afc392329ed5a473bcf304565adf9c2588ba4bc060f7d215519005b8303e3

4- After the Code4rena audit is completed and the project is live, I recommend the audit process to continue, projects like immunefi do this. https://immunefi.com/

5- Pause Mechanism
This is a chaotic situation, which can be thought of as a choice between decentralization and security. Having a pause mechanism makes sense in order not to damage user funds in case of a possible problem in the project.

6- Upgradability
There are use cases of the Upgradable pattern in defi projects using mathematical models, but it is a design and security option.



## e) Other Audit Reports and Automated Findings 

**Automated Findings:**
https://github.com/code-423n4/2023-10-brahma/blob/main/bot-report.md

**Slither Report:**
https://github.com/code-423n4/2023-10-brahma#building-slither-security-report

**Other Audit Reports (Ackee - Trust):**
[Ackee Audit Report](https://github.com/Brahma-fi/brahma-security/blob/master/audits/brahma-fi-consolev2-audit-10-23-ackee.pdf)

[Trust Audit Report](https://github.com/Brahma-fi/brahma-security/blob/master/audits/brahma-fi-console-audit-2023-05-trust.pdf)


##  e) Packages and Dependencies Analysis 📦

| Package                                                                                                                                     | Version                                                                                                               | Usage in the project                               | Audit Recommendation                                                                                                             |
| ------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| [`openzeppelin`](https://www.npmjs.com/package/@openzeppelin/contracts)         | [![npm](https://img.shields.io/npm/v/@openzeppelin/contracts.svg)](https://www.npmjs.com/package/@openzeppelin/contracts)     |       openzeppelin-contracts/security/ReentrancyGuard.sol                          ,openzeppelin-contracts/utils/structs/EnumerableSet.sol |-  Version `4.9.3` is used by the project, it is recommended to use the newest version `5.0.0`                                                                                          |
| [`solady`](https://www.npmjs.com/package/solady)                                                                                        | [![npm](https://img.shields.io/npm/v/solady)](https://www.npmjs.com/package/solady)   | SignatureCheckerLib.sol, </br> EIP712.sol,   |  - The latest updated version is used.  </br> - An updated audit of the solady library was done by SpearBit last week, I recommend checking this out [Audit Report](https://github.com/Vectorized/solady/blob/main/audits/cantina-solady-report.pdf)                                                                                  |
| [`safe-contracts`](https://www.npmjs.com/package/@safe-global/safe-contracts)                                                                                        | [![npm](https://img.shields.io/npm/v/@safe-global/safe-contracts)](https://www.npmjs.com/package/@safe-global/safe-contracts)   | GnosisSafe.sol, </br> GnosisSafeStorage.sol,  </br> DefaultCallbackHandler.sol, </br> ISignatureValidator.sol |   -  Version `1.3.0` is used by the project, it is recommended to use the newest version `1.4.0`     |     


## f) Other recommendations

✅ The use of assembly in project codes is very low, I especially recommend using such useful and gas-optimized code patterns; https://github.com/dragonfly-xyz/useful-solidity-patterns/tree/main/patterns/assembly-tricks-1

✅ It is seen that the latest versions of imported important libraries such as Openzeppelin are not used in the project codes, it should be noted. https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts/

✅A good model can be used to systematically assess the risk of the project, for example this modeling is recommended; https://www.notion.so/Smart-Contract-Risk-Assessment-3b067bc099ce4c31a35ef28b011b92ff#7770b3b385444779bf11e677f16e101e






## g) New insights and learning from this audit 

1. Enhanced User Experience in DeFi:
Brahma Console v2 is focused on improving the user experience in decentralized finance (DeFi) for smart contract wallet users. By integrating with Gnosis Safe, it allows users to configure automated strategies for common DeFi operations, reducing the complexity and effort required to interact with various DeFi protocols.

2. Security and Custody:
Security is a core feature of the Brahma Console. Users retain full custody of their funds while utilizing the platform’s automation features. The architecture ensures that users don't have to compromise on security for the sake of convenience, striking a balance that is crucial in the DeFi space.

3. Risk Mitigation with Sub-accounts:
Users have access to SafeSub-accounts, a feature designed to reduce risks associated with interacting with various protocols. These sub-accounts isolate interactions, creating a layer of security that protects the main console account from potential vulnerabilities or adverse effects that might arise during transactions or interactions with DeFi protocols.

4. Flexible Control Mechanism:
The governance and control mechanisms are clearly defined and flexible. Console Accounts, operated by multiple users, have overarching authority, while SubAccounts are managed by delegated Operator accounts. Operators have restricted rights, managed by SafeModerator, ensuring that control is distributed, but within a secure and defined boundary. Executors are accounts authorized to make module transactions on a subAccount, providing another layer of flexibility and control.

5. Asset Management(for Funds, DAOs)
![image](https://github.com/code-423n4/2023-10-brahma/assets/104318932/94b681f4-654c-459b-8284-39af4b709b31)







### Time spent:
16 hours

### Time spent:
16 hours