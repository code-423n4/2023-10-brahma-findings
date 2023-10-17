## Hardcoded Constants 

### Risk Rating: 

Low

### Lines of Code Affected:

```solidity
bytes32 public constant _WALLET_REGISTRY_HASH = 0xa9429271a79fb5764ef 217f6f41451716b3de5b7cdfd360140244af25e8ed7000;

bytes32 public constant _SAFE_DEPLOYER_HASH = 0xa9429271a79fb5764efa79f614551716b3de5b7c dfd360140244af25e8ed7001; 
```

### Impact:

The addresses of the `WalletRegistry` and `SafeDeployer` contracts are currently hardcoded using constants `_WALLET_REGISTRY_HASH` and `_SAFE_DEPLOYER_HASH`.

This means that upgrading these contracts would break the `PolicyRegistry` logic as it relies on these specific addresses.

### Proof of Concept:

Suppose the `WalletRegistry` contract is upgraded to `WalletRegistryV2` with a new address. The `PolicyRegistry` will continue using the hardcoded `_WALLET_REGISTRY_HASH` leading to errors.

### Recommended Mitigation: 

Instead of hardcoded hashes, fetch the addresses dynamically from the `AddressProvider`:

```solidity
// Get wallet registry address
address walletRegistry = AddressProvider(_addressProvider).getAddress(WALLET_REGISTRY_ID); 

// Get safe deployer address
address safeDeployer = AddressProvider(_addressProvider).getAddress(SAFE_DEPLOYER_ID);
```

This allows flexibly changing these addresses via the AddressProvider rather than requiring a contract upgrade.

---

## No Rate Limiting

### Risk Rating: 

Low  

### Lines of Code Affected:

```solidity
function updatePolicy(address account, bytes32 policyCommit) external {
   // Code snippet
}
```

### Impact:

There is currently no limit enforced on how often an authorized entity can update policies. 

This allows attackers to spam policy updates if they gain access to authorized accounts. They could rapidly modify policies before owners can respond.

Frequent policy updates also increase gas usage on-chain, creating denial of service risk.

### Proof of Concept:

An attacker who compromises an authorized account can make rapid policy updates:

```solidity
for (uint i = 0; i < 100; i++) {

  policyRegistry.updatePolicy(targetAddress, maliciousPolicy);

}
```

This allows flooding targets with policy updates.

### Recommended Mitigation:

Implement a simple rate limiting mechanism:

```solidity
// Mapping for last update timestamp 
mapping(address => uint256) public lastUpdateTime;

// Limit to 1 update per 2 hours 
uint256 public constant UPDATE_LIMIT = 2 hours;

function updatePolicy(address account, bytes32 policy) external {

  require(lastUpdateTime[msg.sender] + UPDATE_LIMIT < block.timestamp, "Rate limited"); 

  // Rest of logic

  lastUpdateTime[msg.sender] = block.timestamp;

}
```

This restricts each authorized entity to a maximum update frequency. More complex limiting mechanisms can also be implemented.


## Unclear Revert Reasons

### Risk Rating: 
QA 

### Lines of Code Affected:

```solidity
error PolicyCommitInvalid();

error UnauthorizedPolicyUpdate(); 
```

### Impact:

The custom errors `PolicyCommitInvalid` and `UnauthorizedPolicyUpdate` currently do not contain any details on the reason for failure. 

When transactions revert due to these errors, dApps and users have no insights into why the transaction failed. This hampers debugging efforts and transaction troubleshooting.

Lack of clear revert reasons also poses security risks. Attackers can intentionally trigger failures to map out vulnerabilities without detection.

### Proof of Concept:

When the following transaction fails, the reason is unclear from the error:

```solidity
policyRegistry.updatePolicy(targetAddress, invalidPolicy)
``` 

This could fail due to an invalid policy or unauthorized sender, but that cannot be distinguished currently.

### Recommended Mitigation:

Enhance custom errors to include relevant details on the failure reason:

```solidity
error PolicyCommitInvalid(address account, bytes32 policy); 

error UnauthorizedPolicyUpdate(address sender, address account);
```

Emit these errors with relevant data to provide debugging information.



## Overflow Risk in `_genNonce`

### Risk Rating: 
Low

### Lines of Code Affected:

```solidity
// Increment counter on each deployment
ownerSafeCount[ownersHash]++; 

uint256 nonce = uint256(
  keccak256(abi.encodePacked(  
    ..., 
    ownerSafeCount[ownersHash],
    ...))
);
```

### Impact:

The `_genNonce` function uses an incrementing counter to generate a unique nonce for each safe deployment. 

This counter could eventually overflow if 2^256 safe deployments are performed per owner set. This would lead to non-unique nonces being generated.

The risk is currently low due to the improbably large number of deployments required. But it could become a realistic issue in the very long term.

### Proof of Concept:

By incrementing the counter in a loop, we can demonstrate overflow:

```solidity
function testNonceOverflow() public {
  uint256 preCount = ownerSafeCount[ownersHash];

  for (uint i = 0; i < 2**256; i++) {
    ownerSafeCount[ownersHash]++; 
  }

  uint256 postCount = ownerSafeCount[ownersHash] 
  
  assertEq(preCount, postCount);
}
```

### Recommended Mitigation:

Use OpenZeppelin's `Counters` library which has overflow protection:

```solidity
using Counters for Counters.Counter;
Counters.Counter private _ownerCounts[ownersHash];

// Increment with overflow protection
_ownerCounts[ownersHash].increment(); 
```

This will prevent the nonce generation from ever overflowing.
