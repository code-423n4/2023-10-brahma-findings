1. Wrong revert message in registerWallet function

In registerWallet function of WalletRegistry contract, particularly line 36 below:

     if (isWallet[msg.sender]) revert AlreadyRegistered();

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/WalletRegistry.sol#L36

The isWallet mapping store which addresses are registered wallets. If the msg.sender passed inside the isWallet mapping is not stored in it, it reverts. 

However, the revert message is wrong - AlreadyRegistered();

Instead, the revert message should be not NotRegistered();