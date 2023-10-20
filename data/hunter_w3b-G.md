# Gas Optimization For Brahma Protocol

While striving for enhanced code clarity in the provided snippets, certain functions have been abbreviated to emphasize affected sections.

Developers should remain vigilant during the incorporation of these proposed modifications to avert potential vulnerabilities. Despite prior testing of the optimizations, developers bear the responsibility of conducting comprehensive reevaluation.

Conducting code reviews and additional testing is highly recommended to mitigate any plausible hazards that may arise from the refactoring endeavor.

These optimizations aim to reduce the gas costs associated with contract deployment and execution. Below is a summary of the key points and issues discussed:

# Summary

| Number | Issues                                                                                                   | Instances |
| :----: | :------------------------------------------------------------------------------------------------------- | :-------: |
| [G-01] | Using mappings instead of arrays to avoid length checks                                                  |     2     |
| [G-02] | Shorten the array rather than copying to a new one                                                       |     3     |
| [G-03] | Use assembly to validate msg.sender                                                                      |     7     |
| [G-04] | Using a positive conditional flow to save a NOT opcode                                                   |     9     |
| [G-05] | Use selfdestruct in the constructor if the contract is one-time use                                      |           |
| [G-06] | Consider using alternatives to OpenZeppelin                                                              |           |
| [G-07] | Using assembly to revert with an error message                                                           |    40     |
| [G-08] | Use SUB or XOR instead of ISZERO(EQ()) to check for inequality (more efficient in certain scenarios)     |     1     |
| [G-09] | Split revert statements                                                                                  |     5     |
| [G-10] | It is sometimes cheaper to cache calldata                                                                |     3     |
| [G-11] | Expressions for constant values such as a call to keccak256(), should use immutable rather than constant |     1     |
| [G-12] | Cache storage variables: write and read storage variables exactly once                                   |     7     |
| [G-13] | Always use Named Returns                                                                                 |    27     |
| [G-14] | Use bitmaps instead of bools when a significant amount of booleans are used                              |     1     |
| [G-15] | Internal functions not called by the contract should be removed to save deployment Gas                   |     5     |
| [G-16] | Empty blocks should be removed or emit something                                                         |    14     |
| [G-17] | Using Storage instead of memory for structs/arrays saves gas                                             |     2     |
| [G-18] | Use assembly to write address storage values                                                             |     4     |
| [G-19] | Avoid contract existence checks by using low level calls                                                 |    10     |
| [G-20] | Use solidity version 0.8.20 to gain some gas boost                                                       |    15     |
| [G-21] | Use ++i instead of i++ to increment                                                                      |     2     |
| [G-22] | Using fixed bytes is cheaper than using string                                                           |     5     |
| [G-23] | Multiple accesses of a mapping/array should use a local variable cache                                   |     8     |
| [G-24] | abi.encode() is less efficient than abi.encodePacked()                                                   |     4     |
| [G-25] | Use assembly to reuse memory space when making more than one external call.                              |     8     |
| [G-26] | Missing zero address checks in the constructor                                                           |     2     |
| [G-27] | Use hardcode address instead address(this)                                                               |     2     |
| [G-28] | Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4                    |     6     |

### These optimization techniques and recommendations can help developers reduce gas costs and improve the efficiency of the Brahma Protocol contracts. It's essential for developers to exercise caution and conduct thorough testing and code reviews when implementing these optimizations to ensure the security and correctness of their contracts.

## [G-01] Using mappings instead of arrays to avoid length checks

When storing a list or group of items that you wish to organize in a specific order and fetch with a fixed key/index, it’s common practice to use an array data structure. This works well, but did you know that a trick can be implemented to save 2,000+ gas on each read using a mapping?

See the example below

```solidity
/// get(0) gas cost: 4860
contract Array {
    uint256[] a;

    constructor() {
        a.push() = 1;
        a.push() = 2;
        a.push() = 3;
    }

    function get(uint256 index) external view returns (uint256) {
        return a[index];
    }
}

/// get(0) gas cost: 2758
contract Mapping {
    mapping(uint256 => uint256) a;

    constructor() {
        a[0] = 1;
        a[1] = 2;
        a[2] = 3;
    }

    function get(uint256 index) external view returns (uint256) {
        return a[index];
    }
}

```

Just by using a mapping, we get a gas saving of 2102 gas. Why? Under the hood when you read the value of an index of an array, solidity adds bytecode that checks that you are reading from a valid index (i.e an index strictly less than the length of the array) else it reverts with a panic error (Panic(0x32) to be precise). This prevents from reading unallocated or worse, allocated storage/memory locations.

Due to the way mappings are (simply a key => value pair), no check like that exists and we are able to read from the a storage slot directly. It’s important to note that when using mappings in this manner, your code should ensure that you are not reading an out of bound index of your canonical array.

```solidity
File: core/registries/WalletRegistry.sol

25    mapping(address wallet => address[] subAccountList) public walletToSubAccountList;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L25

```solidity
File: src/core/SafeDeployer.sol

116        Types.Executable[] memory txns;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L116

## [G-02] Shorten the array rather than copying to a new one

Inline-assembly can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array

```solidity
File: core/SafeDeployer.sol

119            txns = new Types.Executable[](2);

132            txns = new Types.Executable[](1);

174        Types.Executable[] memory txns = new Types.Executable[](2);
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L119

## [G-03] Use assembly to validate msg.sender

If you want to save gas by optimizing the validation of msg.sender using inline assembly in Solidity, you can create a custom validation routine. This can be helpful when you have specific requirements that cannot be efficiently met using built-in Solidity constructs.

For-Example

```solidity
pragma solidity ^0.8.0;

contract AddressValidator {
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    function validateSender() public view returns (bool) {
        address sender;

        assembly {
            // Load the current value of msg.sender
            sender := sload(0)
        }

        return sender == owner;
    }
}
```

In this example, we have a simple contract AddressValidator with an owner address. The validateSender function uses inline assembly to load the msg.sender value from storage and compares it to the contract's owner. If they match, the function returns true; otherwise, it returns false.

Please note that while this approach can save gas by reducing the gas cost of the validation, it adds complexity to your contract and should be used sparingly. In most cases, the built-in msg.sender provided by Solidity is efficient and secure, and manually optimizing it using inline assembly might not be necessary or worth the added complexity.

```solidity
File: core/AddressProvider.sol

63        if (msg.sender != pendingGovernance) {

140        if (msg.sender != governance) revert NotGovernance(msg.sender);

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L63

```solidity
File: core/registries/PolicyRegistry.sol

52        } else if (msg.sender == account && walletRegistry.isWallet(account)) {

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L52C1-L53C1

```solidity
File: core/registries/ExecutorRegistry.sol

55        if (_walletRegistry.subAccountToWallet(_subAccount) != msg.sender) revert NotOwnerWallet();

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol#L55

```solidity
File: core/registries/WalletRegistry.sol

36        if (isWallet[msg.sender]) revert AlreadyRegistered();

50        if (msg.sender != AddressProviderService._getAuthorizedAddress(_SAFE_DEPLOYER_HASH)) revert InvalidSender();


```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L36

```solidity
File: core/AddressProviderService.sol

63        if (msg.sender != addressProvider.governance()) {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol#L63

## [G-04] Using a positive conditional flow to save a NOT opcode

This is the same example we gave at the beginning of the article. In the code snippet below, the second function avoids an unnecessary negation. In theory, the extra ! increases the computational cost. But as we noted at the top of the article, you should benchmark both methods because the compiler is can sometimes optimize this.

For-Example

```solidity

function cond() public {
    if (!condition) {
        action1();
    }
    else {
        action2();
    }
}

function cond() public {
    if (condition) {
        action2();
    }
    else {
        action1();
    }
}
```

```solidity
File: src/libraries/SafeHelper.sol

77        if (!success) revert SafeExecTransactionFailed();
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L77

```solidity
File: core/TransactionValidator.sol

197        if (!IGnosisSafe(_subAccount).isModuleEnabled(ownerConsole)) revert InvalidModule();
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L197

```solidity
File: core/AddressProvider.sol

82        if (!_overrideCheck) {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L82

```solidity
File: core/registries/ExecutorRegistry.sol

40        if (!_walletRegistry.isOwner(msg.sender, _subAccount)) revert NotOwnerWallet();

42        if (!subAccountToExecutors[_subAccount].add(_executor)) revert AlreadyExists();

57        if (!subAccountToExecutors[_subAccount].remove(_executor)) revert DoesNotExist();
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol#L57C1-L58C1

```solidity
File: src/core/SafeDeployer.sol

92        if (!_walletRegistry.isWallet(msg.sender)) revert NotWallet();

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L92

```solidity
File: src/core/ExecutorPlugin.sol

96        if (!success) revert ModuleExecutionFailed();

112        if (!executorRegistry.isExecutor(execRequest.account, execRequest.executor)) {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L96C19-L96C19

## [G-05] Use selfdestruct in the constructor if the contract is one-time use

Sometimes, contracts are used to deploy several contracts in one transaction, which necessitates doing it in the constructor.

If the contract’s only use is the code in the constructor, then selfdestructing at the end of the operation will save gas.

Although selfdestruct is set for removal in an upcoming hardfork, it will still be supported in the constructor per [EIP 6780](https://eips.ethereum.org/EIPS/eip-6780)

## [G-06] Consider using alternatives to OpenZeppelin

OpenZeppelin is a great and popular smart contract library, but there are other alternatives that are worth considering. These alternatives offer better gas efficiency and have been tested and recommended by developers.

Two examples of such alternatives are [Solmate](https://github.com/transmissions11/solmate) and [Solady](https://github.com/Vectorized/solady).

Solmate is a library that provides a number of gas-efficient implementations of common smart contract patterns. Solady is another gas-efficient library that places a strong emphasis on using assembly.

## [G-07] Using assembly to revert with an error message

When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error message. This can in most cases be further optimized by using assembly to revert with the error message.

Here’s an example;

```solidity
/// calling restrictedAction(2) with a non-owner address: 24042
contract SolidityRevert {
address owner;
uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num)  external {
        require(owner == msg.sender, "caller is not owner");
        specialNumber = num;
    }

}

/// calling restrictedAction(2) with a non-owner address: 23734
contract AssemblyRevert {
address owner;
uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num)  external {
        assembly {
            if sub(caller(), sload(owner.slot)) {
                mstore(0x00, 0x20) // store offset to where length of revert message is stored
                mstore(0x20, 0x13) // store length (19)
                mstore(0x40, 0x63616c6c6572206973206e6f74206f776e657200000000000000000000000000) // store hex representation of message
                revert(0x00, 0x60) // revert with data
            }
        }
        specialNumber = num;
    }

}
```

From the example above we can see that we get a gas saving of over 300 gas when reverting wth the same error message with assembly against doing so in solidity. This gas savings come from the memory expansion costs and extra type checks the solidity compiler does under the hood.

```solidity
File: core/SafeEnabler.sol


48        require(module != address(0) && module != _SENTINEL_MODULES, "GS101");

52        require(modules[module] == address(0), "GS102");
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L48

```solidity
File: src/libraries/SafeHelper.sol

77        if (!success) revert SafeExecTransactionFailed();

105        if (len == 0) revert InvalidMultiSendInput();

114                revert InvalidMultiSendCall(i);

169            revert UnableToParseOperation();

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L77

```solidity
File: core/ConsoleFallbackHandler.sol

44            require(safe.signedMessages(messageHash) != 0, "Hash not approved");

51            require(policyValidator.isPolicySignatureValid(msg.sender, messageHash, _signature), "Policy not approved");

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L44

```solidity
File: core/TransactionValidator.sol

186        if (guard != AddressProviderService._getAuthorizedAddress(_SAFE_MODERATOR_HASH)) revert InvalidGuard();

190            revert InvalidFallbackHandler();

197        if (!IGnosisSafe(_subAccount).isModuleEnabled(ownerConsole)) revert InvalidModule();

223            revert TxnUnAuthorized();

243            revert TxnUnAuthorized();

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L190

```solidity
File: core/AddressProvider.sol

64            revert NotPendingGovernance(msg.sender);

101        if (registries[_key] != address(0)) revert RegistryAlreadyExists();

132            revert AddressProviderUnsupported();

140        if (msg.sender != governance) revert NotGovernance(msg.sender);

148        if (addr == address(0)) revert NullAddress();

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L64

```solidity
File: core/PolicyValidator.sol

109            revert NoPolicyCommit();

117            revert TxnExpired(expiryEpoch);

137            revert InvalidSignature();

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L109

```solidity
File: core/registries/ExecutorRegistry.sol

40        if (!_walletRegistry.isOwner(msg.sender, _subAccount)) revert NotOwnerWallet();

42        if (!subAccountToExecutors[_subAccount].add(_executor)) revert AlreadyExists();

55        if (_walletRegistry.subAccountToWallet(_subAccount) != msg.sender) revert NotOwnerWallet();

57        if (!subAccountToExecutors[_subAccount].remove(_executor)) revert DoesNotExist();

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol#L40

```solidity
File: core/registries/WalletRegistry.sol

36        if (isWallet[msg.sender]) revert AlreadyRegistered();

37        if (subAccountToWallet[msg.sender] != address(0)) revert IsSubAccount();

50        if (msg.sender != AddressProviderService._getAuthorizedAddress(_SAFE_DEPLOYER_HASH)) revert InvalidSender();

51        if (subAccountToWallet[_subAccount] != address(0)) revert AlreadyRegistered();

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L36

```solidity
File: src/core/AddressProviderService.sol

28        if (_addressProvider == address(0)) revert InvalidAddressProvider();

64            revert NotGovernance(msg.sender);

73        if (_addr == address(0)) revert InvalidAddress();
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol#L28

```solidity
File: src/core/SafeDeployer.sol

88        if (_policyCommit == bytes32(0)) revert InvalidCommitment();

92        if (!_walletRegistry.isWallet(msg.sender)) revert NotWallet();

237                    revert SafeProxyCreationFailed();

242                revert SafeProxyCreationFailed();

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L88

```solidity
File: src/core/ExecutorPlugin.sol

96        if (!success) revert ModuleExecutionFailed();

113            revert InvalidExecutor();

119            revert InvalidSignature();

144            revert InvalidExecutor();

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L96

## [G-08] Use SUB or XOR instead of ISZERO(EQ()) to check for inequality (more efficient in certain scenarios)

When using inline assembly to compare the equality of two values (e.g if owner is the same as caller()), It is sometimes more efficient to do

```solidity

if sub(caller, sload(owner.slot)) {
    revert(0x00, 0x00)
}

//over doing this

if eq(caller, sload(owner.slot)) {
    revert(0x00, 0x00)
}

```

xor can accomplish the same thing, but be aware that xor will consider a value with all the bits flipped to be equal also, so make sure that this cannot become an attack vector.

This trick will depend on the compiler version used and the context of the code.

```solidity
FIle: core/ConsoleFallbackHandler.sol

159            if iszero(mload(0x00)) { revert(add(response, 0x20), mload(response)) }

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L159

## [G-09] Split revert statements

Similar to splitting require statements, you will usually save some gas by not having a boolean operator in the if statement.

```solidity
contract CustomErrorBoolLessEfficient {
    error BadValue();

    function requireGood(uint256 x) external pure {
        if (x < 10 || x > 20) {
            revert BadValue();
        }
    }
}

contract CustomErrorBoolEfficient {
    error TooLow();
    error TooHigh();

    function requireGood(uint256 x) external pure {
        if (x < 10) {
            revert TooLow();
        }
        if (x > 20) {
            revert TooHigh();
        }
    }
}

```

```solidity
File: src/core/PolicyValidator.sol

135        if (trustedValidator.code.length == 0 && validatorSignature.length == 0) {
136            // TrustedValidator is an EOA and no trustedValidator signature is provided
137            revert InvalidSignature();

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L135C1-L138C10

```solidity
File: core/registries/PolicyRegistry.sol

46            currentCommit == bytes32(0)
47                && msg.sender == AddressProviderService._getAuthorizedAddress(_SAFE_DEPLOYER_HASH)


52        } else if (msg.sender == account && walletRegistry.isWallet(account)) {

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L45C9-L47C99

```solidity
File: /src/core/ExecutorPlugin.sol

117        if (execRequest.executor.code.length == 0 && execRequest.executorSignature.length == 0) {
118            // Executor is an EOA and no executor signature is provided
119            revert InvalidSignature();
        }
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L117C1-L120C10

## [G-10] It is sometimes cheaper to cache calldata

Although the calldataload instruction is a cheap opcode, the solidity compiler will sometimes output cheaper code if you cache calldataload. This will not always be the case, so you should test both possibilities.

```solidity
contract LoopSum {
    function sumArr(uint256[] calldata arr) public pure returns (uint256 sum) {
        uint256 len = arr.length;
        for (uint256 i = 0; i < len; ) {
            sum += arr[i];
            unchecked {
                ++i;
            }
        }
    }
}

```

```solidity
File: contracts/src/core/SafeDeployer.sol

56    function deployConsoleAccount(address[] calldata _owners, uint256 _threshold, bytes32 _policyCommit, bytes32 _salt)

82    function deploySubAccount(address[] calldata _owners, uint256 _threshold, bytes32 _policyCommit, bytes32 _salt)

219    function _createSafe(address[] calldata _owners, bytes memory _initializer, bytes32 _salt)
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L56

## [G-11] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

```solidity
File: src/core/ConsoleFallbackHandler.sol

24    bytes4 internal constant SIMULATE_SELECTOR = bytes4(keccak256("simulate(address,bytes)"));
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L24

## [G-12] Cache storage variables: write and read storage variables exactly once

You will see the following pattern frequently in efficient solidity code. Reading from a storage variable costs at least 100 gas as Solidity does not cache the storage read. Writes are considerably more expensive. Therefore, you should manually cache the variable to do exactly one storage read and exactly one storage write.

For-Example:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

contract Counter1 {
    uint256 public number;

    function increment() public {
        require(number < 10);
        number = number + 1;
    }
}

contract Counter2 {
    uint256 public number;

    function increment() public {
        uint256 _number = number;
        require(_number < 10);
        number = _number + 1;
    }
}
```

The first function reads counter twice, the second code reads it once.

```solidity
File: src/core/AddressProvider.sol

// @audit   governance  is a state varibale
66        emit GovernanceTransferred(governance, msg.sender);
67        governance = msg.sender;


// @audit pendingGovernance  is stv  used in same function
63        if (msg.sender != pendingGovernance) {
68        delete pendingGovernance;

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L66

```solidity
File: src/core/SafeEnabler.sol

// @audit _SENTINEL_MODULES is a state varibale
48        require(module != address(0) && module != _SENTINEL_MODULES, "GS101");

53        modules[module] = modules[_SENTINEL_MODULES];

54        modules[_SENTINEL_MODULES] = module;

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L48

## [G-13] Always use Named Returns

The solidity compiler outputs more efficient code when the variable is declared in the return statement. There seem to be very few exceptions to this in practice, so if you see an anonymous return, you should test it with a named return instead to determine which case is most efficient.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

contract NamedReturn {
    function myFunc1(uint256 x, uint256 y) external pure returns (uint256) {
        require(x > 0);
        require(y > 0);

        return x * y;
    }
}

contract NamedReturn2 {
    function myFunc2(uint256 x, uint256 y) external pure returns (uint256 z) {
        require(x > 0);
        require(y > 0);

        z = x * y;
    }
}

```

```solidity
File: src/libraries/TypeHashHelper.sol

65        return keccak256(

85        return keccak256(

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/TypeHashHelper.sol#L65

```solidity
File: src/libraries/SafeHelper.sol

94        return signatures;

144        return address(uint160(uint256(bytes32(guardAddress))));

154        return address(uint160(uint256(bytes32(fallbackHandlerAddress))));
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L94

```solidity
File: src/core/TransactionValidator.sol

173        return false;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L173

```solidity
File: src/core/ConsoleFallbackHandler.sol

61        return getMessageHashForSafe(GnosisSafe(payable(msg.sender)), message);

70        return keccak256(abi.encodePacked(bytes1(0x19), bytes1(0x01), safe.domainSeparator(), safeMessageHash));

86        return (value == EIP1271_MAGIC_VALUE) ? UPDATED_MAGIC_VALUE : bytes4(0);

95        return array;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L61

```solidity
File: src/core/AddressProvider.sol

113        return authorizedAddresses[_key];

122        return registries[_key];
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L113

```solidity
File: src/core/PolicyValidator.sol

79        return isPolicySignatureValid(account, transactionStructHash, signatures);

141        return SignatureCheckerLib.isValidSignatureNow(trustedValidator, txnValidityDigest, validatorSignature);

175        return (_NAME, _VERSION);
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L79

```solidity
File: src/core/registries/ExecutorRegistry.sol

68        return subAccountToExecutors[_subAccount].contains(_executor);

76        return subAccountToExecutors[_subAccount].values();
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol#L68

```solidity
File: src/core/registries/WalletRegistry.sol

64        return walletToSubAccountList[_wallet];

74        return subAccountToWallet[_subAccount] == _wallet;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L64

```solidity
File: src/core/AddressProviderService.sol

36        return address(addressProvider);
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol#L36

```solidity
File: src/core/SafeDeployer.sol

147        return abi.encodeCall(

192        return abi.encodeCall(

254        return uint256(keccak256(abi.encodePacked(_ownersHash, ownerSafeCount[_ownersHash]++, _salt, VERSION)));
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L147

```solidity
File: src/core/ExecutorPlugin.sol

76        return txnResult;

97        return txnResult;

160        return (_NAME, _VERSION);
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L76

## [G-14] Use bitmaps instead of bools when a significant amount of booleans are used

A common pattern, especially in airdrops, is to mark an address as “already used” when claiming the airdrop or NFT mint.

However, since it only takes one bit to store this information, and each slot is 256 bits, that means one can store a 256 flags/booleans with one storage slot.

```solidity
File: src/core/registries/WalletRegistry.sol

27    mapping(address => bool) public isWallet;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L27

## [G-15] Internal functions not called by the contract should be removed to save deployment Gas

```solidity
File: src/core/PolicyValidator.sol

174    function _domainNameAndVersion() internal pure override returns (string memory name, string memory version) {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L174

```solidity
File: src/core/AddressProviderService.sol

44    function _getRegistry(bytes32 _key) internal view returns (address registry) {

54    function _getAuthorizedAddress(bytes32 _key) internal view returns (address authorizedAddress) {

62    function _onlyGov() internal view {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol#L44C1-L45C1

```solidity
File: src/core/ExecutorPlugin.sol

159    function _domainNameAndVersion() internal pure override returns (string memory name, string memory version) {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L159C1-L160C1

## [G-16] Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.
If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation.

If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be
nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when
the code is later modified (if( x ) {}else if(y){ . . . }else{ . . . } => if ( !x) {if(y) { . . . }else{ . . .}}) .
Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas.

```solidity
File: src/core/TransactionValidator.sol

54    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}


81    function validatePostTransactionOverridable(bytes32, /*txHash */ bool, /*success */ address /*console */ )
82        external
83        view
84    {}
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L54

```solidity
File: src/core/SafeModeratorOverridable.sol

23    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}



86    function checkModuleTransaction(
87        address, /* to */
88        uint256, /* value */
89        bytes memory, /* data */
90        Enum.Operation, /* operation */
91        address /* module */
92    ) external override {}
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol#L23C1-L24C1

```solidity
File: src/core/SafeModerator.sol


17    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}



80    function checkModuleTransaction(
81        address, /* to */
82        uint256, /* value */
83        bytes memory, /* data */
84        Enum.Operation, /* operation */
85        address /* module */
86    ) external override {}
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModerator.sol#L17

```solidity
File: src/core/ConsoleFallbackHandler.sol

29    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L29

```solidity
File: src/core/PolicyValidator.sol

30    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L30

```solidity
File: core/registries/PolicyRegistry.sol

24    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L24

```solidity
File: core/registries/ExecutorRegistry.sol

29    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol#L29C1-L30C1

```solidity
File: core/registries/WalletRegistry.sol


29    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L29

```solidity
File: src/core/SafeDeployer.sol

42    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L142

```solidity
File: src/core/ExecutorPlugin.sol

60    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L60

## [G-17] Using Storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from
storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array.

If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read.

Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to
be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read.

The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```solidity
File: src/core/SafeDeployer.sol

116        Types.Executable[] memory txns;

174        Types.Executable[] memory txns = new Types.Executable[](2);
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L116C1

## [G-18] Use assembly to write address storage values

For-Example

```solidity

    function setEntryPoint(IEntryPoint _entryPoint) public onlyOwner {
           entryPoint = _entryPoint;
       }


    function setEntryPoint(IEntryPoint _entryPoint) public onlyOwner {
        assembly {
            sstore(entryPoint.slot, _entryPoint)
        }
    }
```

```solidity
File: src/core/SafeEnabler.sol

33    address internal immutable _self;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L33

```solidity
File: src/core/AddressProvider.sol

27    address public governance;

29    address public pendingGovernance;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L27

```solidity
File: src/core/AddressProviderService.sol

25    AddressProvider public immutable addressProvider;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol#L25C1-L26C1

## [G-19] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

```solidity
File: src/core/AddressProviderService.sol

45        registry = addressProvider.getRegistry(_key);

55        authorizedAddress = addressProvider.getAuthorizedAddress(_key);
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol#L55

```solidity
File: src/core/ConsoleFallbackHandler.sol

85        bytes4 value = validator.isValidSignature(abi.encode(_dataHash), _signature);
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L85C1-L86C1

```solidity
File: src/core/ExecutorPlugin.sol

73        TransactionValidator(AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH))

148        TransactionValidator(AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH))
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L73C1-L74C1

```solidity
File: src/core/PolicyValidator.sol

63        uint256 nonce = IGnosisSafe(account).nonce();

107            PolicyRegistry(AddressProviderService._getRegistry(_POLICY_REGISTRY_HASH)).commitments(account);
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L63

```solidity
File: src/libraries/SafeHelper.sol

143        bytes memory guardAddress = IGnosisSafe(safe).getStorageAt(_GUARD_STORAGE_SLOT, 1);

153        bytes memory fallbackHandlerAddress = IGnosisSafe(safe).getStorageAt(_FALLBACK_HANDLER_STORAGE_SLOT, 1);
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L143C1-L144C1

```solidity
File: src/core/TransactionValidator.sol

194            WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH)).subAccountToWallet(_subAccount);
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L194

## [G-20] Use solidity version 0.8.20 to gain some gas boost

Upgrade to the latest solidity version 0.8.20 to get additional gas savings. See latest release for reference:

https://blog.soliditylang.org/2023/05/10/solidity-0.8.20-release-announcement

```solidity
File: src/libraries/TypeHashHelper.sol

5   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/TypeHashHelper.sol#L5

```solidity
File: src/libraries/SafeHelper.sol

5   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L5

```solidity
File: src/core/TransactionValidator.sol

5   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L5

```solidity
File: src/core/SafeModeratorOverridable.sol

5   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol#L5

```solidity
File: src/core/SafeEnabler.sol

5   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L5

```solidity
File: src/core/SafeModerator.sol

5   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModerator.sol#L5

```solidity
File: src/core/Constants.sol

5   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/Constants.sol#L5

```solidity
File: src/core/ConsoleFallbackHandler.sol

5   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L5

```solidity
File: src/core/AddressProvider.sol

5   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L5

```solidity
File: src/core/PolicyValidator.sol
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L5

```solidity
File: src/core/registries/PolicyRegistry.sol

5   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L5

```solidity
File: src/core/registries/ExecutorRegistry.sol

5   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol#L5

```solidity
File: src/core/registries/WalletRegistry.sol

5   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L5

```solidity
File: src/core/AddressProviderService.sol

5   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol#L5

```solidity
File: src/core/SafeDeployer.sol

5   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L5

```solidity
File: src/core/ExecutorPlugin.sol

5   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L5

## [G-21] Use ++i instead of i++ to increment

The reason behind this is in way ++i and i++ are evaluated by the compiler.

i++ returns i(its old value) before incrementing i to a new value. This means that 2 values are stored on the stack for usage whether you wish to use it or not. ++i on the other hand, evaluates the ++ operation on i (i.e it increments i) then returns i (its incremented value) which means that only one item needs to be stored on the stack

```solidity
File: src/core/SafeDeployer.sol

254        return uint256(keccak256(abi.encodePacked(_ownersHash, ownerSafeCount[_ownersHash]++, _salt, VERSION)));
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L254

```solidity
File: src/core/ExecutorPlugin.sol

131                nonce: executorNonce[execRequest.account][execRequest.executor]++
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L131

## [G-22] Using fixed bytes is cheaper than using string

Bytes constants are more efficient than string constants

```solidity
File: src/core/PolicyValidator.sol

26    string private constant _NAME = "PolicyValidator";

28    string private constant _VERSION = "1.0";
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L26

```solidity
File: src/core/SafeDeployer.sol

23    string public constant VERSION = "1";
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L23

```solidity
File: src/core/ExecutorPlugin.sol

53    string private constant _NAME = "ExecutorPlugin";

55    string private constant _VERSION = "1.0";
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L53

## [G-23] Multiple accesses of a mapping/array should use a local variable cache

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping’s value in a local storage or calldata variable when the value is accessed [multiple times](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0), saves ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Caching an array’s struct avoids recalculating the array offsets into memory/calldata.

```solidity
File: src/core/AddressProvider.sol

101        if (registries[_key] != address(0)) revert RegistryAlreadyExists();
102        registries[_key] = _registry;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L101C1-L102C38

```solidity
File: src/core/registries/PolicyRegistry.sol

67        emit UpdatedPolicyCommit(account, policyCommit, commitments[account]);
68        commitments[account] = policyCommit;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L67C1-L68C45

```solidity
File: src/core/registries/WalletRegistry.sol

36        if (isWallet[msg.sender]) revert AlreadyRegistered();

38        isWallet[msg.sender] = true;


51        if (subAccountToWallet[_subAccount] != address(0)) revert AlreadyRegistered();
52        subAccountToWallet[_subAccount] = _wallet;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L51C1-L52C51

## [G-24] abi.encode() is less efficient than abi.encodePacked()

```solidity
File: src/libraries/TypeHashHelper.sol

66            abi.encode(

86            abi.encode(
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/TypeHashHelper.sol#L86

```solidity
File: src/core/ConsoleFallbackHandler.sol

69        bytes32 safeMessageHash = keccak256(abi.encode(SAFE_MSG_TYPEHASH, keccak256(message)));
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L69C1-L70C1

```solidity
File: src/core/SafeDeployer.sol

225        bytes32 ownersHash = keccak256(abi.encode(_owners));
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L225C1-L226C1

## [G-25] Use assembly to reuse memory space when making more than one external call.

An operation that causes the solidity compiler to expand memory is making external calls. When making external calls the compiler has to encode the function signature of the function it wishes to call on the external contract alongside it’s arguments in memory. As we know, solidity does not clear or reuse memory memory so it’ll have to store these data in the next free memory pointer which expands memory further.

With inline assembly, we can either use the scratch space and free memory pointer offset to store this data (as above) if the function arguments do not take up more than 96 bytes in memory. Better still, if we are making more than one external call we can reuse the same memory space as the first calls to store the new arguments in memory without expanding memory unnecessarily. Solidity in this scenario would expand memory by as much as the returned data length is. This is because the returned data is stored in memory (in most cases). If the return data is less than 96 bytes, we can use the scratch space to store it to prevent expanding memory.

See the example below;

```solidity
contract Called {
    function add(uint256 a, uint256 b) external pure returns(uint256) {
        return a + b;
    }
}


contract Solidity {
    // cost: 7262
    function call(address calledAddress) external pure returns(uint256) {
        Called called = Called(calledAddress);
        uint256 res1 = called.add(1, 2);
        uint256 res2 = called.add(3, 4);

        uint256 res = res1 + res2;
        return res;
    }
}


contract Assembly {
    // cost: 5281
    function call(address calledAddress) external view returns(uint256) {
        assembly {
            // check that calledAddress has code deployed to it
            if iszero(extcodesize(calledAddress)) {
                revert(0x00, 0x00)
            }

            // first call
            mstore(0x00, hex"771602f7")
            mstore(0x04, 0x01)
            mstore(0x24, 0x02)
            let success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res1 := mload(0x60)

            // second call
            mstore(0x04, 0x03)
            mstore(0x24, 0x4)
            success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res2 := mload(0x60)

            // add results
            let res := add(res1, res2)

            // return data
            mstore(0x60, res)
            return(0x60, 0x20)
        }
    }
}

```

We save approximately 2,000 gas by using the scratch space to store the function selector and it’s arguments and also reusing the same memory space for the second call while storing the returned data in the zero slot thus not expanding memory.

If the arguments of the external function you wish to call is above 64 bytes and if you are making one external call, it wouldn’t save any significant gas writing it in assembly. However, if making more than one call. You can still save gas by reusing the same memory slot for the 2 calls using inline assembly.

Note: Always remember to update the free memory pointer if the offset it points to is already used, to avoid solidity overriding the data stored there or using the value stored there in an unexpected way.

Also note to avoid overwriting the zero slot (0x60 memory offset) if you have undefined dynamic memory values within that call stack. An alternative is to explicitly define dynamic memory values or if used, to set the slot back to 0x00 before exiting the assembly block.

```solidity
File: src/core/ConsoleFallbackHandler.sol

44            require(safe.signedMessages(messageHash) != 0, "Hash not approved");

51            require(policyValidator.isPolicySignatureValid(msg.sender, messageHash, _signature), "Policy not approved");

52            safe.checkSignatures(messageHash, _data, _signature);
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L44

```solidity
File: src/core/registries/PolicyRegistry.sol

50        } else if (walletRegistry.isOwner(msg.sender, account)) {

52        } else if (msg.sender == account && walletRegistry.isWallet(account)) {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L50

```solidity
File: src/core/SafeDeployer.sol

92        if (!_walletRegistry.isWallet(msg.sender)) revert NotWallet();

98        _walletRegistry.registerSubAccount(msg.sender, _subAcc);

101        PolicyRegistry(AddressProviderService._getRegistry(_POLICY_REGISTRY_HASH)).updatePolicy(_subAcc, _policyCommit);

```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L92

## [G-26] Missing zero address checks in the constructor

Missing checks for zero-addresses may lead to infunctional protocol, if the variable addresses are updated incorrectly. It also wast gas as it requires the redeployment of the contract.

```solidity
File: src/core/SafeEnabler.sol

33        _self = address(this);
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L33

```solidity
File: src/core/AddressProvider.sol

45        governance = _governance;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L45

## [G-27] Use hardcode address instead address(this)

```solidity
File: src/libraries/SafeHelper.sol

74            _generateSingleThresholdSignature(address(this)) // signatures
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L74

```solidity
File: src/core/AddressProvider.sol

131        if (IAddressProviderService(_newAddress).addressProviderTarget() != address(this)) {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L131C1-L132C1

## [G-28] Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4

In Solidity version 0.8.4 and later, it's recommended to use bytes.concat() instead of abi.encodePacked() when you want to concatenate multiple byte arrays or strings. The bytes.concat() function is a safer and more explicit way to concatenate data.

```solidity
File: src/libraries/SafeHelper.sol

88        bytes memory signatures = abi.encodePacked(

119            bytes memory encodedTxn = abi.encodePacked(

125                packedTxns = abi.encodePacked(packedTxns, encodedTxn);
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L88

```solidity
File: src/core/ConsoleFallbackHandler.sol

70        return keccak256(abi.encodePacked(bytes1(0x19), bytes1(0x01), safe.domainSeparator(), safeMessageHash));
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L70

```solidity
File: src/core/SafeDeployer.sol

144            data: abi.encodePacked(WalletRegistry.registerWallet.selector)

254        return uint256(keccak256(abi.encodePacked(_ownersHash, ownerSafeCount[_ownersHash]++, _salt, VERSION)));
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L144
