In WalletRegistry.sol, the registerSubAccount() method should check address _wallet != address _subAccount.
This will prevent a subaccount being the same as console account address.
```sol
  function registerSubAccount(address _wallet, address _subAccount) external {
        if (msg.sender != AddressProviderService._getAuthorizedAddress(_SAFE_DEPLOYER_HASH)) revert InvalidSender();
        if (subAccountToWallet[_subAccount] != address(0)) revert AlreadyRegistered();
        subAccountToWallet[_subAccount] = _wallet;
        walletToSubAccountList[_wallet].push(_subAccount);
        emit RegisterSubAccount(_wallet, _subAccount);
    }
```