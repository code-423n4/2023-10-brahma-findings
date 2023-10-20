## [L-01] SafeDeployer's deployConsoleAccount and deploySubAccount can be front-run

The `deployConsoleAccount` and `deploySubAccount` methods in `SafeDeployer` accept a `salt` parameter to give newly-created safes a predictable address, ideally identical across chains. This expectation can however be broken by any actor front-running the creation of a safe with the same parameters, and a user who doesn't check the address returned by these functions may be tricked into using the safe created by a third-party.

This does however not seem to pose a significant security risk, because for an actor to create a safe at the exact address expected by the user, the same owner, initialization, and salt have to be provided; these constraints make the newly-created Gnosis contract secure even if adopted by mistake.

## [L-02] WalletRegistry's registerWallet is permissionless

This allows any address, including EOAs, to register themselves as a wallet. While this does not seem to pose a direct threat to the in-scope contracts, it is possible that other contracts use this information to allow them restricted actions.

## [I-01] SafeEnabler.sol is not necessary

When setting up the Sub accounts, there's no need to make a delegateCall to SafeEnabler: the Sub safe can make a self-call because the same methods (enable module & set guard) are already available with a regular call to self.

## [I-02] Ownership checks in ExecutorRegistry are inconsistent

When checking for authorized callers in `ExecutorRegistry`, `registerExecutor` uses `_walletRegistry.isOwner` while `deRegisterExecutor` uses `subAccountToWallet` for the same purpose. Consider harmonizing the checks to improve code readability.

