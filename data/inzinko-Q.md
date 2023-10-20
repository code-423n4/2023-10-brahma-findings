## Validators may not be able to verify policy commits quickly Enough 

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/PolicyValidator.sol#L120C7-L141C113

The volume of transactions, may impede the speed at which a trusted validator makes sure the transaction being submitted follow the policy commit, this might be common if the policy commit EIP 712 Digest is written vaguely and the Validator has to be able to interpret what the policy commit allows or disallows. This process might hold up other pending transactions and cause a delay. This is as a result of their being a single Trusted Validator as specified in the ```AddressRegistry```

```solidity
  // Build validation struct hash
        bytes32 validationStructHash = TypeHashHelper._buildValidationStructHash(
            TypeHashHelper.Validation({
                transactionStructHash: transactionStructHash,
                policyHash: policyHash,
                expiryEpoch: expiryEpoch
            })
        );

        // Build EIP712 digest with validation struct hash
        bytes32 txnValidityDigest = _hashTypedData(validationStructHash);

        address trustedValidator = AddressProviderService._getAuthorizedAddress(_TRUSTED_VALIDATOR_HASH);

        // Empty Signature check for EOA signer
        if (trustedValidator.code.length == 0 && validatorSignature.length == 0) {
            // TrustedValidator is an EOA and no trustedValidator signature is provided
            revert InvalidSignature();
        }

        // Validate signature
        return SignatureCheckerLib.isValidSignatureNow(trustedValidator, txnValidityDigest, validatorSignature);
```
This can even cause more problem in which a front-running exploit can occur, if the transaction needed to be carried out by the actors in a sub account require quick and immediate approval, a User not necessarily malicious can actually get their transaction included in the mempool and executed first, thereby changing the expected result the sub account transaction intend to achieve

## Operators of a Sub Account can create and Deploy their own Sub account

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeDeployer.sol#L82C5-L103C6

This is possible as The Gnosis Safe is being set up that way, if an Owner of a console can deploy their own Sub account then the "Technical" owner of a sub account can also deploy their own sub account. But for any Transaction that involves moving out assets or any transactions not included in the policy hash, the trusted validator will not allow it. But Just as a Note the Operators can deploy their own Sub Account

```solidity
function deploySubAccount(address[] calldata _owners, uint256 _threshold, bytes32 _policyCommit, bytes32 _salt)
        external
        nonReentrant
        returns (address _subAcc)
    {
        // Policy commit is required for sub account
        if (_policyCommit == bytes32(0)) revert InvalidCommitment();

        // Check if msg.sender is a registered wallet
        WalletRegistry _walletRegistry = WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH));
        if (!_walletRegistry.isWallet(msg.sender)) revert NotWallet();

        // Deploy sub account
        _subAcc = _createSafe(_owners, _setupSubAccount(_owners, _threshold, msg.sender), _salt);

        // Register sub account to wallet
        _walletRegistry.registerSubAccount(msg.sender, _subAcc);

        // Update policy commit for sub account
        PolicyRegistry(AddressProviderService._getRegistry(_POLICY_REGISTRY_HASH)).updatePolicy(_subAcc, _policyCommit);
        emit SubAccountDeployed(_subAcc, msg.sender);
    }
```

## Operators can Loosen the threshold requirements and Remove other Operators 

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/lib/safe-contracts/contracts/base/OwnerManager.sol#L117C5-L124C6

This is also possible as Operators essentially are owners of the sub Account, Removing a threshold and removing another owner can only be done with a safe transaction, and a number of operators have that chance to remove an operator themselves without the Console Account Knowing about it, they can also reduce the threshold to any amount they wish.

```solidity
function changeThreshold(uint256 _threshold) public authorized {
        // Validate that threshold is smaller than number of owners.
        require(_threshold <= ownerCount, "GS201");
        // There has to be at least one Safe owner.
        require(_threshold >= 1, "GS202");
        threshold = _threshold;
        emit ChangedThreshold(threshold);
    }
```

## Zero Check not Available when registering Policy Commits

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/registries/PolicyRegistry.sol#L35C3-L60C1

A zero check for address of the account should be put in place to avoid further computations, in the ```PolicyRegistry``` contract, when updating polices, since we know a zero address wont pass those if and else statements to avoid further work, just put a check whether the address being passed is a zero address or not.

```solidity
  function updatePolicy(address account, bytes32 policyCommit) external {
        if (policyCommit == bytes32(0)) {
            revert PolicyCommitInvalid();
        }

        WalletRegistry walletRegistry = WalletRegistry(AddressProviderService._getRegistry(_WALLET_REGISTRY_HASH));

        bytes32 currentCommit = commitments[account];

        // solhint-disable no-empty-blocks
        if (
            currentCommit == bytes32(0)
                && msg.sender == AddressProviderService._getAuthorizedAddress(_SAFE_DEPLOYER_HASH)
        ) {
            // In case invoker is safe  deployer
        } else if (walletRegistry.isOwner(msg.sender, account)) {
            //In case invoker is updating on behalf of sub account
        } else if (msg.sender == account && walletRegistry.isWallet(account)) {
            // In case invoker is a registered wallet
        } else {
            revert UnauthorizedPolicyUpdate();
        }
        // solhint-enable no-empty-blocks
        _updatePolicy(account, policyCommit);
    }

```