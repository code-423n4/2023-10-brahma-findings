# 1. Operator whitespace

The assignment operator must have exactly one space on both sides of it and on the same row.

## Proof Of Concept
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ConsoleFallbackHandler.sol#L49-L50
```
PolicyValidator policyValidator =
    PolicyValidator(AddressProviderService._getAuthorizedAddress(_POLICY_VALIDATOR_HASH));
```

## Recommendation

```
PolicyValidator policyValidator = PolicyValidator(AddressProviderService._getAuthorizedAddress(_POLICY_VALIDATOR_HASH));
```

## 2. Variable naming can be confusing

In your code there is a file `Constants.sol`. This file contains constants used by many contracts, and these constants are symbol-based and will be imported externally with a preceding underscore hyphen.

However, in the `ExecutorPlugin.sol` file and more below, the constants in the file itself also have an underscore hyphens. This can be confusing.

## Proof Of Concepts
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ExecutorPlugin.sol#L52-L55
```
    /// @notice EIP712 domain name
    string private constant _NAME = "ExecutorPlugin";
    /// @notice EIP712 domain version
    string private constant _VERSION = "1.0";
```

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/PolicyValidator.sol#L25-L28
```
    /// @notice EIP712 domain name
    string private constant _NAME = "PolicyValidator";
    /// @notice EIP712 domain version
    string private constant _VERSION = "1.0";
```

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeDeployer.sol#L29-L30
```
    bytes32 internal constant _SAFE_CREATION_FAILURE_REASON =
        0xd7c71a0bdd2eb2834ad042153c811dd478e4ee2324e3003b9522e03e7b3735dc;
```

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeEnabler.sol#L25-L30
```
    /// @notice address of sentinel modules
    address internal constant _SENTINEL_MODULES = address(0x1);

    /// @notice guard storage slot
    /// @dev keccak256("guard_manager.guard.address")
    bytes32 internal constant _GUARD_STORAGE_SLOT = 0x4a204f620c8c5ccdca3fd54d003badd85ba500436a431f0cbda4f558c93c34c8;
```

## Recommendation
Only use a preceding underscore hyphen for constants supplied from the `Constants.sol` file.
```
    /// @notice EIP712 domain name
    string private constant NAME = "ExecutorPlugin";
    /// @notice EIP712 domain version
    string private constant VERSION = "1.0";
```


