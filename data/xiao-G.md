# Gas Optimizations

## [G‑01] Fill slots with uint
If a boolean value true or false is stored in a memory slot, it actually takes up the entire memory slot. If you use uint256(1) and uint256(2), they make fuller use of the memory slot.

```solidity
File: src/core/AddressProvider.sol

77:  function setAuthorizedAddress(bytes32 _key, address _authorizedAddress, bool _overrideCheck) external 
```

```solidity
File: src/core/SafeDeployer.sol

110:  function _setupConsoleAccount(address[] memory _owners, uint256 _threshold, bool _policyHashValid) private view
```

```solidity
File: src/core/SafeModerator.sol

70:  function checkAfterExecution(bytes32 txHash, bool success) external view override 
```

```solidity
File: src/core/SafeModeratorOverridable.sol

76:  function checkAfterExecution(bytes32 txHash, bool success) external view override
```

## [G‑02]Using bytes will save gas more than string
As a rule of thumb, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data. If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.

```solidity
File: src/core/ExecutorPlugin.sol

53:  string private constant _NAME = "ExecutorPlugin";
     string private constant _VERSION = "1.0";
```

```solidity
File: src/core/SafeDeployer.sol

23:  string public constant VERSION = "1";
```

```solidity
File: src/core/PolicyValidator.sol

26:  string private constant _NAME = "PolicyValidator";
     string private constant _VERSION = "1.0";
```

## [G‑03]Use indexed events as they are less costly compared to non-indexed ones
Using the indexed keyword for value types such as uint, bool, and address saves gas costs, as seen in the example below.

```solidity
File: src/core/SafeEnabler.sol

19:  event EnabledModule(address module);
     event ChangedGuard(address guard);
```
## [G‑04]Use Modifiers Instead of Functions To Save Gas


```solidity
File: src/core/AddressProvider.sol

139:  function _onlyGov() internal view {
        if (msg.sender != governance) revert NotGovernance(msg.sender);
    }

147:  function _notNull(address addr) internal pure {
        if (addr == address(0)) revert NullAddress();
    }
```
### Use functions to consume gas:
gas	55236 gas
transaction cost	48031 gas 
execution cost	26599 gas 

### Use modifiers to consume gas:✅
gas	55181 gas
transaction cost	47983 gas 
execution cost	26551 gas 

