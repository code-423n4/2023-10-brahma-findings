|     |     |
| --- | --- |
| \[G-01\] | Use assembly to validate `msg.sender` |
| \[G-02\] | Use hardcode address instead `address(this)` |
| \[G-03\] | uint8 is not always cheaper than uint256 |
| \[G-04\] | Use Assembly To Check For address(0) |
| \[G-05\] | Use bytes32 rather than string/bytes. |
| \[G-06\] | Multiple accesses of a mapping/array should use a storage pointer |
| \[G-07\] | When possible, use assembly instead of `unchecked{}` |
| \[G‑08\] | Using `bool`s for storage incurs overhead |
| \[G-9\] | State variables should be cached in stack variables rather than re-reading them from storage |
| \[G-10\] | Unnecessary declaration of state variable |
| \[G-11\] | Usage of "UINTS", "INTS" smaller than 32 Bytes (256 bits) results in Increased Gas Consumption. |
| \[G-12\] | Empty blocks should be removed or emit something |
| \[G-13\] | The result of a function call should be cached rather than re-calling the function |
| \[G-14\] | Events are not indexed |
| \[G-15\] | abi.encode() is less efficient than abi.encodePacked() |

## \[G-01\] Use assembly to validate `msg.sender`

We can use assembly to efficiently validate msg.sender for the didPay and uniswapV3SwapCallback functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.

```
63:	if (msg.sender != pendingGovernance) {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L63

```
140:     if (msg.sender != governance) revert NotGovernance(msg.sender);
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L140

```
45:	if (
            currentCommit == bytes32(0)
                && msg.sender == AddressProviderService._getAuthorizedAddress(_SAFE_DEPLOYER_HASH)
        ) {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L45

```
52:     } else if (msg.sender == account && walletRegistry.isWallet(account)) {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol#L52

```
55:         if (_walletRegistry.subAccountToWallet(_subAccount) != msg.sender) revert NotOwnerWallet();
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol#L55

```
50:    if (msg.sender != AddressProviderService._getAuthorizedAddress(_SAFE_DEPLOYER_HASH)) revert InvalidSender();
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol#L50

```
63:        if (msg.sender != addressProvider.governance()) {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol#L63

## \[G-02\] Use hardcode address instead `address(this)`

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded `address`. Foundry’s script.sol and solmate’s `LibRlp.sol` contracts can help achieve this.

References: <ins>https://book.getfoundry.sh/reference/forge-std/compute-create-address</ins>

<ins>https://twitter.com/transmissions11/status/1518507047943245824</ins>

```
74:  _generateSingleThresholdSignature(address(this)) // signatures
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L74

```
33:	_self = address(this);
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L33

```
82:	if (address(this) == _self) revert OnlyDelegateCall();
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L82

```
131:	        if (IAddressProviderService(_newAddress).addressProviderTarget() != address(this)) {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L131

## \[G-03\] uint8 is not always cheaper than uint256

The EVM only operates on 32 bytes/ 256 bits at a time. This means that if you use uint8, EVM has to first convert it uint256 to work on it and the conversion costs extra gas! You may wonder, What were the devs thinking? Why did they create smaller variables then? The answer lies in packing. In solidity, you can pack multiple small variables into one slot, but if you are defining a lone variable and can’t pack it, it’s optimal to use a uint256 rather than uint8.

```
24:   uint8 operation;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/TypeHashHelper.sol#L24

```
110:	uint8 call = uint8(Enum.Operation.Call);

112:	call = uint8(Enum.Operation.DelegateCall);
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L110

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L112

```
21:	uint8 public constant DIFFER_SAFE_MOD = 0;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol#L21

## \[G-04\] Use Assembly To Check For address(0)

it's generally more gas-efficient to use assembly to check for a zero address (address(0)) than to use the == operator.

The reason for this is that the == operator generates additional instructions in the EVM bytecode, which can increase the gas cost of your contract. By using assembly, you can perform the zero address check more efficiently and reduce the overall gas cost of your contract.

Here's an example of how you can use assembly to check for a zero address:

```
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

By using assembly to perform the zero address check, we can make our code more gas-efficient and reduce the overall cost of our contract. It's important to note that assembly can be more difficult to read and maintain than Solidity code, so it should be used with caution and only when necessary.

```
52:	require(modules[module] == address(0), "GS102");
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol#L52

```
148:  if (addr == address(0)) revert NullAddress();
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L148

```
28:         if (_addressProvider == address(0)) revert InvalidAddressProvider();
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol#L28

```
73:          if (_addr == address(0)) revert InvalidAddress();
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol#L73

```
244:  } while (_safe == address(0));
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L244

### Mitigation

Using `assembly` to check for the zero address can result in significant gas savings compared to using a Solidity expression; especially if the check is performed frequently or in a loop. However, it’s important to note that using `assembly` can make the code less readable and harder to maintain, so it should be used judiciously and with caution.

## \[G-05\] Use bytes32 rather than string/bytes.

If you can fit your data in 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is much cheaper in solidity. Basically, Any fixed size variable in solidity is cheaper than variable size.

```
26:	  string private constant _NAME = "PolicyValidator";
    /// @notice EIP712 domain version
27:      string private constant _VERSION = "1.0";
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L26C2-L28C46

```
53:  string private constant _NAME = "ExecutorPlugin";
    /// @notice EIP712 domain version
54:       string private constant _VERSION = "1.0";
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L53C3-L55C46

```
174:	 function _domainNameAndVersion() internal pure override returns (string memory name, string memory version) {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol#L174

```
59:     function _domainNameAndVersion() internal pure override returns (string memory name, string memory version) {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol#L159

## \[G-06\] Multiple accesses of a mapping/array should use a storage pointer

Caching a mapping’s value in a storage pointer when the value is accessed multiple times saves ~40 gas per access due to not having to perform the same offset calculation every time. Help the Optimizer by saving a storage variable’s reference instead of repeatedly fetching it.

To achieve this, declare a storage pointer for the variable and use it instead of repeatedly fetching the reference in a map or an array.

```
File:  core/registries/ExecutorRegistry.sol

42: if (!subAccountToExecutors[_subAccount].add(_executor)) revert AlreadyExists();

57:  if (!subAccountToExecutors[_subAccount].remove(_executor)) revert DoesNotExist();

68: return subAccountToExecutors[_subAccount].contains(_executor);

76: return subAccountToExecutors[_subAccount].values();
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol

in the ExecutorRegistry.sol `subAccountToExecutors[_subAccount]` is accessed multiple time.

```
File:  src/core/registries/WalletRegistry.sol

51:  if (subAccountToWallet[_subAccount] != address(0)) revert AlreadyRegistered();

52:  subAccountToWallet[_subAccount] = _wallet;

74: return subAccountToWallet[_subAccount] == _wallet;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol

in the  WalletRegistry.sol `subAccountToWallet[_subAccount]` and `walletToSubAccountList[_wallet]` are accessed multiple time.

## \[G-07\] When possible, use assembly instead of `unchecked{}`

You can also use unchecked{} for even more gas savings but this will not check to see if i overflows. For best gas savings, use inline assembly, however, this limits the functionality you can achieve.

```
131:	unchecked {
                ++i;
            }
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol#L131

## \[G‑08\] Using `bool`s for storage incurs overhead

```
// Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.
```

Use `uint256(1)` and `uint256(2)` for true/false to avoid a Gwarmaccess (**<ins>100 gas</ins>**) for the extra SLOAD, and to avoid Gsset (**20000 gas**) when changing from `false` to `true`, after having been `true` in the past.

```
61:    bool _policyHashValid = _policyCommit != bytes32(0);
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L61

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L155

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModerator.sol#L70

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L77

## \[G-09\] State variables should be cached in stack variables rather than re-reading them from storage

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

```
62: function acceptGovernance() external {
        if (msg.sender != pendingGovernance) {
            revert NotPendingGovernance(msg.sender);
        }
        emit GovernanceTransferred(governance, msg.sender);
        governance = msg.sender;
        delete pendingGovernance;
    }
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol#L63

## \[G-10\] Unnecessary declaration of state variable

`DIFFER_SAFE_MOD ` is declared  but not used anywhere in the contract.

```
uint8 public constant DIFFER_SAFE_MOD = 0;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol#L21

## \[G-11\] Usage of "UINTS", "INTS" smaller than 32 Bytes (256 bits) results in Increased Gas Consumption.

📌 Using Elements that are: Lower than 32 Bytes = Higher Gas Usage BECAUSE "EVM operates on 32 Bytes at a time & if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size"

```
24:	  bytes4 internal constant SIMULATE_SELECTOR = bytes4(keccak256("simulate(address,bytes)"));

27:       bytes4 internal constant UPDATED_MAGIC_VALUE = 0x1626ba7e;
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L24

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L27

```
85:	bytes4 value = validator.isValidSignature(abi.encode(_dataHash), _signature);
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol#L85

## \[G-12\] Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (`if(x){}else if(y){...}else{...}` =\> `if(!x){if(y){...}else{...}}`)

```
81:	  function validatePostTransactionOverridable(bytes32, /*txHash */ bool, /*success */ address /*console */ )
        external
        view
    {}
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol#L81

```
86:  function checkModuleTransaction(
        address, /* to */
        uint256, /* value */
        bytes memory, /* data */
        Enum.Operation, /* operation */
        address /* module */
    ) external override {}
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol#L86

```
80:	   function checkModuleTransaction(
        address, /* to */
        uint256, /* value */
        bytes memory, /* data */
        Enum.Operation, /* operation */
        address /* module */
    ) external override {}
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModerator.sol#L80

## \[G-13\] The result of a function call should be cached rather than re-calling the function

External calls are expensive. Results of external function calls should be cached rather than call them multiple times. Consider caching the following

```
67:  function isExecutor(address _subAccount, address _executor) external view returns (bool) {
        return subAccountToExecutors[_subAccount].contains(_executor);
    }
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol#L67

```
75: function getExecutorsForSubAccount(address _subAccount) external view returns (address[] memory) {
        return subAccountToExecutors[_subAccount].values();
    }
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol#L75

## <ins>\[G-14\] Events are not indexed</ins>

The emitted events are not indexed, making off-chain scripts such as front-ends of dApps difficult to filter the events efficiently.

Recommend adding the `indexed` keyword in each event,

```
event SafeProxyCreationFailure(address indexed singleton, uint256 indexed nonce, bytes initializer);
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol#L32

## \[G-15\] abi.encode() is less efficient than abi.encodePacked()

In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost.

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/TypeHashHelper.sol#L66

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/TypeHashHelper.sol#L86