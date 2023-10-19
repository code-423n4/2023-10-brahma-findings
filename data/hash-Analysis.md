## Approach
1. Gained high level understanding of the system through the docs and `Readme` provided in the C4 repo
2. Went through the codebase associating the code portions with the functionalities
3. Analysed each section identified earlier more in depth and kept track of anomalies
4. Explored various ways the anomalies could be exploited
5. Checked `Safe` contracts for possible integration vulnerabilities with the system
6. Went through code line-by-line to check for gas optimizations and non-critical issues.   

## Mechanism Review
The system allows user's to distribute functionality/permissions across a set of subaccount's and is designed to enhance the DeFi experience on smart contract wallets. User's own console accounts which are `Safe` wallets and can create subaccount's in which the console account will be a module without restrictions essentially giving complete authority of the subaccount to the owner. The owner can enable a set of operators and executors for the subaccount. The allowable actions for these actors are managed using a `policyHash`. It requires that the operation/transaction being performed by any of these actor's must be approved by a trusted validator in accordance with the policy set by the owner. The implementation details of how the policies set by the owner is enforced by the validator's signature is not in scope of the review and is a major component of the system. 

All accounts (console and subaccounts) are created using the `createProxyWithNonce()` function of `SafeFactory`. This opens up possibilities of front running the account creation and utilizing the obtained address in various ways. No issue related with this could be identified now as the risk associated with front-runned subaccounts are managed with the guard and the front-run of console account creation can be an attack only due to bad configurations/ improper deployment process which is common to all `SafeWallet`s.

## Systemic risk
Since the entire suite is unupgradable, in case of any future exploit on shared contracts like PolicyValidator, SafeModerator, Registries etc., all user's might have to themselves remove the interaction with affected contract by changing the guards (for subaccounts).
The trust on validator.
 
## Centralization risks
Collusion between a validator and an operator/executor of a subaccount has the potential to  take over a subaccount and execute arbitrary transactions.

### Time spent:
15 hours