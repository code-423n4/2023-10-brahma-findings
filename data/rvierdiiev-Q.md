## QA-01. ExecutorRegistry.registerExecutor and deRegisterExecutor functions are costly to do when you need to handle several executors
## Description
ExecutorRegistry allows owner to register executors for subaccounts that can initiate transactions on thise subaccounts. In order to add executor [`ExecutorRegistry.registerExecutor` function is used](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol#L38-L44) and this function allows to add only 1 executor per time. As result main console wallet should do a lot of txs if it wants to add several executors, but not only 1. Because of that, such transaction will be very costly, especially on ethereum.
## Recommendation
Better make `registerExecutor` and `deRegisterExecutor` receive array as param of executors.