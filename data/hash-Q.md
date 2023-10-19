## Summary

### Low Risk Issues


|ID|Title|Instances|
|-|:-|:-:|
| [L-01](#l-01-different-nonce-used-among-safe-owners-and-validator)| Different nonce used among safe owners and validator | 1 |


Total: 1 instances over 1 issues

### Non-critical Issues


|ID|Title|Instances|
|-|:-|:-:|
| [NC-01](#nc-01-update-comments-to-improvise-the-grammar)| Update comments to improvise the grammar | 1 |
| [NC-02](#nc-02-update-comment-to-provide-clearer-meaning)| Update comment to provide clearer meaning | 4 |
| [NC-03](#nc-03-remove-extra-space)| Remove extra space | 1 |
| [NC-04](#nc-04-commet-provided-is-wrong)| Commet provided is wrong | 3 |
| [NC-05](#nc-05-natspec-order)| Natspec order | 1 |
| [NC-06](#nc-06-typos)| Typos | 3 |
| [NC-07](#nc-07-unneeded-comments)| Unneeded comments | 1 |


Total: 14 instances over 7 issues

## Low Risk Issues

### [L-01] Different nonce used among safe owners and validator
Since the `Safe` nonce gets incremented in the `executeTransaction()` function, the validator signature is checked against the next nonce while the safe owners sign for the current nonce.

*There are 1 instances of this issue*

```solidity
File: src/core/PolicyValidator.sol

54:            function isPolicySignatureValid(
55:                address account,
56:                address to,
57:                uint256 value,
58:                bytes memory data,
59:                Enum.Operation operation,
60:                bytes calldata signatures
61:            ) external view returns (bool) {
62:                // Get nonce from safe
63:    =>          uint256 nonce = IGnosisSafe(account).nonce(); // @audit nonce is incremented in safe prior to this 

```

*GitHub*: [54](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L54-#L63)

## Non-critical Issues

### [NC-01] Update comments to improvise the grammar

*There are 1 instances of this issue*

```diff
File: src/core/registries/ExecutorRegistry.sol

+++               use exist instead of exists
33:             * @dev Adds new executor if it doesn't already exists else reverts with AlreadyExists()

```

*GitHub*: [33](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol#L33-#L33)


### [NC-02] Update comment to provide clearer meaning

*There are 4 instances of this issue*

```diff
File: src/core/TransactionValidator.sol

+++                   the guard or fallback is being removed
163:                 * In case these conditions are met, the guard is being removed, return true

```

*GitHub*: [163](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L163-#L163)
```diff
File: src/core/PolicyValidator.sol

+++               using validator would be better than backend since it is mentioned so everywhere else
153:             * @return expiryEpoch extracted expiry epoch signed by brahma backend

```

*GitHub*: [153](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L153-#L153)
```diff
File: src/core/registries/ExecutorRegistry.sol

+++               subAccount instead of subAcc
35:             * @param _subAccount subAcc address to add executor to

+++               subAccount instead of subAcc
50:             * @param _subAccount subAcc address remove executor from // mns-clearer-comment

```

*GitHub*: [35](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol#L35-#L35), [50](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol#L50-#L50)


### [NC-03] Remove extra space

*There are 1 instances of this issue*

```solidity
File: src/core/registries/PolicyRegistry.sol

49:                    // In case invoker is safe  deployer

```

*GitHub*: [49](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L49-#L49)


### [NC-04] Commet provided is wrong

*There are 3 instances of this issue*

```diff
File: src/core/ConsoleFallbackHandler.sol

+++             returns the magic value which is bytes4 and not bool
37:             * @return a bool upon valid or invalid signature with corresponding _data
38:             */
39:            function isValidSignature(bytes memory _data, bytes memory _signature) public view override returns (bytes4) {
40:                // Caller should be a Safe
41:                GnosisSafe safe = GnosisSafe(payable(msg.sender));
42:                ... more code ...
54:                return EIP1271_MAGIC_VALUE;
55:            }


+++             checkSignatures is a view method. Typo of save instead of safe. The return type is bytes4 and not bool
75:             * @dev Should return whether the signature provided is valid for the provided data.
76:             *       The save does not implement the interface since `checkSignatures` is not a view method.
77:             *       The method will not perform any state changes (see parameters of `checkSignatures`)
78:             * @param _dataHash Hash of the data signed on the behalf of address(msg.sender)
79:             * @param _signature Signature byte array associated with _dataHash
80:             * @return a bool upon valid or invalid signature with corresponding _dataHash
81:             * @notice See https://github.com/gnosis/util-contracts/blob/bb5fe5fb5df6d8400998094fb1b32a178a47c3a1/contracts/StorageAccessible.sol

```

*GitHub*: [37](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L37-#L55), [75](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L75-#L81)
```diff
File: src/core/PolicyValidator.sol

+++             transactionStructHash could be from either execution plugin or from safe's executeTransaction() function 
96:             * @param transactionStructHash execution digest from ExecutorPlugin

```

*GitHub*: [96](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L96-#L96)


### [NC-05] Natspec order
Follow the same convention of `param` followed by `return`

*There are 1 instances of this issue*

```solidity
File: src/core/registries/ExecutorRegistry.sol

71:            /**
72:             * @return all the executors for a subAccount
73:             * @param _subAccount address of subAccount

```

*GitHub*: [71](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol#L71-#L73)


### [NC-06] Typos
*There are 3 instances of this issue*

```diff
File: src/core/TransactionValidator.sol

+++              Brhma => Brahma
78:             * @notice Provides on-chain guarantees on security critical expected states of a Brhma console account

```

*GitHub*: [78](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L78-#L78)
```diff
File: src/core/SafeModeratorOverridable.sol

+++          Brhma => Brahma
14:         * @notice A guard that validates transactions and allows only policy abiding txns, on Brhma console account and can be overriden by removal of guard

```

*GitHub*: [14](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol#L14-#L14)
```diff
File: src/core/SafeDeployer.sol

+++          Brhma => Brahma
176:                // Enable Brhma Console account as module on sub Account

```

*GitHub*: [176](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L176-#L176)


### [NC-07] Unneeded comments
*There are 1 instances of this issue*

```solidity
File: src/core/SafeDeployer.sol

234:                        // KEK

```

*GitHub*: [234](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L234-#L234)