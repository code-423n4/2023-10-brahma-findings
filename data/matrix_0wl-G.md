## Gas Optimizations

|       | Issue                                                                            |
| ----- | :------------------------------------------------------------------------------- |
| GAS-1 | Functions guaranteed to revert when called by normal users can be marked payable |
| GAS-2 | Usage of `uint`/`int` smaller than 32 bytes (256 bits) incurs overhead           |
| GAS-3 | Use bytes32 instead of string                                                    |

### [GAS-1] Functions guaranteed to revert when called by normal users can be marked payable

#### Description:

If a function modifier such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

#### **Proof Of Concept**

```solidity
File: src/core/AddressProvider.sol

54:         _onlyGov();

78:         _onlyGov();

98:         _onlyGov();

139:     function _onlyGov() internal view {

```

```solidity
File: src/core/AddressProviderService.sol

62:     function _onlyGov() internal view {

```

```solidity
File: src/core/SafeEnabler.sol

44:         _onlyDelegateCall();

67:         _onlyDelegateCall();

81:     function _onlyDelegateCall() private view {

```

### [GAS-2] Usage of `uint`/`int` smaller than 32 bytes (256 bits) incurs overhead

#### Description:

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html

Use a larger size then downcast where needed.

#### **Proof Of Concept**

```solidity
File: src/core/ExecutorPlugin.sol

128:                 operation: uint8(SafeHelper._parseOperationEnum(execRequest.exec.callType)),

```

```solidity
File: src/core/PolicyValidator.sol

22:     error TxnExpired(uint32 expiryEpoch);

71:                 operation: uint8(operation),

113:         (uint32 expiryEpoch, bytes memory validatorSignature) = _decompileSignatures(signatures);

116:         if (expiryEpoch < uint32(block.timestamp)) {

159:         returns (uint32 expiryEpoch, bytes memory validatorSignature)

164:         uint32 sigLength = uint32(bytes4(_signatures[length - 8:length - 4]));

165:         expiryEpoch = uint32(bytes4(_signatures[length - 4:length]));

```

```solidity
File: src/core/SafeModeratorOverridable.sol

21:     uint8 public constant DIFFER_SAFE_MOD = 0;

```

```solidity
File: src/libraries/SafeHelper.sol

110:             uint8 call = uint8(Enum.Operation.Call);

112:                 call = uint8(Enum.Operation.DelegateCall);

```

```solidity
File: src/libraries/TypeHashHelper.sol

24:         uint8 operation;

40:         uint32 expiryEpoch;

```

### [GAS-40] MINE - Unnecessary libraries

#### Description:

Libraries are often only imported for a small number of uses, meaning that they can contain a significant amount of code that is redundant to your contract. If you can safely and effectively implement the functionality imported from a library within your contract, it is optimal to do so.
[Source](https://betterprogramming.pub/how-to-write-smart-contracts-that-optimize-gas-spent-on-ethereum-30b5e9c5db85)

#### **Proof Of Concept**

#### Recommended Mitigation Steps:

Upgrade to at least 0.8.4

```solidity
File: src/core/ConsoleFallbackHandler.sol

7: import "safe-contracts/handler/DefaultCallbackHandler.sol";

8: import "safe-contracts/interfaces/ISignatureValidator.sol";

9: import "safe-contracts/GnosisSafe.sol";

```

### [GAS-3] Use bytes32 instead of string

#### Description:

Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.

#### **Proof Of Concept**

```solidity
File: src/core/ExecutorPlugin.sol

53:     string private constant _NAME = "ExecutorPlugin";

55:     string private constant _VERSION = "1.0";

159:     function _domainNameAndVersion() internal pure override returns (string memory name, string memory version) {

```

```solidity
File: src/core/PolicyValidator.sol

26:     string private constant _NAME = "PolicyValidator";

28:     string private constant _VERSION = "1.0";

174:     function _domainNameAndVersion() internal pure override returns (string memory name, string memory version) {

```

```solidity
File: src/core/SafeDeployer.sol

23:     string public constant VERSION = "1";

233:             } catch Error(string memory reason) {

```
