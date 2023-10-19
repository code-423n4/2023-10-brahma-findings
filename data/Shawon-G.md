 # Use assembly in place of abi.decode to extract calldata values more efficiently

Instead of using abi.decode, we can use assembly to decode our desired calldata values directly.
This will allow us to avoid decoding calldata values that we will not use.
```
file: contracts/src/core/ConsoleFallbackHandler.sol

    69    bytes32 safeMessageHash = keccak256(abi.encode(SAFE_MSG_TYPEHASH, keccak256(message)));
    85    bytes4 value = validator.isValidSignature(abi.encode(_dataHash), _signature);
```
# Public function that can be made internal 

The function `getMessageHashForSafe` is called as a return value from `getMessageHash` function.
As this function is called internally,it can be set as internal to save gas.

```
file: contracts/src/core/ConsoleFallbackHandler.sol

function getMessageHash(bytes memory message) public view returns (bytes32) {
        return getMessageHashForSafe(GnosisSafe(payable(msg.sender)), message);
    }


function getMessageHashForSafe(GnosisSafe safe, bytes memory message) public view returns (bytes32) {
        bytes32 safeMessageHash = keccak256(abi.encode(SAFE_MSG_TYPEHASH, keccak256(message)));
        return keccak256(abi.encodePacked(bytes1(0x19), bytes1(0x01), safe.domainSeparator(), safeMessageHash));
    }

```

# Optimizing the check order for cost efficient function execution
 the `_onlygov()` check of `setGovernance` function should come first before `_notNull` function call.
 By doing this check first, the `_onlygov` function will check if the function caller is the "gov" of this
contract and revert if that's not the case and in this case,there is no need to check if the value is null or not
hence,it comes second and saves from unnecessary usage of gas.

``` file: core/AddressProvider.sol

   function setGovernance(address _newGovernance) external {
        _notNull(_newGovernance);
        _onlyGov();
        emit GovernanceTransferRequested(governance, _newGovernance);
        pendingGovernance = _newGovernance;
    }
 ```


#  Dead code 
the `_onlyGov` function of `AddressProviderService.sol` has this function 
```  function _onlyGov() internal view {
        if (msg.sender != addressProvider.governance()) {
            revert NotGovernance(msg.sender);
        }
    } 
```
it is meant to check if the function caller is the governance or not.But it was not used in any of the functions of this contract
Consider removing this function or implementing it in the coresponding functions


# Refactor check order of `executeTransaction` function
 
Inside this function,the value of `txnResult` is first set and the `validatePreExecutorTransaction`function which "Validates policy signature" is called
to validate the policy signature.
If we run into an unexpected situation and `validatePreExecutorTransaction` function reverts,the whole `executeTransaction` function would revert
but by that time, `txnResult` would have already taken up memory space.hence,used up alot of gas.
But if we set the `validatePreExecutorTransaction` function first and then set the `txnResult` we can save memory space hence saving up gas in these situations.

```
file: core/ExecutorPlugin.sol
 
 function executeTransaction(ExecutionRequest calldata execRequest) external nonReentrant returns (bytes memory) {
        _validateExecutionRequest(execRequest);

        bytes memory txnResult = _executeTxnAsModule(execRequest.account, execRequest.exec);

        TransactionValidator(AddressProviderService._getAuthorizedAddress(_TRANSACTION_VALIDATOR_HASH))
            .validatePostExecutorTransaction(msg.sender, execRequest.account);

        return txnResult;
    }
```