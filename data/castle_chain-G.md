## perform the checks first will save gas in case of revert . 
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ExecutorPlugin.sol#L117-L119

```solidity
        if (execRequest.executor.code.length == 0 && execRequest.executorSignature.length == 0) {
            // Executor is an EOA and no executor signature is provided
            revert InvalidSignature();
```
in the contract `executorPlugin` , after the execution alot of the logic in the function this check might revert so the check should be performed first  . 

also in the `policyValidator` contract the check should performed before the logic 
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/PolicyValidator.sol#L135-L138
```solidity 
    function isPolicySignatureValid(address account, bytes32 transactionStructHash, bytes calldata signatures)
        public
        view
        returns (bool)
    {
        // Get policy hash from registry
        bytes32 policyHash =
            PolicyRegistry(AddressProviderService._getRegistry(_POLICY_REGISTRY_HASH)).commitments(account);
        if (policyHash == bytes32(0)) {
            revert NoPolicyCommit();
        }


        // Get expiry epoch and validator signature from signatures
        (uint32 expiryEpoch, bytes memory validatorSignature) = _decompileSignatures(signatures);


        // Ensure transaction has not expired
        if (expiryEpoch < uint32(block.timestamp)) {
            revert TxnExpired(expiryEpoch);
        }


        // Build validation struct hash
        bytes32 validationStructHash = TypeHashHelper._buildValidationStructHash(
            TypeHashHelper.Validation({
                transactionStructHash: transactionStructHash,
                policyHash: policyHash,
                expiryEpoch: expiryEpoch
            })
        );


        // Build EIP712 digest with validation struct hash
        bytes32 txnValidityDigest = _hashTypedData(validationStructHash);


        address trustedValidator = AddressProviderService._getAuthorizedAddress(_TRUSTED_VALIDATOR_HASH);


        // Empty Signature check for EOA signer
        if (trustedValidator.code.length == 0 && validatorSignature.length == 0) {
            // TrustedValidator is an EOA and no trustedValidator signature is provided
            revert InvalidSignature();
        }


        // Validate signature
        return SignatureCheckerLib.isValidSignatureNow(trustedValidator, txnValidityDigest, validatorSignature);
    }
``` 
## check the mapping `subAccountToWallet` directly without call the function `isOwner()` will save some gas .

check the mapping directly will save gas because the function `isOwner()` check the mapping and return the value and then compare the results . 
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/ExecutorRegistry.sol#L40

and the function `deRegisterExecutor` check the mapping that determine whether the account is the owner or not 
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/ExecutorRegistry.sol#L55
```
        if (_walletRegistry.subAccountToWallet(_subAccount) != msg.sender) revert NotOwnerWallet();
``` 

