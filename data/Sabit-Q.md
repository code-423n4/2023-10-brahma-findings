1. Redundant SafeProxyCreationFailed()

`
do {
            try IGnosisProxyFactory(gnosisProxyFactory).createProxyWithNonce(gnosisSafeSingleton, _initializer, nonce)
            returns (address _deployedSafe) {
                _safe = _deployedSafe;
            } catch Error(string memory reason) {
                // KEK
                if (keccak256(bytes(reason)) != _SAFE_CREATION_FAILURE_REASON) {
                    // A safe is already deployed with the same salt, retry with bumped nonce
                    revert SafeProxyCreationFailed();
                }
                emit SafeProxyCreationFailure(gnosisSafeSingleton, nonce, _initializer);
                nonce = _genNonce(ownersHash, _salt);
            } catch {
                revert SafeProxyCreationFailed();
            }
`

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeDeployer.sol#L234-L243

There are two revert SafeProxyCreationFailed() statements - one inside the catch block and one outside. This is redundant, one statement would suffice.