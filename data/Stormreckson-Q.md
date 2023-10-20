https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/WalletRegistry.sol#L35-L42

The `registerWallet` function doesn't check if the `safeDeployer` contract address  that initiated the call is the authorized address.

```
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

```
As stated in the function comments the call should "only be made by the `SafeDeployer.sol` contract and the wallet address its self" validation for the safe deployer address isn't made. It is advised to check the `SafeDeployer.sol` address against the `AddressProviderSerivce.sol` `SAFE_DEPLOYER_HASH` to ensure only valid `SafeDeployer.sol` contract addresses can call the function mitigating any future risk of compromise.