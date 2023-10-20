## [L-01] Documentation and Function Logic are Inconsistent

In the documentation, the [diagram](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/docs/Architecture.md) shows a function called `execTransaction`

<ins>https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L68</ins>

However, in `ExecutorPlugin.sol` the function is called `executeTransaction`

Similarly, `checkAfterExecution` and `validatePostTransactionOverridable` are shown to have upper camel case names in the diagram when they're actually lower camel case.

<ins>https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModerator.sol</ins>
<ins>https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol</ins>