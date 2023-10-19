# Report


## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Use assembly to check for `address(0)` | 14 |
| [GAS-2](#GAS-2) | Using bools for storage incurs overhead | 1 |
| [GAS-3](#GAS-3) | Use calldata instead of memory for function arguments that do not get mutated | 15 |
| [GAS-4](#GAS-4) | Use Custom Errors | 4 |
| [GAS-5](#GAS-5) | Don't initialize variables with default value | 2 |
| [GAS-6](#GAS-6) | Functions guaranteed to revert when called by normal users can be marked `payable` | 3 |
| [GAS-7](#GAS-7) | `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) | 2 |
| [GAS-8](#GAS-8) | Using `private` rather than `public` for constants, saves gas | 4 |
| [GAS-9](#GAS-9) | `internal` functions not called by the contract should be removed | 10 |

### [GAS-1] Use assembly to check for `address(0)`
*Saves 6 gas per instance*

*Instances (14)*:
```solidity
File: core/AddressProvider.sol

101:         if (registries[_key] != address(0)) revert RegistryAlreadyExists();

148:         if (addr == address(0)) revert NullAddress();

```

```solidity
File: core/AddressProviderService.sol

28:         if (_addressProvider == address(0)) revert InvalidAddressProvider();

73:         if (_addr == address(0)) revert InvalidAddress();

```

```solidity
File: core/PolicyValidator.sol

108:         if (policyHash == bytes32(0)) {

```

```solidity
File: core/SafeDeployer.sol

61:         bool _policyHashValid = _policyCommit != bytes32(0);

88:         if (_policyCommit == bytes32(0)) revert InvalidCommitment();

244:         } while (_safe == address(0));

```

```solidity
File: core/SafeEnabler.sol

48:         require(module != address(0) && module != _SENTINEL_MODULES, "GS101");

52:         require(modules[module] == address(0), "GS102");

```

```solidity
File: core/registries/PolicyRegistry.sol

36:         if (policyCommit == bytes32(0)) {

46:             currentCommit == bytes32(0)

```

```solidity
File: core/registries/WalletRegistry.sol

37:         if (subAccountToWallet[msg.sender] != address(0)) revert IsSubAccount();

51:         if (subAccountToWallet[_subAccount] != address(0)) revert AlreadyRegistered();

```

### [GAS-2] Using bools for storage incurs overhead
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

*Instances (1)*:
```solidity
File: core/registries/WalletRegistry.sol

27:     mapping(address => bool) public isWallet;

```

### [GAS-3] Use calldata instead of memory for function arguments that do not get mutated
Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.

*Instances (14)*:
```solidity
File: core/ConsoleFallbackHandler.sol

39:     function isValidSignature(bytes memory _data, bytes memory _signature) public view override returns (bytes4) {

39:     function isValidSignature(bytes memory _data, bytes memory _signature) public view override returns (bytes4) {

60:     function getMessageHash(bytes memory message) public view returns (bytes32) {

68:     function getMessageHashForSafe(GnosisSafe safe, bytes memory message) public view returns (bytes32) {

```

```solidity
File: core/PolicyValidator.sol

58:         bytes memory data,

```

```solidity
File: core/SafeModerator.sol

36:         bytes memory data,

43:         bytes memory signatures,

83:         bytes memory, /* data */

```

```solidity
File: core/SafeModeratorOverridable.sol

42:         bytes memory data,

49:         bytes memory signatures,

89:         bytes memory, /* data */

```

```solidity
File: core/TransactionValidator.sol

65:         // Check if guard or fallback handler is being removed, if yes, skip policy validation

97:             txParams.from, txParams.to, txParams.value, txParams.data, txParams.operation, txParams.signatures

129:      * @notice Provides on-chain guarantees on security critical expected states of subAccount for executor plugin

```

### [GAS-4] Use Custom Errors
[Source](https://blog.soliditylang.org/2021/04/21/custom-errors/)
Instead of using error strings, to reduce deployment and runtime cost, you should use Custom Errors. This would save both deployment and runtime cost.

*Instances (4)*:
```solidity
File: core/ConsoleFallbackHandler.sol

44:             require(safe.signedMessages(messageHash) != 0, "Hash not approved");

51:             require(policyValidator.isPolicySignatureValid(msg.sender, messageHash, _signature), "Policy not approved");

```

```solidity
File: core/SafeEnabler.sol

48:         require(module != address(0) && module != _SENTINEL_MODULES, "GS101");

52:         require(modules[module] == address(0), "GS102");

```

### [GAS-5] Don't initialize variables with default value

*Instances (2)*:
```solidity
File: core/SafeModeratorOverridable.sol

21:     uint8 public constant DIFFER_SAFE_MOD = 0;

```

```solidity
File: libraries/SafeHelper.sol

107:         uint256 i = 0;

```

### [GAS-6] Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

*Instances (3)*:
```solidity
File: core/AddressProvider.sol

139:     function _onlyGov() internal view {

```

```solidity
File: core/AddressProviderService.sol

62:     function _onlyGov() internal view {

```

```solidity
File: core/SafeEnabler.sol

81:     function _onlyDelegateCall() private view {

```

### [GAS-7] `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too)
*Saves 5 gas per loop*

*Instances (2)*:
```solidity
File: core/ExecutorPlugin.sol

131:                 nonce: executorNonce[execRequest.account][execRequest.executor]++

```

```solidity
File: core/SafeDeployer.sol

254:         return uint256(keccak256(abi.encodePacked(_ownersHash, ownerSafeCount[_ownersHash]++, _salt, VERSION)));

```

### [GAS-8] Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*Instances (4)*:
```solidity
File: core/SafeDeployer.sol

23:     string public constant VERSION = "1";

```

```solidity
File: core/SafeModeratorOverridable.sol

21:     uint8 public constant DIFFER_SAFE_MOD = 0;

```

```solidity
File: libraries/TypeHashHelper.sol

49:     bytes32 public constant TRANSACTION_PARAMS_TYPEHASH =

56:     bytes32 public constant VALIDATION_PARAMS_TYPEHASH =

```

### [GAS-9] `internal` functions not called by the contract should be removed
If the functions are required by an interface, the contract should inherit from that interface and use the `override` keyword

*Instances (10)*:
```solidity
File: core/AddressProviderService.sol

44:     function _getRegistry(bytes32 _key) internal view returns (address registry) {

54:     function _getAuthorizedAddress(bytes32 _key) internal view returns (address authorizedAddress) {

62:     function _onlyGov() internal view {

```

```solidity
File: libraries/SafeHelper.sol

63:     function _executeOnSafe(address safe, address target, Enum.Operation op, bytes memory data) internal {

103:     function _packMultisendTxns(Types.Executable[] memory _txns) internal pure returns (bytes memory packedTxns) {

142:     function _getGuard(address safe) internal view returns (address) {

152:     function _getFallbackHandler(address safe) internal view returns (address) {

163:     function _parseOperationEnum(Types.CallType callType) internal pure returns (Enum.Operation operation) {

```

```solidity
File: libraries/TypeHashHelper.sol

64:     function _buildTransactionStructHash(Transaction memory txn) internal pure returns (bytes32) {

84:     function _buildValidationStructHash(Validation memory validation) internal pure returns (bytes32) {

```


