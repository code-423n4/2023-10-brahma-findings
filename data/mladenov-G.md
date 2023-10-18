# [G-01] Use modifiers instead functions

### Description
There are functions for checking zero address,governance address and etc. These function can be replaced with modifiers.
Modifiers will use less gas becuase they will be executed before function call and if there is an error the function won't be executed.


### Context

[AddressProvider.sol#130](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L130)

[AddressProvider.sol#139](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L139)

[AddressProvider.sol#147](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L147)

### Migrations steps
Create modifiers and attach them to all functions which need to use these modifiers

Example:
```solidity
modifier _onlyGov() {
    if (msg.sender != governance) revert NotGovernance(msg.sender);
    _;
}

function setGovernance(address _newGovernance) external _onlyGov {
        _notNull(_newGovernance);
        emit GovernanceTransferRequested(governance, _newGovernance);
        pendingGovernance = _newGovernance;
    }

```

## Tools Used

Manual review

Remix Ide