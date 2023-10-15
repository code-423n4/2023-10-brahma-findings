Abi.encodePacked() should be replaced with string.concat()


since Solidity 0.8.12 you can finally use string.concat(). allows one to concatenate a list of bytes/strings, without extra padding. 

bytes.concat() also use but it's older version. bytes.concat() was added because abi.encodePacked() might be deprecated in favor of having more specific functions at some point in the future. Concatenating bytes arrays before hashing seems to be its main use case for now.

The conversions make the use of bytes.concat() for string a bit verbose, which is why -> https://github.com/ethereum/solidity/issues/12087




There are 5 instances of this issue:

```
File: src/libraries/SafeHelper.sol

125:                 packedTxns = abi.encodePacked(packedTxns, encodedTxn);

```

```
File: src/core/ConsoleFallbackHandler.sol

70:          return keccak256(abi.encodePacked(bytes1(0x19), bytes1(0x01), safe.domainSeparator(), safeMessageHash));
```

```
File: src/libraries/SafeHelper.sol

119              bytes memory encodedTxn = abi.encodePacked(
120                  bytes1(call), bytes20(_txns[i].target), bytes32(_txns[i].value), bytes32(calldataLength), _txns[i].data
121:             );

```

```
File: src/core/SafeDeployer.sol

144:             data: abi.encodePacked(WalletRegistry.registerWallet.selector)
```

```
File: src/libraries/SafeHelper.sol

88           bytes memory signatures = abi.encodePacked(
89               bytes12(0), // Padding for signature verifier address
90               bytes20(owner), // Signature Verifier
91               bytes32(0), // Position of extra data bytes (last set of data)
92               bytes1(hex"01") // Signature Type - 1 (presigned transaction)
93:          );
```