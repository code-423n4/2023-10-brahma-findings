# Low Risk And Non-Critical Issues

## 1.Duplicate function name
The same function name is used in the PolicyValidator contract, which may cause confusion
```
function isPolicySignatureValid(
        address account,
        address to,
        uint256 value,
        bytes memory data,
        Enum.Operation operation,
        bytes calldata signatures
    ) external view returns (bool) {
        ......
        return isPolicySignatureValid(account, transactionStructHash, signatures);
    }
    function isPolicySignatureValid(address account, bytes32 transactionStructHash, bytes calldata signatures)
        public
        view
        returns (bool)
    { ......
    }
```
### Recommended Mitigation
Change isPolicySignatureValid to _isPolicySignatureValid.
```
function isPolicySignatureValid(
        address account,
        address to,
        uint256 value,
        bytes memory data,
        Enum.Operation operation,
        bytes calldata signatures
    ) external view returns (bool) {
        ......
        return _isPolicySignatureValid(account, transactionStructHash, signatures);
    }
    function _isPolicySignatureValid(address account, bytes32 transactionStructHash, bytes calldata signatures)
        public
        view
        returns (bool)
    { ......
    }
```