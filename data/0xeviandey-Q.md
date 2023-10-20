### [L-01] Consider removing duplicate storage of nounce
In the SafeDeployer contract the internal function _createSafe have a duplicate removing the second duplicate since it was already stored above.

nounce calls getnounce passing in the _ownersHash and _salt and the returned value is stored as an unsigned integer

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L228

There is no need to to store nounce again here

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L240

Consider removing the duplicate.





