## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Missing checks for `address(0)` when assigning values to address state variables | 2 |
| [NC-2](#NC-2) | Event is missing `indexed` fields | 4 |
| [NC-3](#NC-3) | Functions not used internally could be marked external | 4 |
### [NC-1] Missing checks for `address(0)` when assigning values to address state variables

*Instances (2)*:
```solidity
File: core/AddressProvider.sol

45:         governance = _governance;

56:         pendingGovernance = _newGovernance;

```

### [NC-2] Event is missing `indexed` fields
Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

*Instances (4)*:
```solidity
File: core/SafeDeployer.sol

32:     event SafeProxyCreationFailure(address indexed singleton, uint256 indexed nonce, bytes initializer);

```

```solidity
File: core/SafeEnabler.sol

19:     event EnabledModule(address module);

20:     event ChangedGuard(address guard);

```

```solidity
File: core/registries/PolicyRegistry.sol

19:     event UpdatedPolicyCommit(address indexed account, bytes32 policyCommit, bytes32 oldPolicyCommit);

```

### [NC-3] Functions not used internally could be marked external

*Instances (4)*:
```solidity
File: core/ConsoleFallbackHandler.sol

39:     function isValidSignature(bytes memory _data, bytes memory _signature) public view override returns (bytes4) {

60:     function getMessageHash(bytes memory message) public view returns (bytes32) {

```

```solidity
File: core/SafeEnabler.sol

43:     function enableModule(address module) public {

66:     function setGuard(address guard) public {

```


## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) |  `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()` | 2 |
| [L-2](#L-2) | Initializers could be front-run | 4 |

### [L-1]  `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()`
Use `abi.encode()` instead which will pad items to 32 bytes, which will [prevent hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode) (e.g. `abi.encodePacked(0x123,0x456)` => `0x123456` => `abi.encodePacked(0x1,0x23456)`, but `abi.encode(0x123,0x456)` => `0x0...1230...456`). "Unless there is a compelling reason, `abi.encode` should be preferred". If there is only one argument to `abi.encodePacked()` it can often be cast to `bytes()` or `bytes32()` [instead](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739).
If all arguments are strings and or bytes, `bytes.concat()` should be used instead

*Instances (2)*:
```solidity
File: core/ConsoleFallbackHandler.sol

70:         return keccak256(abi.encodePacked(bytes1(0x19), bytes1(0x01), safe.domainSeparator(), safeMessageHash));

```

```solidity
File: core/SafeDeployer.sol

254:         return uint256(keccak256(abi.encodePacked(_ownersHash, ownerSafeCount[_ownersHash]++, _salt, VERSION)));

```

### [L-2] Initializers could be front-run
Initializers could be front-run, allowing an attacker to either set their own values, take ownership of the contract, and in the best case forcing a re-deployment

*Instances (4)*:
```solidity
File: core/SafeDeployer.sol

32:     event SafeProxyCreationFailure(address indexed singleton, uint256 indexed nonce, bytes initializer);

219:     function _createSafe(address[] calldata _owners, bytes memory _initializer, bytes32 _salt)

230:             try IGnosisProxyFactory(gnosisProxyFactory).createProxyWithNonce(gnosisSafeSingleton, _initializer, nonce)

239:                 emit SafeProxyCreationFailure(gnosisSafeSingleton, nonce, _initializer);

```

