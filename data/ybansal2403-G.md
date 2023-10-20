## Gas Optimizations

| Number                                                                          | Issue                                                                  | Instances |
| ------------------------------------------------------------------------------- | :--------------------------------------------------------------------- | :-------: |
| [G-01](#avoid-contract-existence-checks-by-using-low-level-calls)               | Avoid contract existence checks by using low level calls               |    29     |
| [G-02](#internal-private-functions-only-called-once-can-be-inlined-to-save-gas) | internal/private functions only called once can be inlined to save gas |     5     |
| [G-03](#use-hardcode-address-instead-address-this)                              | Use hardcode address instead address(this)                             |     2     |
| [G-04](#use-assembly-to-check-for-address-0)                                    | Use Assembly To Check For address(0)                                   |     4     |
| [G-05](#use-modifiers-instead-of-functions-to-save-gas)                         | Use Modifiers Instead of Functions To Save Gas                         |     3     |
| [G-06](#do-not-calculate-constants-variables)                                   | Do not calculate constants variables                                   |     1     |
| [G-07](#use-assembly-for-math-add-sub-mul-div)                                  | Use assembly for math (add, sub, mul, div)                             |     3     |
| [G-08](#nesting-if-statements-is-cheaper-than-using-&&)                         | Nesting if-statements is cheaper than using &&                         |     2     |

<a name='avoid-contract-existence-checks-by-using-low-level-calls'></a>

## [G-01] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

Total Instances: `29`

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L64
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L143C9-L143C92
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L153C8-L153C113

```solidity
File:contracts/src/libraries/SafeHelper.sol
64: bool success = IGnosisSafe(safe).execTransaction(

143:bytes memory guardAddress = IGnosisSafe(safe).getStorageAt(_GUARD_STORAGE_SLOT, 1);

153: bytes memory fallbackHandlerAddress = IGnosisSafe(safe).getStorageAt(_FALLBACK_HANDLER_STORAGE_SLOT, 1);

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L166
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L168
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L182
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L183C8-L183C79
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L194C13-L194C120
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L197
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L219
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L239

```solidity
File:contracts/src/core/TransactionValidator.sol
166:if (SafeHelper._GUARD_REMOVAL_CALLDATA_HASH == keccak256(_data)) {

168:} else if (SafeHelper._FALLBACK_REMOVAL_CALLDATA_HASH == keccak256(_data)) {

182:address guard = SafeHelper._getGuard(_subAccount);

183: address fallbackHandler = SafeHelper._getFallbackHandler(_subAccount);

194:WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH)).subAccountToWallet(_subAccount);

197:if (!IGnosisSafe(_subAccount).isModuleEnabled(ownerConsole)) revert InvalidModule();

219:  !PolicyValidator(AddressProviderService._getAuthorizedAddress(_POLICY_VALIDATOR_HASH)).isPolicySignatureValid(

239: !PolicyValidator(AddressProviderService._getAuthorizedAddress(_POLICY_VALIDATOR_HASH)).isPolicySignatureValid(


```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol#L52
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol#L77

```solidity
File:contracts/src/core/SafeModeratorOverridable.sol
52:  TransactionValidator(AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH))

77: TransactionValidator(AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH))

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L41
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L50
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L61
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L84
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L93C9-L93C59

```solidity
File:contracts/src/core/ConsoleFallbackHandler.sol
41:  GnosisSafe safe = GnosisSafe(payable(msg.sender));

50:PolicyValidator(AddressProviderService._getAuthorizedAddress(_POLICY_VALIDATOR_HASH));

61: return getMessageHashForSafe(GnosisSafe(payable(msg.sender)), message);

84: ISignatureValidator validator = ISignatureValidator(msg.sender);

93:GnosisSafe safe = GnosisSafe(payable(msg.sender));

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L131

```solidity
File:contracts/src/core/AddressProvider.sol
131:if (IAddressProviderService(_newAddress).addressProviderTarget() != address(this)) {

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L63
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L107

```solidity
File:contracts/src/core/PolicyValidator.sol
63:uint256 nonce = IGnosisSafe(account).nonce();

107:PolicyRegistry(AddressProviderService._getRegistry(_POLICY_REGISTRY_HASH)).commitments(account);

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L40C9-L40C116

```solidity
File:contracts/src/core/registries/PolicyRegistry.sol
40:WalletRegistry walletRegistry = WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH));
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol#L39
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol#L54

```solidity
File:contracts/src/core/registries/ExecutorRegistry.sol
39: WalletRegistry _walletRegistry = WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH));

54: WalletRegistry _walletRegistry = WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH));
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L73
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L90
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L109
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L148

```solidity
File:contracts/src/core/ExecutorPlugin.sol
73: TransactionValidator(AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH))

90:  (bool success, bytes memory txnResult) = IGnosisSafe(_account).execTransactionFromModuleReturnData(

109: ExecutorRegistry(AddressProviderService._getRegistry(_EXECUTOR_REGISTRY_HASH));

148: TransactionValidator(AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH))
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L91C8-L91C116

```solidity
File:contracts/src/core/SafeDeployer.sol
91: WalletRegistry _walletRegistry = WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH))
```

<a name='internal-private-functions-only-called-once-can-be-inlined-to-save-gas'></a>

## [G-02] internal/private functions only called once can be inlined to save gas

_The following instance is missed in the automated report_

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

Total Instances: `5`

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L87C5-L87C101

```solidity
File:contracts/src/libraries/SafeHelper.sol
87:function _generateSingleThresholdSignature(address owner) internal pure returns (bytes memory) {

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L66C4-L66C77

```solidity
File:contracts/src/core/registries/PolicyRegistry.sol
66:function _updatePolicy(address account, bytes32 policyCommit) internal {

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L86C5-L86C88
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L106

```solidity
File:contracts/src/core/ExecutorPlugin.sol
86: function _executeTxnAsModule(address _account, Types.Executable memory _executable)

106:function _validateExecutionRequest(ExecutionRequest calldata execRequest) internal {

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L156C5-L160C6

```solidity
File:contracts/src/core/PolicyValidator.sol
156: function _decompileSignatures(bytes calldata _signatures)
157:        internal
158:        pure
159:        returns (uint32 expiryEpoch, bytes memory validatorSignature)
160:    {

```

<a name='use-hardcode-address-instead-address-this'></a>

## [G-03] Use hardcode address instead address(this)

It can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

Here's an example of how you can use a hardcoded address instead of address(this):

```solidity
contract MyContract {
    address public myAddress = 0x1234567890123456789012345678901234567890;

    function doSomething() public {
        // Use myAddress instead of address(this)
        require(msg.sender == myAddress, "Caller is not authorized");

        // Do something
    }
}

```

In the above example, we have a contract MyContract with a public address variable myAddress. Instead of using address(this) to retrieve the contract's address, we have pre-calculated and hardcoded the address in the variable. This can help to reduce the gas cost of our contract and make our code more efficient.

Total Instances: `2`

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L33

```solidity
File:contracts/src/core/SafeEnabler.sol
33:        _self = address(this);

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L131C9-L131C93

```solidity
File:contracts/src/core/AddressProvider.sol
131:if (IAddressProviderService(_newAddress).addressProviderTarget() != address(this)) {

```

<a name='use-assembly-to-check-for-address-0'></a>

## [G-04] Use Assembly To Check For address(0)

it's generally more gas-efficient to use assembly to check for a zero address (address(0)) than to use the == operator.

The reason for this is that the == operator generates additional instructions in the EVM bytecode, which can increase the gas cost of your contract. By using assembly, you can perform the zero address check more efficiently and reduce the overall gas cost of your contract.

Here's an example of how you can use assembly to check for a zero address:

```solidity
contract MyContract {
    function isZeroAddress(address addr) public pure returns (bool) {
        uint256 addrInt = uint256(addr);

        assembly {
            // Load the zero address into memory
            let zero := mload(0x00)

            // Compare the address to the zero address
            let isZero := eq(addrInt, zero)

            // Return the result
            mstore(0x00, isZero)
            return(0, 0x20)
        }
    }
}
```

In the above example, we have a function isZeroAddress that takes an address as input and returns a boolean value indicating whether the address is equal to the zero address. Inside the function, we convert the address to an integer using uint256(addr), and then use assembly to compare the integer to the zero address.

By using assembly to perform the zero address check, we can make our code more gas-efficient and reduce the overall cost of our contract. It's important to note that assembly can be more difficult to read and maintain than Solidity code, so it should be used with caution and only when necessary

Total Instances: `4`

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L48
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L52

```solidity
File:contracts/src/core/SafeEnabler.sol

48:require(module != address(0) && module != _SENTINEL_MODULES, "GS101");

52: require(modules[module] == address(0), "GS102");

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L37
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L51

```solidity
File:contracts/src/core/registries/WalletRegistry.sol

37: if (subAccountToWallet[msg.sender] != address(0)) revert IsSubAccount();

51: if (subAccountToWallet[_subAccount] != address(0)) revert AlreadyRegistered();

```

<a name='use-modifiers-instead-of-functions-to-save-gas'></a>

## [G-05] Use Modifiers Instead of Functions To Save Gas

Example of two contracts with modifiers and internal view function:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
contract Inlined {
    function isNotExpired(bool _true) internal view {
        require(_true == true, "Exchange: EXPIRED");
    }
function foo(bool _test) public returns(uint){
            isNotExpired(_test);
            return 1;
    }
}
// SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
contract Modifier {
modifier isNotExpired(bool _true) {
        require(_true == true, "Exchange: EXPIRED");
        _;
    }
function foo(bool _test) public isNotExpired(_test)returns(uint){
        return 1;
    }
}

```

Differences:

Deploy Modifier.sol 108727 Deploy Inlined.sol 110473 Modifier.foo 21532 Inlined.foo 21556

Total Instances: `3`

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L81C5-L83C6

```solidity
File:contracts/src/core/SafeEnabler.sol
81:function _onlyDelegateCall() private view {
82:        if (address(this) == _self) revert OnlyDelegateCall();
83: }
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L139C5-L141C6
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L147C5-L149C6

```solidity
File:contracts/src/core/AddressProvider.sol
139:function _onlyGov() internal view {
140:        if (msg.sender != governance) revert NotGovernance(msg.sender);
141:    }
.
.
.
.
147:function _notNull(address addr) internal pure {
148:        if (addr == address(0)) revert NullAddress();
149:    }
```

<a name='do-not-calculate-constants-variables'></a>

## [G-06] Do not calculate constants variables

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

Total Instances: `1`

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L24

```solidity
File:contracts/src/core/ConsoleFallbackHandler.sol
24: bytes4 internal constant SIMULATE_SELECTOR = bytes4(keccak256("simulate(address,bytes)"));

```

<a name='use-assembly-for-math-add-sub-mul-div'></a>

## [G-07] Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety. If using Solidity versions < 0.8.0 and you are using Safemath, you can gain significant gas savings by using assembly to calculate values and checking for overflow/underflow.

Addition:

```solidity
//addition in Solidity
function addTest(uint256 a, uint256 b) public pure {
    uint256 c = a + b;
}
```

Gas: 303

```solidity
//addition in assembly
function addAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := add(a, b)
        if lt(c, a) {
            mstore(0x00, "overflow")
            revert(0x00, 0x20)
        }
    }
}

```

Gas: 263

Subtraction

```solidity

//subtraction in Solidity
function subTest(uint256 a, uint256 b) public pure {
  uint256 c = a - b;
}
```

Gas: 300

```solidity
//subtraction in assembly
function subAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := sub(a, b)
        if gt(c, a) {
            mstore(0x00, "underflow")
            revert(0x00, 0x20)
        }
    }
}
```

Gas: 263

Multiplication

```solidity
//multiplication in Solidity
function mulTest(uint256 a, uint256 b) public pure {
    uint256 c = a * b;
}
```

Gas: 325

```solidity
//multiplication in assembly
function mulAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := mul(a, b)
        if lt(c, a) {
            mstore(0x00, "overflow")
            revert(0x00, 0x20)
        }
    }
}
```

Gas: 265

Division

```solidity
//division in Solidity
function divTest(uint256 a, uint256 b) public pure {
    uint256 c = a * b;
}
```

Gas: 325

```solidity
//division in assembly
function divAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := div(a, b)
        if gt(c, a) {
            mstore(0x00, "underflow")
            revert(0x00, 0x20)
        }
    }
}
```

Gas: 265

Total Instances: `3`

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L164
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L165
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L166

```solidity
File:contracts/src/core/PolicyValidator.sol
164:        uint32 sigLength = uint32(bytes4(_signatures[length - 8:length - 4]));

165:        expiryEpoch = uint32(bytes4(_signatures[length - 4:length]));

166:        validatorSignature = _signatures[length - 8 - sigLength:length - 8];

```

<a name='nesting-if-statements-is-cheaper-than-using-&&'></a>

## [G-08] Nesting if-statements is cheaper than using &&

_The following instance is missed in the automated report_
Total Instances: `2`

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L45C9-L48C10
https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L52C9-L52C80

```solidity
File:contracts/src/core/registries/PolicyRegistry.sol
45:if (
46:            currentCommit == bytes32(0)
47:                && msg.sender == AddressProviderService._getAuthorizedAddress(_SAFE_DEPLOYER_HASH)
48:        )
.
.
.
52:} else if (msg.sender == account && walletRegistry.isWallet(account)) {

```
