## Summary

### Low Issues

Total of **5 issues**:

|ID|Issue|
|:--:|:---|
| [L-01] | Attacker can deploy bait `Console` and `SubAccount`s with his own address |
| [L-02] | Anyone can deploy to the same address of any `Console` on different chains |
| [L-03] | Anyone can deploy to the same address of `Console` with different `policy` cross-chain |
| [L-04] | `SafeDeployer` - Unsafe threshold is allowed |
| [L-05] | Broken invariants: Main Console Account removals |


### Non Critical Issues

Total of **5 issues**:

|ID|Issue|
|:--:|:---|
| [NC-01] | `SubAccount` can be deployed to same address with different `policy` cross-chain |
| [NC-02] | Any address can register as `wallet`  in `WalletRegistry` |
| [NC-03] | There is no way to deregister `subAccount` or `wallet` in `WalletRegistry` |
| [NC-04] | `_ensureAddressProvider()` doesn't completely ensure address is address provider |
| [NC-05] | Caution with `Safe` deployments and `AddressProvider` registry |

---


## Low Issues
### [L-01] - Attacker can deploy bait `Console` and `SubAccount`s with his own address
[`src/core/SafeDeployer.sol`](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L56-L103)

An attacker can deploy fake `Console` and `SubAccount`s that are very similar to real ones: they have the same `threshold`, `policyCommit`, `salt`, the only difference is that one of `owners` is the attacker's address. If operators would accidentally start using one of these bait `Safe`s, user funds can be lost.

**Recommendation**: Warn users in the documentation and via comments in code against this attack vector.

---

### [L-02] - Anyone can deploy to the same address of any `Console` on different chains
`SafeDeployer.sol` - [`deployConsole()`](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L56-L71)

The address of a deployed `Console` won't change if it's deployed with the same parameters with the same owners nonce on different chains. This would be undesirable for the original `Console` owners who want to deploy their `Safe` cross-chain to the same address for convenience.

**Recommendation**: Consider using an additional parameter in `_genNonce()` (used in `_createSafe()`) to generate a unique `nonce`. This will prevent a bad actor to deploy to the same address since the address generated will be based on the sender.


### [L-03] - Anyone can deploy to the same address of `Console` with different `policy` cross-chain
`SafeDeployer.sol` - [`deployConsole()`](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L56-L71)

The address of a deployed `Console` won't change if it's deployed with a different `policyCommitment`. Anyone can mimic an original `Console` to deploy with the same parameters on different chains even with a different `policyCommitment`. Original users might start to use the `Safe` to soon discover they have an undesirable `policy` action executed by the `Trusted Validator`.

**Recommendation**: Consider having an additional parameter for the `policyCommitment` in `_createSafe()` that gets used in `_genNonce()` to generate a unique `nonce` that differs based on different policies.


### [L-04] - `SafeDeployer` - Unsafe threshold is allowed
`SafeDeployer.sol` - [`_setupSubAccount()`](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L196), [`_setupConsoleAccount()`](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L151)

`Safe`s deployed with a threshold of one can be taken over by a single signer. In case one of the signers turns malicious, or just one private key gets compromised -> The signer can renounce other owners of the `Safe` and transfer all funds out.

**Recommendation**: Warn users against this attack in documentation and comments in code. A stricter solution would be to enforce a minimum threshold of two in `deployConsoleAccount()` and `deploySubAccount()`.


### [L-05] - Broken invariants: Main Console Account removals
`TransactionValidator.sol` - [`_isConsoleBeingOverriden()`](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L149-L174)

Two of the main in variants stated by the project are:
>  Main Console Account should always be able to remove  `SafeModeratorOverridable`  without validation from  `PolicyValidator`

> Main Console Account should always be able to remove  `ConsoleFallbackHandler`  without validation from `PolicyValidator`

In `_isConsoleBeingOverriden()` the hash of the data `keccak256(_data)`  is checked if:

```solidity
keccak256(_data) == keccak256(abi.encodeCall(IGnosisSafe.setGuard, address(0))) ||
keccak256(_data) == keccak256(abi.encodeCall(IGnosisSafe.setFallbackHandler, (address(0)))
```
The invariants does not hold true if a `Console` account:
- removes the `SafeModeratorOverridable` or `ConsoleFallbackHandler` by changing the `guard` or `fallbackHandler` to a different `guard` or `fallbackHandler`
- removes the `guard` and `fallbackHandler` by setting to the dead address `address(0xdead)`
- batches multiple transactions with `GnosisMultisend` - (e.g. setting both guard and fallback to `address(0`) in one transaction or other actions after setting either to `address(0)`)

When this happens `_isConsoleBeingOverriden()` will return `false` and `PolicyValidator` has to validate the transaction. 

**Recommendation**: Document these edge cases to warn operators. Consider whether the added complexity is worth it to solve this. Alternatively be more specific with the main invariants: "Main Console Account should always be able to set `address(0)` to remove `SafeModeratorOverridable` / `ConsoleFallbackHandler` in a single call".

---


## Non-critical Issues

### [NC-01] - `SubAccount` can be deployed to same address with different `policyCommitment` cross-chain
`SafeDeployer.sol` - [`deploySubAccount()`](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L82-L103)

The deployer of a `SubAccount` on one chain can deploy to the same address with same parameters in other chains, but with a different `policyCommitment`. Consider to have an additional `policyCommitment` param while generating an address in `_createSafe()` -> `genNonce()` to generate a unique nonce to different policies.


### [NC-02] -  Any address can register as `wallet`  in `WalletRegistry`
`WalletRegistry.sol` - [`registerWallet()`](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L35-L40)

There is no restriction on who can register a `wallet` or besides it's not a `subAccount`. Any EOA can register as a `wallet`. If registering would be restricted to `SafeDeployer`, only the expected `Safe`'s would be registered instead of any `msg.sender`.

### [NC-03] -  There is no way to deregister `subAccount` or `wallet` in `WalletRegistry`
`WalletRegistry.sol` - [`registerWallet()`](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L35-L40), [`registerSubAccount()`](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L49-L55)

Once an account is registered, there is no way to renounce a `wallet` or `subAccount` registration. For example if a `SubAccount` would be unused or deemed dangerous, it can't be de-registered from the `Console` in `WalletRegistry`.


### [NC-04] - `_ensureAddressProvider()` doesn't completely ensure address is address provider
`src/core/AddressProvider.sol` - [`_ensureAddressProvider()`](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L130-L134)
```solidity
    function _ensureAddressProvider(address _newAddress) internal view {
        if (IAddressProviderService(_newAddress).addressProviderTarget() != address(this)) {
            revert AddressProviderUnsupported();
        }
    }
```
Anyone can implement a `addressProviderTarget()` function or var on an arbitrary address that returns the address provider's address. However only governance restricted functions use this helper, thus there is no risk currently.


### [NC-05] -  Caution with `Safe` deployments and `AddressProvider` registry
Be mindful that `Safe` deployment addresses can differ between chains.
For example the version `1.3` `GnosisSafe` contract has deployments on:
|Contract|Chain|Address
|:--:|:--:|:--|
| `GnosisSafe` | Optimism | `0x69f4D1788e39c87893C980c06EdF4b7f686e2938`
| `GnosisSafeL2` | Optimism | `0xfb1bffC9d739B8D520DaF37dF666da4C687191EA`
| `GnosisSafe`  | Polygon |  `0xd9Db270c1B5E3Bd161E8c8503c55cEABeE709552`
| `GnosisSafeL2`  | Polygon | `0x3E5c63644E683549055b9Be8653de26E0B4CD36E`

 `v1.4.1` `Safe` is only deployed on `Ethereum`, `BSC` and `Gnosis`.

Refer to this repository for the up-to-date collection of `Safe` deployments: https://github.com/safe-global/safe-deployments/

---