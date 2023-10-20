## Description

The PolicyValidator contract is responsible for checking signature added by the trusted validator. It gets the current nonce for the account in [isPolicySignatureValid](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L63) function and calculates the hash of the message for which the signature is validated.

However, the retrieved nonce is not the current one, because the nonce is increased before the guard contract is called (https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/lib/safe-contracts/contracts/GnosisSafe.sol#L143). It means that the trusted validator's signature must be generated for the nonce greater by one than the current one.

## Impact

Trusted validator does not use the current nonce for which the main signatures were generated what can lead to generation of incorrect signatures if the trusted validator retrieves the nonce before the transaction is submitted or takes the same nonce for which owners generated signatrues. 

## Recommendation

Consider using the previous nonce (lowered by 1) when verifying the signature of trusted verifier in the guard contract.