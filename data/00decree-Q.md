#### [Non-critical] rename `_owners` to `_operators` for sake of clarity and avoid confusion.
In the doc it was mentioned that "A gnosis safe operated by the delegatee accounts called `Operators`", but `Operators` is not clearly visible/understandable in the code. 

Rename the param and natspec `_owners` to `_operators` at `deploySubAccount()` and `_setupSubAccount()` for clarity.
```
File: SafeDeployer.sol

077:      * @param _owners list of safe owners
...
083:     function deploySubAccount(address[] calldata _owners, uint256 _threshold, bytes32 _policyCommit, bytes32 _salt)
...
087:     {
...
095:         // Deploy sub account
096:         _subAcc = _createSafe(_owners, _setupSubAccount(_owners, _threshold, msg.sender), _salt);
...
104:     }
```
github: [link](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeDeployer.sol#L73-L103)

```
File: SafeDeployer.sol

165:      * @param _owners list of owners addresses
168:      */
169:     function _setupSubAccount(address[] memory _owners, uint256 _threshold, address _consoleAccount)
...
173:     {
...
193:         return abi.encodeCall(
194:             IGnosisSafe.setup,
195:             (
196:                 _owners,
...
204:             )
205:         );
206:     }

```
github: [link](https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeDeployer.sol#L162C1-L205)

---
#### [Non-critical] Outdated documentation flowchart image in `Architecture.md`
[github-link1](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/docs/images/console%20account%20execTransaction.png)
`_isGuardRemovalTransaction()` is not found in codebase, replace with `_isConsoleBeingOverriden()`.

[github-link2](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/docs/images/console-guard-removal.png)
`_isGuardBeingRemoved()` is not found in codebase and, replace with `_isConsoleBeingOverriden()`. Issue involving `_isGuardBeingRemoved` in previous audit report [link](https://github.com/Brahma-fi/brahma-security/blob/master/audits/brahma-fi-consolev2-audit-10-23-ackee.pdf) was marked as fixed, possibly removed from codebase.
