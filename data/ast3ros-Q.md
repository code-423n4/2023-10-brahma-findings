# [L-1] Incorrect Import of GnosisSafeStorage.sol Version

The contract SafeEnabler.sol imports an outdated version of `GnosisSafeStorage.sol`. Specifically, the nonce variable type is incorrectly defined as:

        bytes32 internal nonce;

https://github.com/code-423n4/2023-10-brahma/blob/1c5c0684aeff493cc51923b65426cc4d6abaef27/contracts/src/core/SafeEnabler.sol#L7

The correct type for the nonce variable is uint256:

        uint256 internal nonce;

https://github.com/safe-global/safe-contracts/blob/e57df14ea96dc7dabf93f041c7531f2ab6755c76/contracts/libraries/GnosisSafeStorage.sol#L17

## Recommended Mitigation Steps

Update the SafeEnabler.sol contract to import the latest version of GnosisSafeStorage.sol to ensure the nonce variable is of type uint256.


# [L-2] Unrestricted Wallet Registration in WalletRegistry

In the `WalletRegistry` contract, any account can register itself as a wallet without restrictions. The intention, as per the documentation, is that only the safe deployer or the wallet itself should be able to register. However, this restriction is not implemented in the `registerWallet()` function:

    /**
     * @notice Registers a wallet
     * @dev Can only be called by safe deployer or the wallet itself
     */
    function registerWallet() external {
        if (isWallet[msg.sender]) revert AlreadyRegistered();
        if (subAccountToWallet[msg.sender] != address(0)) revert IsSubAccount();
        isWallet[msg.sender] = true;
        emit RegisterWallet(msg.sender);
    }

Due to this oversight, any account can establish subAccounts with multiple hierarchies. This could potentially result in gas consumption issues. If there are several nested subAccounts, a transaction could hit the gas limit, leading to "out of gas" errors during subAccount execution.

Moreover, if a regular account (which isn't a Gnosis wallet) is registered, it might not trigger the expected policy validation hooks, potentially compromising the integrity of the system.