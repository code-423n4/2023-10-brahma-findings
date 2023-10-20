## QA-01. ExecutorRegistry.registerExecutor and deRegisterExecutor functions are costly to do when you need to handle several executors
## Description
ExecutorRegistry allows owner to register executors for subaccounts that can initiate transactions on thise subaccounts. In order to add executor [`ExecutorRegistry.registerExecutor` function is used](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol#L38-L44) and this function allows to add only 1 executor per time. As result main console wallet should do a lot of txs if it wants to add several executors, but not only 1. Because of that, such transaction will be very costly, especially on ethereum.
## Recommendation
Better make `registerExecutor` and `deRegisterExecutor` receive array as param of executors.

## QA-02. Subaccounts signatures may not work through eip1271
## Description
When any transaction is executed on Safe wallet, then it's `nonce` is increased.
PolicyValidator [uses safe's nonce](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L63) in order to validate signatures. This means that every time when subaccount requests transaction execution or transaction approve(signature to be used by other wallet), then trusted validator fetches its safe's nonce and signs message with this nonce inside, so later it can be checked.

`ConsoleFallbackHandler.isValidSignature` implements eip1271, so anyone can query Safe if signature is valid. Protocol has implemented their own fallback [in order to add policy check](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L49-L51). This check will do exactly same as for tx's checking. It will fetch safe's nonce in order to check if signature is valid.

So how this can be used? For example subaccount is owner of another Safe wallet and that Safe wallet would like to execute tx. So owners start to sign signatures, which can take some time to get them all. During the time when all signatures will be collected and `ConsoleFallbackHandler.isValidSignature` will be called on subaccount, subaccount may execute another tx(that will use same nonce), which will increase safe's nonce and as result will invalidate previous signature, so tx on another wallet will not be executed and new signature will be needed.

## Recommendation
Do not know good one. Maybe it's a non-issue at all, but would like protocol to see this.