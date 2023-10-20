# Address Provider system lack of versioning/backward compatibility may cause safes to break during core contract updates.

Since the AdressProvider system does not impose any interface backward compatibility nor enable batch updates, core contract changes that involve more than one contract can break existing safes using previous versions of the core contracts.

## POC
Say a change in made in the ExecutorRegistry contract that requires changes in calling contracts including the ExecutorPlugin contract. Existing Sub Account safes that have the previous version of the ExecutorPlugin registered as a mudule can not be automatically updated, resulting in functions reveting (due to potential interface changes) or unexpected results due to changed logic, even if the interface didn't change. Furthermore, additional dependant contracts that are accessed through the AddressProvider might break as well due to the fact that there is no way to batch update multiple core contracts in one transaction.

## Tools Used
Foundry, VSCode

## Recommended Mitigation Steps
As a first step, functionality can be added to batch update addresses to avoid inconsistenties. Possible interface can be:  
```function setAuthorizedAddress(bytes32[] _keys, address[] _authorizedAddresses, bool _overrideCheck) external {```

To solve the problem of consistency with previous-version core contracts registed as modules it might be necessary to replace the AddressProvider system with a more standard contract upgradability system that maintains the contract address while changing/Adding functionality.

