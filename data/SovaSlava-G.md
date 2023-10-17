## GAS Report

### [G-1] Replace expression with value
We can immediately assign the final value of the expression to the constant, this will reduce the cost of the contract transaction
```diff
- bytes4 internal constant SIMULATE_SELECTOR = bytes4(keccak256("simulate(address,bytes)"));
+ bytes4 internal constant SIMULATE_SELECTOR = 0xbd61951d;
```
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ConsoleFallbackHandler.sol#L24

### [G-2] No need to convert the value to bytes
We can write hex values immediately with a prefix hex""
```diff
- return keccak256(abi.encodePacked(bytes1(0x19), bytes1(0x01), safe.domainSeparator(), safeMessageHash));
+ return keccak256(abi.encodePacked(hex"19", hex"01", safe.domainSeparator(), safeMessageHash));
```
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ConsoleFallbackHandler.sol#L70

### [G-3] Unnecessary zero check
The first 32-byte word of the return data contains the `success` value and we store it at 0x00 address.
We know that call in simulate function will always be a revert (that's what it says in the comments), which means that the value of the first variable from result (bool success) will always be zero. Therefore there is no point in checking whether 0 or not 0.
```diff
function simulate(address targetContract, bytes calldata calldataPayload)
        external
        returns (bytes memory response) {
  ...
  pop(
                call(
                    gas(),
                    // address() has been changed to caller() to use the implementation of the Safe
                    caller(),
                    0,
                    internalCalldata,
                    calldatasize(),
                    // The `simulateAndRevert` call always reverts, and
                    // instead encodes whether or not it was successful in the return
                    // data. The first 32-byte word of the return data contains the
                    // `success` value, so write it to memory address 0x00 (which is
                    // reserved Solidity scratch space and OK to use).
                    0x00,  
                    0x20
                )
            )
	let responseSize := sub(returndatasize(), 0x20)
    response := mload(0x40)
    mstore(0x40, add(response, responseSize))
    returndatacopy(response, 0x20, responseSize)
	
-    if iszero(mload(0x00)) { revert(add(response, 0x20), mload(response)) }
+    revert(add(response, 0x20), mload(response)) 

}
```
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ConsoleFallbackHandler.sol#L159

### [G-4] Need to directly do call to mapping to read the data

The contract WalletRegistry.sol has an isOwner() function that accesses the public mapping of own contract. To save gas, other contracts can directly access mapping
```solidity
contract WalletRegistry is AddressProviderService {

....
mapping(address subAccount => address wallet) public subAccountToWallet;
...
function isOwner(address _wallet, address _subAccount) external view returns (bool) {
        return subAccountToWallet[_subAccount] == _wallet;
}
```
ExecutorRegistry.sol
```diff
 function registerExecutor(address _subAccount, address _executor) external {
        WalletRegistry _walletRegistry = WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH));
       
-        if (!_walletRegistry.isOwner(msg.sender, _subAccount)) revert NotOwnerWallet();
+        if (_walletRegistry.subAccountToWallet(_subAccount) != msg.sender)  revert NotOwnerWallet(); 
        
        if (!subAccountToExecutors[_subAccount].add(_executor)) revert AlreadyExists();
        emit RegisterExecutor(_subAccount, msg.sender, _executor);
    }
```
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/ExecutorRegistry.sol#L40
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/PolicyRegistry.sol#L50


### [G-5] Unnecessary reading of storage variable
For an event emit,  _updatePolicy() function reads the storage value. Although it was already read in function updatePolicy(). We can pass value of 'currentCommit' ,the previously read, as an argument to function _updatePolicy.

```diff
    function updatePolicy(address account, bytes32 policyCommit) external {
        if (policyCommit == bytes32(0)) {
            revert PolicyCommitInvalid();
        }
        WalletRegistry walletRegistry = WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH));
        
        bytes32 currentCommit = commitments[account];

    ...
      
      // solhint-enable no-empty-blocks
-       _updatePolicy(account, policyCommit);
+       _updatePolicy(account, policyCommit, currentCommit);
  }

    /**
     * @notice Internal function to update policy commit for an account
     * @param account address of account to set policy commit for
     * @param policyCommit policy commit hash to set
     */
-    function _updatePolicy(address account, bytes32 policyCommit) internal {
+    function _updatePolicy(address account, bytes32 policyCommit, bytes32 currentCommit) internal {

-       emit UpdatedPolicyCommit(account, policyCommit, commitments[account]);
+       emit UpdatedPolicyCommit(account, policyCommit, currentCommit);
        commitments[account] = policyCommit;
    }
```
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/PolicyRegistry.sol#L58-L67

### [G-6] Assigning a zero value
Operation.call is a zero value - so you don’t have to assign it because the uint8 variable initially has a zero value

```solidity
// contracts/interfaces/external/IGnosisSafe.sol
contract Enum {
    enum Operation {
        Call,           // <---- 0
        DelegateCall    // <---- 1
    }
}
```
```diff
 do {
            // Enum.Operation.Call is 0
            // @audit-think [G] может не нужно присваивать нулевое? посмотреть нашел ил бот
-            uint8 call = uint8(Enum.Operation.Call); 
+            uint8 call;  
            if (_txns[i].callType == Types.CallType.DELEGATECALL) {
                call = uint8(Enum.Operation.DelegateCall);
            } else if (_txns[i].callType == Types.CallType.STATICCALL) {
                revert InvalidMultiSendCall(i);
            }
```

### [G-7] Easier address convert
Can use less conversion to save gas
```diff
   function _getGuard(address safe) internal view returns (address) {
        bytes memory guardAddress = IGnosisSafe(safe).getStorageAt(_GUARD_STORAGE_SLOT, 1);
-        return address(uint160(uint256(bytes32(guardAddress))));
+        return address(uint160(bytes20(guardAddress)));
    }
```

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/libraries/SafeHelper.sol#L144




