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

2- https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeDeployer.sol#L56-L57

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeDeployer.sol#L82-L86
```
    function deployConsoleAccount(address[] cacalldatalldata _owners, uint256 _threshold, bytes32 _policyCommit, bytes32 _salt)
```
```
    function deploySubAccount(address[] calldata _owners, uint256 _threshold, bytes32 _policyCommit, bytes32 _salt)
        external
        nonReentrant
        returns (address _subAcc)
```

The follow functions aren't checking the length of the array passed. It is advisable to check the length of the array of it's greater than zero before performing any iteration in the array.
