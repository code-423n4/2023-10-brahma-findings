- Check if `_newGovernance == governance` and revert to avoid unnecessary emit.

https://github.com/code-423n4/2023-10-brahma/blob/a6424230052fc47c4215200c19a8eef9b07dfccc/contracts/src/core/AddressProvider.sol#L52-L57

- The functions don't follow CEI pattern -

https://github.com/code-423n4/2023-10-brahma/blob/a6424230052fc47c4215200c19a8eef9b07dfccc/contracts/src/core/AddressProvider.sol#L52-L57

https://github.com/code-423n4/2023-10-brahma/blob/a6424230052fc47c4215200c19a8eef9b07dfccc/contracts/src/core/AddressProvider.sol#L62-L69

- The contract should be deployed with a timelock to allow for changes to be made in a controlled manner.

- The address(0) check missing on `_executor` address

https://github.com/code-423n4/2023-10-brahma/blob/a6424230052fc47c4215200c19a8eef9b07dfccc/contracts/src/core/registries/ExecutorRegistry.sol#L38-L44

- The `isOwner()` will return true if `_wallet == addr0` AND `_subAccount == addr0`, when actually it should revert.

https://github.com/code-423n4/2023-10-brahma/blob/a6424230052fc47c4215200c19a8eef9b07dfccc/contracts/src/core/registries/WalletRegistry.sol#L67