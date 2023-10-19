**[L-01]** In `safeDeployer.deployConsoleAccount`, a user can add duplicate owners in the list of owners for a wallet either mistakenly or intentionally for malicious reasons. 

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeDeployer.sol#L56 

**Recommendation**
Add mechanism to make sure owners are not duplicated in the list.

**[L-02]** In an event where any of other owners mistakenly approve a transaction, there is no way to revoke the approval. 

**Recommendation**
Include a mechanism to revoke approval before the transaction is executed. 

**[N-01]** In safe deployer.sol, `preComputeAccount event` and `preComputeAccount custom error` is declared but never used

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeDeployer.sol#L35 

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeDeployer.sol#L39 

**Recommendation**
Remove these event and custom error if they are not being used

**[N-02]** Consider making `_onlyGov`  function in AddressProvider.sol a modifier instead. 
Modifiers provide a clean, efficient, and readable way to apply common logic to multiple functions and usually a common convention to implement access control. 

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/AddressProvider.sol#L139 

**[N-03]** Coding style in WalletRegistry.sol
According to solidity style guide, In order of layout of contracts, events should come before custom errors.  Non-critical but its good practice to following this layout convention thought the codebase. 


