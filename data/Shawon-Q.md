# Missing parameter valuation 
Consider using zero address validation for costructors and functions as assinging accidental null values can create unexpected situations and
force for redeployment (policyRegistry,ExecutorRegistry.sol,SafeDeployer.sol,TransactionValidator.sol,SafeModeratorOverridable.sol,SafeModerator.sol,ConsoleFallbackHandler.sol and others)
```
 constructor(address _addressProvider) AddressProviderService(_addressProvider) {}
```

# Missing modifier for external functions 

function `registerWallet()`  has this dev comment "Can only be called by safe deployer or the wallet itself".
But we don't see any kind of require statements or function modifier regarding this comment.
Consider removing this comment or adding an if or require statement
``` 
file: core/registries/WalletRegistry.sol

 function registerWallet() external {
        if (isWallet[msg.sender]) revert AlreadyRegistered();
        if (subAccountToWallet[msg.sender] != address(0)) revert IsSubAccount();
        isWallet[msg.sender] = true;
        emit RegisterWallet(msg.sender);
    } 
```

# _notNull function inside constructor

```
file: core/AddressProviderService.sol

27 constructor(address _addressProvider) {
28       if (_addressProvider == address(0)) revert InvalidAddressProvider();
29      addressProvider = AddressProvider(_addressProvider);
30  }

72 function _notNull(address _addr) internal pure {
73        if (_addr == address(0)) revert InvalidAddress();
74   }

```
Inside the constructor,we see that there is a input validator used using if statement.
But this contract already has a `_notNull` function which does exactly the same thing.
Consider removing this if statement and calling the `_notNull` function from the constructor

