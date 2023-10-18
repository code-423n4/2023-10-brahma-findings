# Smart contract version cannot be updated
Declaring _VERSION as a constant variable can lead to issues with version numbers not being updated when upgrading later contracts

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L55C29-L55C38

```solidity
    string private constant _NAME = "ExecutorPlugin";
    string private constant _VERSION = "1.0";
```
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L28
```solidity
    string private constant _NAME = "PolicyValidator";
    string private constant _VERSION = "1.0";
```
Declaring VERSION as a constant variable can lead to issues with version numbers not being updated when upgrading later contracts

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L23

```solidity
    string public constant VERSION = "1";
```



# No valid input validation is performed to ensure that a given _wallet address exists in the mapping
In the `getSubAccountsForWallet` function, valid input validation is not performed to ensure that a given _wallet address exists in the mapping. This can lead to problems when calling functions on addresses that do not exist in the mapping.

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L63
## Suggestions and Recommendations:
It is recommended that logic be added to the function to ensure that a given _wallet address exists in the mapping to prevent operations on non-existent addresses. 
```solidity
function getSubAccountsForWallet(address _wallet) external view returns (address[] memory) {
    require(walletToSubAccountList[_wallet].length > 0, "Wallet address not found in mapping");
    return walletToSubAccountList[_wallet];
}
```
