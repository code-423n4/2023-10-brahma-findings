[N-01] 
Additional getters of public mappings are implemented. Either directly call the public mapping to retrieve the desired values, or make the mappings internal or private.

There are № instances of this issue:

```
File: src/core/AddressProvider.sol
function getAuthorizedAddress(bytes32 _key) external view returns (address) {
        return authorizedAddresses[_key];
    }
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L112-L114

```
File: src/core/AddressProvider.sol
 function getRegistry(bytes32 _key) external view returns (address) {
        return registries[_key];
    }
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L121-L123


