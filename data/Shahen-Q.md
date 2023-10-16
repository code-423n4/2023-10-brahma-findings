### As per the solidity documentation have mentioned about packing dynamic types will produce collisions

- Even though VERSION is a constant variable ([Line 23](https://github.com/code-423n4/2023-10-brahma/blob/a6424230052fc47c4215200c19a8eef9b07dfccc/contracts/src/core/SafeDeployer.sol#L23)) it's still a dynamic type in solidity. Either use a fixed length string for the VERSION variable or you can hash the string before packing it to ensure that the length of the packed data is always the same and will prevent collisions in ([Line 254](https://github.com/code-423n4/2023-10-brahma/blob/a6424230052fc47c4215200c19a8eef9b07dfccc/contracts/src/core/SafeDeployer.sol#L254)) in the `_getNonce()` function.


https://github.com/code-423n4/2023-10-brahma/blob/a6424230052fc47c4215200c19a8eef9b07dfccc/contracts/src/core/SafeDeployer.sol#L23

https://github.com/code-423n4/2023-10-brahma/blob/a6424230052fc47c4215200c19a8eef9b07dfccc/contracts/src/core/SafeDeployer.sol#L254