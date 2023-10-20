# 1) overview 
the protocol uses a well-structured codebase , to enable the automation and the delegation of the execution of the transactions and allow the executors and the operators to perform defi activities on the subAccount that delegated them , by 
1) register the main safe as a console on the wallet registry , and then deploy the subAccount 
2) then the subAccount should enable the executorPlugin as a module on it 
3) the console should register some executors to execute the transaction 
5) the executors will use the contract `executorPlugin` to request the execution 
6) the request should be validated by the executor signature and the policy with the validator signature . 


# 2) centralization risks 
### depending on trusted validator as a single contract or EOA will cause centralization risks and should be a network of nodes that validate the transactions with the policy 
the protocol should build a network contains multiple nodes to verify the policy with the transaction and guarantee that the verification process done correctly without any manipulation by a malicious party 
# 3) evaluating the codebase (protocol) . 
## adding a function to allow the mainConsole to remove a subAccount from the wallet registry 
consider adding this function to the wallet registry contract to allow the main console`s owners to remove the sub account in case the operators be malicious . 

## allow setting the executors by the operators since they are restricted by the policy . 
the protocol only allow the owners of the main console to register the executor , it will be better to allow the operators to set the executors since the sub account is always protected by the policy and the trusted validator , so this will enhance the automation of the process by letting the operators set the executors . 

## add the logic to enable the executorPlugin as a module on the subAccount , when setup the subAccount . 
enable the `executorPlugin` contract as a module on the subAccount is a necessary step to allow the executors to execute the transaction on the subAccount so this step should be added in the setup by calling the safeEnabler contract and enable the executorPlugin as module 
this will enhance the user experience and will allow the immediate activity of the automation process . 


# 3) comments for the judge to contextualize my findings
## allow the main console to change the fallback handler to custom or more enhanced one , and the automation process to keep working after this change . 

since the consoleFallBackHandler`s functionality is to validate the ERC-1271 signatures and it has no role in the validation process of the transactions , and the protocol allow the main console to modify the handler but this will freeze the functionality of the executorPlugin , so to achieve a better user experience the protocol can allow this modification , to allow the user to add more logic to the fallback hooks of transactions . 

consider remove this check 
```solidity 
        if (fallbackHandler != AddressProviderService._getAuthorizedAddress(_CONSOLE_FALLBACK_HANDLER_HASH)) {
            revert InvalidFallbackHandler();
        }
```

## allow the executor to set the expiry epoch to allow him to perform certain strategies 
as the executor is the responsible for the defi interaction and the automation process and he is the party that build the transaction and execute it , the executor should set the expiry Epoch to make sure that  the transaction will be executed within an appropriate time so the strategy that the executor perform is done correctly and the transaction has not been executed with a stale data , so this will enhance all the automation process and will evaluate the implementation of the strategies by the executors . 

the recommendations are to allow the executor to sign the expiry epoch with the transaction and validate it before the execution of the transactions . 

# 4) Architecture recommendations
## adding a function to set the guard as safeModerator and the fallbackHandler as consoleFallbackHandler of the console when the wallet is registered and make it as an option represented by a bool value . 

in the wallet registry consider adding a function to optionally set the guard of the main console as the safeModerator contract and the fallbackHandler as the ConsoleFallbackHandler in case of the safe is registered directly from the wallet registry not the safeDeployer contract . 





### Time spent:
60 hours