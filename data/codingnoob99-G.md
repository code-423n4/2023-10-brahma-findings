The function setAuthorizedAddress has a boolean wich is really expensive. A uint is cheaper and can be used the same way.
//function setAuthorizedAddress(bytes32 _key, address _authorizedAddress, bool _overrideCheck)
Instead of Using _overrideCheck as a bool to check true/false use a uint to check 0/1.


PoC
testing:testFail1(bytes32,address,bool) (runs: 256, μ: 5251, ~: 5251)
//The testFail1 is with the bool and the testFail is with the uint
testing:testFail(bytes32,address,uint40) (runs: 256, μ: 5222, ~: 5222)

Recommended:
if (!_overrideCheck)
if (_overrideCheck == 1)

https://github.com/code-423n4/2023-10-brahma/blob/a6424230052fc47c4215200c19a8eef9b07dfccc/contracts/src/core/AddressProvider.sol#L77