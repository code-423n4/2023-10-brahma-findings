# QA Report

## Summary

| id | title |
| --- | --- |
| [L-01](#l-01-changing-address-for-safemoderator-or-consolefallbackhandler-will-break-existing-subaccounts) | Changing address for `SafeModerator` or `ConsoleFallbackHandler` will break existing `subAccounts` |
| [L-02](#l-02-no-opt-out-of-creating-a-safe-without-looping-over-nonces) | No opt-out of creating a safe without looping over nonces |
| [R-01](#r-01-confusing-that-two-different-methods-of-determining-owner-are-used) | Confusing that two different methods of determining owner are used |
| [NC-01](#nc-01-misleading-natspec) | Misleading natspec | 
| [NC-02](#nc-02-wrong-bytes-in-comment) | Wrong bytes in comment |
| [NC-03](#nc-03-unused-function)| Unused function |

## Low

### L-01 Changing address for `SafeModerator` or `ConsoleFallbackHandler` will break existing `subAccounts`

During execution of operations on sub accounts, fallback handler and guard are not allowed to change:

[`TransactionValidator::_checkSubAccountSecurityConfig`](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L182-L191):
```solidity
File: contracts/src/core/TransactionValidator.sol

182:        address guard = SafeHelper._getGuard(_subAccount);
183:        address fallbackHandler = SafeHelper._getFallbackHandler(_subAccount);
184:
185:        // Ensure guard has not been disabled
186:        if (guard != AddressProviderService._getAuthorizedAddress(_SAFE_MODERATOR_HASH)) revert InvalidGuard();
187:
188:        // Ensure fallback handler has not been altered
189:        if (fallbackHandler != AddressProviderService._getAuthorizedAddress(_CONSOLE_FALLBACK_HANDLER_HASH)) {
190:            revert InvalidFallbackHandler();
191:        }
```

Here the guard and fallback handler are verified unchanged to what is defined in the address provider service.

The issue is that if [governance changes](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L77-L90) the implementations of either if these, this check will fail for all existing sub accounts. Since they are created with what was the guard and fallback handler at time of safe creation.

The sub accounts are blocked until the console account updates the guard or fallback handler on the sub account.

#### Impact
Changing address of either `SafeModerator` or `ConsoleFallbackHandler` will break every existing `subAccount`. It also forces the owning console accounts to upgrade to the new implementations without any opt-in.

#### Recommendation
Consider storing the values in `pre`-validation and do the comparison against the stored values in `post`-validation. That way they are independent of any changes from the protocol.


### L-02 No opt-out of creating a safe without looping over nonces

When creating new safes, either subAccounts or main console accounts, a nonce is used to create the salt used by the SafeDeployer:

[`SafeDeployer::_createSafe`](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L228-L244) and [`_genNonce`](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L253-L255):
```solidity
228:        uint256 nonce = _genNonce(ownersHash, _salt);
229:        do {
230:            try IGnosisProxyFactory(gnosisProxyFactory).createProxyWithNonce(gnosisSafeSingleton, _initializer, nonce)
231:            returns (address _deployedSafe) {
232:                _safe = _deployedSafe;
233:            } catch Error(string memory reason) {
234:                // KEK
235:                if (keccak256(bytes(reason)) != _SAFE_CREATION_FAILURE_REASON) {
236:                    // A safe is already deployed with the same salt, retry with bumped nonce
237:                    revert SafeProxyCreationFailed();
238:                }
239:                emit SafeProxyCreationFailure(gnosisSafeSingleton, nonce, _initializer);
240:                nonce = _genNonce(ownersHash, _salt);
241:            } catch {
242:                revert SafeProxyCreationFailed();
243:            }
244:        } while (_safe == address(0));

...

253:    function _genNonce(bytes32 _ownersHash, bytes32 _salt) private returns (uint256) {
254:        return uint256(keccak256(abi.encodePacked(_ownersHash, ownerSafeCount[_ownersHash]++, _salt, VERSION)));
255:    }
```

If the wallet creation fails, it retries with a new nonce (and thus a new salt) until it succeeds. This opens up an unlikely but possible attack vector where a rich malevolent actor could front run wallet creation by creating a lot of wallets increasing the gas cost for the victim due to the iterations.

This of course costs even more gas for the attacker as they need to actually create the wallets making this attack unlikely. But still possible.

Since the wallets created by the attacker would be identical, a front run here doesn't really do anything else than what the victim wanted, creating a wallet. Hence the wallet created by a possible attacker would work just as well for the victim.

#### Recommendation
Consider adding a boolean flag if the user wish to iterate over nonces when creating a safe. That way a user can opt-in to having the looping logic or not.


## Recommendations

### R-01 Confusing that two different methods of determining owner are used

[`ExecutorRegistry::registerExecutor`](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol#L40) and [`deRegisterExecutor`](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol#L55) uses two different methods of determining if `msg.sender` is the owner of the `subAccount`:

```solidity
File: contracts/src/core/registries/ExecutorRegistry.sol

40:        if (!_walletRegistry.isOwner(msg.sender, _subAccount)) revert NotOwnerWallet();

55:        if (_walletRegistry.subAccountToWallet(_subAccount) != msg.sender) revert NotOwnerWallet();
```

[`WalletRegistry::isOwner`](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L73-L75) simply does the same check, `subAccountToWallet[_subAccount] == _wallet`.

Using two different methods is confusing.

#### Recommendation
Consider usinng the same method in both calls.

## Informational / Non-critical

### NC-01 Misleading natspec

[`WalletRegistry::registerWallet`](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L31-L40):
```solidity
File: contracts/src/core/registries/WalletRegistry.sol

31:    /**
32:     * @notice Registers a wallet
33:     * @dev Can only be called by safe deployer or the wallet itself
34:     */
```

> `Can only be called by safe deployer or the wallet itself`

It cannot be called by safe deployer as it only instructs the wallet to call it on setup (as it only registers `msg.sender` it would then be the safe deployer address registered as a wallet).

### NC-02 Wrong bytes in comment

while declaring [`SafeHelper._FALLBACK_REMOVAL_CALLDATA_HASH`](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L44):
```solidity
File: contracts/src/libraries/SafeHelper.sol

44:     * abi.encodeCall(IGnosisSafe.setFallbackHandler, (address(0))) = 0xf08a0323000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
```

this is not correct, `abi.encodeCall(IGnosisSafe.setFallbackHandler, (address(0)))` is in fact:
```
0xf08a03230000000000000000000000000000000000000000000000000000000000000000
```

The hash is however calculated on the correct bytes.

### NC-03 Unused function

[`AddressProviderService::_onlyGov`](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol#L62-L66) is not used anywhere in the code.

Consider removing it as having unused functions can cause confusion and is generally considered bad practice.