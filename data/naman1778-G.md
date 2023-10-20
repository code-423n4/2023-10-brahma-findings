## [G-01] Stack variable used as a cheaper cache for a state variable is only used once

If the variable is only accessed once, it’s cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend.

There are 3 instances of this issue in 3 files:

```
File: src/libraries/SafeHelper.sol	

117: uint256 calldataLength = _txns[i].data.length;
```

    diff --git a/contracts/src/libraries/SafeHelper.sol b/contracts/src/libraries/SafeHelper.sol
    index 7830a80..c19f9f0 100644
    --- a/contracts/src/libraries/SafeHelper.sol
    +++ b/contracts/src/libraries/SafeHelper.sol
    @@ -114,10 +114,8 @@ library SafeHelper {
                     revert InvalidMultiSendCall(i);
                 }

    -            uint256 calldataLength = _txns[i].data.length;
    -
                 bytes memory encodedTxn = abi.encodePacked(
    -                bytes1(call), bytes20(_txns[i].target), bytes32(_txns[i].value), bytes32(calldataLength), _txns[i].data
    +                bytes1(call), bytes20(_txns[i].target), bytes32(_txns[i].value), bytes32(_txns[i].data.length), _txns[i].data
                 );

                 if (i != 0) {

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/libraries/SafeHelper.sol

```
File: src/core/SafeEnabler.sol	

69: bytes32 slot = _GUARD_STORAGE_SLOT;
```

    diff --git a/contracts/src/core/SafeEnabler.sol b/contracts/src/core/SafeEnabler.sol
    index 02137cc..53e41ea 100644
    --- a/contracts/src/core/SafeEnabler.sol
    +++ b/contracts/src/core/SafeEnabler.sol
    @@ -66,10 +66,9 @@ contract SafeEnabler is GnosisSafeStorage {
         function setGuard(address guard) public {
             _onlyDelegateCall();

    -        bytes32 slot = _GUARD_STORAGE_SLOT;
             // solhint-disable-next-line no-inline-assembly
             assembly {
    -            sstore(slot, guard)
    +            sstore(_GUARD_STORAGE_SLOT, guard)
             }
             emit ChangedGuard(guard);
         }

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol

```
File: src/core/registries/PolicyRegistry.sol	

42: bytes32 currentCommit = commitments[account];
```

    diff --git a/contracts/src/core/registries/PolicyRegistry.sol b/contracts/src/core/registries/PolicyRegistry.sol
    index 559d5ca..2655bff 100644
    --- a/contracts/src/core/registries/PolicyRegistry.sol
    +++ b/contracts/src/core/registries/PolicyRegistry.sol
    @@ -39,11 +39,9 @@ contract PolicyRegistry is AddressProviderService {

             WalletRegistry walletRegistry = WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH));

    -        bytes32 currentCommit = commitments[account];
    -
             // solhint-disable no-empty-blocks
             if (
    -            currentCommit == bytes32(0)
    +            commitments[account] == bytes32(0)
                     && msg.sender == AddressProviderService._getAuthorizedAddress(_SAFE_DEPLOYER_HASH)
             ) {
                 // In case invoker is safe  deployer

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }

    contract Contract0 {
        uint256 num = 5;
        uint256 sum;
        function not_optimized() public returns(bool){
            uint256 num1 = num;
            sum = num1 + 2;
        }
    }

    contract Contract1 {
        uint256 num = 5;
        uint256 sum;
        function optimized() public returns(bool){
            sum = num + 2;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 63799                                     | 244             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| not_optimized                             | 24462           | 24462 | 24462  | 24462 | 1       |


| Contract1 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 63599                                     | 243             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| optimized                                 | 24460           | 24460 | 24460  | 24460 | 1       |

## [G-02] Revert Transaction as soon as possible

Always try reverting transactions as early as possible when using require statements . In case a transaction revert occurs, the user will pay the gas up until the revert was executed

There are 2 instances of this issue in 1 file:

```
File: src/core/registries/ExecutorRegistry.sol	

38: function registerExecutor(address _subAccount, address _executor) external {
39:     WalletRegistry _walletRegistry = WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH));
40:     if (!_walletRegistry.isOwner(msg.sender, _subAccount)) revert NotOwnerWallet();
41: 
42:     if (!subAccountToExecutors[_subAccount].add(_executor)) revert AlreadyExists();

53: function deRegisterExecutor(address _subAccount, address _executor) external {
54:     WalletRegistry _walletRegistry = WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH));
55:     if (_walletRegistry.subAccountToWallet(_subAccount) != msg.sender) revert NotOwnerWallet();
56: 
57:     if (!subAccountToExecutors[_subAccount].remove(_executor)) revert DoesNotExist();
```

    diff --git a/contracts/src/core/registries/ExecutorRegistry.sol b/contracts/src/core/registries/ExecutorRegistry.sol
    index b563a69..dc86153 100644
    --- a/contracts/src/core/registries/ExecutorRegistry.sol
    +++ b/contracts/src/core/registries/ExecutorRegistry.sol
    @@ -36,10 +36,11 @@ contract ExecutorRegistry is AddressProviderService {
          * @param _executor executor to add
          */
         function registerExecutor(address _subAccount, address _executor) external {
    +        if (!subAccountToExecutors[_subAccount].add(_executor)) revert AlreadyExists();
    +
             WalletRegistry _walletRegistry = WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH));
             if (!_walletRegistry.isOwner(msg.sender, _subAccount)) revert NotOwnerWallet();

    -        if (!subAccountToExecutors[_subAccount].add(_executor)) revert AlreadyExists();
             emit RegisterExecutor(_subAccount, msg.sender, _executor);
         }

    @@ -51,10 +52,11 @@ contract ExecutorRegistry is AddressProviderService {
          * @param _executor executor to remove
          */
         function deRegisterExecutor(address _subAccount, address _executor) external {
    +        if (!subAccountToExecutors[_subAccount].remove(_executor)) revert DoesNotExist();
    +
             WalletRegistry _walletRegistry = WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH));
             if (_walletRegistry.subAccountToWallet(_subAccount) != msg.sender) revert NotOwnerWallet();

    -        if (!subAccountToExecutors[_subAccount].remove(_executor)) revert DoesNotExist();
             emit DeRegisterExecutor(_subAccount, msg.sender, _executor);
         }

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol