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

