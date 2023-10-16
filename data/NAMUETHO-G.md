# c4udit Report

## Files analyzed
- contracts\src\core\AddressProvider.sol
- contracts\src\core\AddressProviderService.sol
- contracts\src\core\ConsoleFallbackHandler.sol
- contracts\src\core\ConsoleOpBuilder.sol
- contracts\src\core\Constants.sol
- contracts\src\core\ExecutorPlugin.sol
- contracts\src\core\PolicyValidator.sol
- contracts\src\core\SafeDeployer.sol
- contracts\src\core\SafeEnabler.sol
- contracts\src\core\SafeModerator.sol
- contracts\src\core\SafeModeratorOverridable.sol
- contracts\src\core\TransactionValidator.sol
- contracts\src\core\registries\ExecutorRegistry.sol
- contracts\src\core\registries\PolicyRegistry.sol
- contracts\src\core\registries\WalletRegistry.sol

## Issues found

### Cache Array Length Outside of Loop

#### Impact
Issue Information: [G002](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g002---cache-array-length-outside-of-loop)

#### Findings:
```
contracts\src\core\AddressProvider.sol::84 => if (_authorizedAddress.code.length != 0) _ensureAddressProvider(_authorizedAddress);
contracts\src\core\ConsoleFallbackHandler.sol::35 => * @param _data Arbitrary length data signed on the behalf of address(msg.sender)
contracts\src\core\ConsoleFallbackHandler.sol::43 => if (_signature.length == 0) {
contracts\src\core\ConsoleOpBuilder.sol::100 => uint256 _numberOfExecutors = executors.length;
contracts\src\core\ConsoleOpBuilder.sol::158 => uint256 _numberOfExecutors = _executors.length;
contracts\src\core\ExecutorPlugin.sol::37 => *  executorSignature = executor's signatures (arbitrary bytes length)
contracts\src\core\ExecutorPlugin.sol::38 => *  validatorSignature = abi.encodePacked(policySignature, length, expiryEpoch)
contracts\src\core\ExecutorPlugin.sol::40 => *      policySignature = validity signature signed by validator (arbitrary bytes length)
contracts\src\core\ExecutorPlugin.sol::41 => *      length = length of `policySignature` (4 bytes)
contracts\src\core\ExecutorPlugin.sol::117 => if (execRequest.executor.code.length == 0 && execRequest.executorSignature.length == 0) {
contracts\src\core\PolicyValidator.sol::41 => *  safeSignature = safe owners signatures (arbitrary bytes length)
contracts\src\core\PolicyValidator.sol::42 => *  validatorSignature = EIP 712 digest signature (arbitrary bytes length)
contracts\src\core\PolicyValidator.sol::43 => *  validatorSignatureLength = length of `validatorSignature` (4 bytes)
contracts\src\core\PolicyValidator.sol::85 => *      validatorSignature = EIP 712 digest signed by `TRUSTED_VALIDATOR`(arbitrary bytes length)
contracts\src\core\PolicyValidator.sol::86 => *      validatorSignatureLength = length of `validatorSignature` (4 bytes)
contracts\src\core\PolicyValidator.sol::135 => if (trustedValidator.code.length == 0 && validatorSignature.length == 0) {
contracts\src\core\PolicyValidator.sol::147 => *  safeSignature = safe owners signatures (arbitrary bytes length)
contracts\src\core\PolicyValidator.sol::148 => *  validatorSignature = EIP 712 digest signed (arbitrary bytes length)
contracts\src\core\PolicyValidator.sol::149 => *  validatorSignatureLength = length of `validatorSignature` (4 bytes)
contracts\src\core\PolicyValidator.sol::161 => uint256 length = _signatures.length;
contracts\src\core\PolicyValidator.sol::162 => if (length < 8) revert InvalidSignatures();
contracts\src\core\PolicyValidator.sol::164 => uint32 sigLength = uint32(bytes4(_signatures[length - 8:length - 4]));
contracts\src\core\PolicyValidator.sol::165 => expiryEpoch = uint32(bytes4(_signatures[length - 4:length]));
contracts\src\core\PolicyValidator.sol::166 => validatorSignature = _signatures[length - 8 - sigLength:length - 8];
```

### Use immutable for OpenZeppelin AccessControl's Roles Declarations

#### Impact
Issue Information: [G006](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g006---use-immutable-for-openzeppelin-accesscontrols-roles-declarations)

#### Findings:
```
contracts\src\core\AddressProvider.sol::32 => * @notice keccak256 hash of authorizedAddress keys mapped to their addresses
contracts\src\core\AddressProvider.sol::38 => * @notice keccak256 hash of registry keys mapped to their addresses
contracts\src\core\AddressProviderService.sol::41 => * @param _key keccak256 key corresponding to registry
contracts\src\core\AddressProviderService.sol::51 => * @param _key keccak256 key corresponding to authorized address
contracts\src\core\ConsoleFallbackHandler.sol::21 => //keccak256("SafeMessage(bytes message)");
contracts\src\core\ConsoleFallbackHandler.sol::24 => bytes4 internal constant SIMULATE_SELECTOR = bytes4(keccak256("simulate(address,bytes)"));
contracts\src\core\ConsoleFallbackHandler.sol::69 => bytes32 safeMessageHash = keccak256(abi.encode(SAFE_MSG_TYPEHASH, keccak256(message)));
contracts\src\core\ConsoleFallbackHandler.sol::70 => return keccak256(abi.encodePacked(bytes1(0x19), bytes1(0x01), safe.domainSeparator(), safeMessageHash));
contracts\src\core\Constants.sol::17 => /// @notice keccak256("ExecutorRegistry")
contracts\src\core\Constants.sol::21 => /// @notice keccak256("WalletRegistry")
contracts\src\core\Constants.sol::24 => /// @notice keccak256("PolicyRegistry")
contracts\src\core\Constants.sol::30 => /// @notice keccak256("ExecutorPlugin")
contracts\src\core\Constants.sol::33 => /// @notice keccak256("ConsoleFallbackHandler")
contracts\src\core\Constants.sol::37 => /// @notice keccak256("GnosisFallbackHandler")
contracts\src\core\Constants.sol::41 => /// @notice keccak256("GnosisMultiSend")
contracts\src\core\Constants.sol::45 => /// @notice keccak256("GnosisProxyFactory")
contracts\src\core\Constants.sol::49 => /// @notice keccak256("GnosisSafeSingleton")
contracts\src\core\Constants.sol::53 => /// @notice keccak256("PolicyValidator")
contracts\src\core\Constants.sol::57 => /// @notice keccak256("SafeDeployer")
contracts\src\core\Constants.sol::60 => /// @notice keccak256("SafeEnabler")
contracts\src\core\Constants.sol::63 => /// @notice keccak256("SafeModerator")
contracts\src\core\Constants.sol::66 => /// @notice keccak256("SafeModeratorOverridable")
contracts\src\core\Constants.sol::70 => /// @notice keccak256("TransactionValidator")
contracts\src\core\Constants.sol::74 => /// @notice keccak256("ConsoleOpBuilder")
contracts\src\core\Constants.sol::81 => /// @notice keccak256("Guardian")
contracts\src\core\Constants.sol::84 => /// @notice keccak256("TrustedValidator")
contracts\src\core\SafeDeployer.sol::27 => * @dev keccak256("Create2 call failed");
contracts\src\core\SafeDeployer.sol::225 => bytes32 ownersHash = keccak256(abi.encode(_owners));
contracts\src\core\SafeDeployer.sol::235 => if (keccak256(bytes(reason)) != _SAFE_CREATION_FAILURE_REASON) {
contracts\src\core\SafeDeployer.sol::254 => return uint256(keccak256(abi.encodePacked(_ownersHash, ownerSafeCount[_ownersHash]++, _salt, VERSION)));
contracts\src\core\SafeEnabler.sol::29 => /// @dev keccak256("guard_manager.guard.address")
contracts\src\core\TransactionValidator.sol::166 => if (SafeHelper._GUARD_REMOVAL_CALLDATA_HASH == keccak256(_data)) {
contracts\src\core\TransactionValidator.sol::168 => } else if (SafeHelper._FALLBACK_REMOVAL_CALLDATA_HASH == keccak256(_data)) {
```

### Long Revert Strings

#### Impact
Issue Information: [G007](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g007---long-revert-strings)

#### Findings:
```
contracts\src\core\AddressProvider.sol::7 => import {IAddressProviderService} from "interfaces/IAddressProviderService.sol";
contracts\src\core\AddressProviderService.sol::7 => import {IAddressProviderService} from "../../interfaces/IAddressProviderService.sol";
contracts\src\core\ConsoleFallbackHandler.sol::7 => import "safe-contracts/handler/DefaultCallbackHandler.sol";
contracts\src\core\ConsoleFallbackHandler.sol::8 => import "safe-contracts/interfaces/ISignatureValidator.sol";
contracts\src\core\ConsoleOpBuilder.sol::6 => import {AddressProviderService} from "src/core/AddressProviderService.sol";
contracts\src\core\ConsoleOpBuilder.sol::7 => import {PolicyRegistry} from "src/core/registries/PolicyRegistry.sol";
contracts\src\core\ConsoleOpBuilder.sol::8 => import {ExecutorRegistry} from "src/core/registries/ExecutorRegistry.sol";
contracts\src\core\ConsoleOpBuilder.sol::11 => import {IGnosisMultiSend} from "interfaces/external/IGnosisMultiSend.sol";
contracts\src\core\ConsoleOpBuilder.sol::12 => import {IGnosisSafe} from "interfaces/external/IGnosisSafe.sol";
contracts\src\core\ExecutorPlugin.sol::7 => import {ReentrancyGuard} from "openzeppelin-contracts/security/ReentrancyGuard.sol";
contracts\src\core\ExecutorPlugin.sol::8 => import {SignatureCheckerLib} from "solady/utils/SignatureCheckerLib.sol";
contracts\src\core\ExecutorPlugin.sol::10 => import {IGnosisSafe} from "interfaces/external/IGnosisSafe.sol";
contracts\src\core\ExecutorPlugin.sol::11 => import {AddressProviderService} from "src/core/AddressProviderService.sol";
contracts\src\core\ExecutorPlugin.sol::12 => import {TransactionValidator} from "src/core/TransactionValidator.sol";
contracts\src\core\ExecutorPlugin.sol::13 => import {ExecutorRegistry} from "src/core/registries/ExecutorRegistry.sol";
contracts\src\core\PolicyValidator.sol::7 => import {SignatureCheckerLib} from "solady/utils/SignatureCheckerLib.sol";
contracts\src\core\PolicyValidator.sol::9 => import {AddressProviderService} from "src/core/AddressProviderService.sol";
contracts\src\core\PolicyValidator.sol::10 => import {PolicyRegistry} from "src/core/registries/PolicyRegistry.sol";
contracts\src\core\PolicyValidator.sol::12 => import {IGnosisSafe, Enum} from "interfaces/external/IGnosisSafe.sol";
contracts\src\core\SafeDeployer.sol::7 => import {ReentrancyGuard} from "openzeppelin-contracts/security/ReentrancyGuard.sol";
contracts\src\core\SafeDeployer.sol::8 => import {AddressProviderService} from "../core/AddressProviderService.sol";
contracts\src\core\SafeDeployer.sol::9 => import {WalletRegistry} from "../core/registries/WalletRegistry.sol";
contracts\src\core\SafeDeployer.sol::10 => import {PolicyRegistry} from "../core/registries/PolicyRegistry.sol";
contracts\src\core\SafeDeployer.sol::11 => import {IGnosisProxyFactory} from "../../interfaces/external/IGnosisProxyFactory.sol";
contracts\src\core\SafeDeployer.sol::12 => import {IGnosisSafe} from "../../interfaces/external/IGnosisSafe.sol";
contracts\src\core\SafeDeployer.sol::14 => import {IGnosisMultiSend} from "../../interfaces/external/IGnosisMultiSend.sol";
contracts\src\core\SafeEnabler.sol::7 => import {GnosisSafeStorage} from "safe-contracts/examples/libraries/GnosisSafeStorage.sol";
contracts\src\core\SafeModerator.sol::7 => import {IGuard, Enum} from "interfaces/external/IGnosisSafe.sol";
contracts\src\core\SafeModerator.sol::8 => import {AddressProviderService} from "src/core/AddressProviderService.sol";
contracts\src\core\SafeModerator.sol::9 => import {TransactionValidator} from "src/core/TransactionValidator.sol";
contracts\src\core\SafeModeratorOverridable.sol::7 => import {IGuard, Enum} from "interfaces/external/IGnosisSafe.sol";
contracts\src\core\SafeModeratorOverridable.sol::8 => import {AddressProviderService} from "src/core/AddressProviderService.sol";
contracts\src\core\SafeModeratorOverridable.sol::9 => import {TransactionValidator} from "src/core/TransactionValidator.sol";
contracts\src\core\TransactionValidator.sol::7 => import {IGnosisSafe, Enum} from "interfaces/external/IGnosisSafe.sol";
contracts\src\core\TransactionValidator.sol::10 => import {AddressProviderService} from "src/core/AddressProviderService.sol";
contracts\src\core\TransactionValidator.sol::11 => import {WalletRegistry} from "src/core/registries/WalletRegistry.sol";
contracts\src\core\registries\ExecutorRegistry.sol::9 => import {EnumerableSet} from "openzeppelin-contracts/utils/structs/EnumerableSet.sol";
contracts\src\core\registries\PolicyRegistry.sol::7 => import {AddressProviderService} from "src/core/AddressProviderService.sol";
contracts\src\core\registries\PolicyRegistry.sol::8 => import {WalletRegistry} from "src/core/registries/WalletRegistry.sol";
```

### Use Shift Right/Left instead of Division/Multiplication if possible

#### Impact
Issue Information: [G008](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md/#g008---use-shift-rightleft-instead-of-divisionmultiplication-if-possible)

#### Findings:
```
contracts\src\core\ConsoleFallbackHandler.sol::123 => // 250 bytes of code and 300 gas at runtime over the
```

