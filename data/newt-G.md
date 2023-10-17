# No need to explicitly initialize variables with default values

If a variable is not set/initialized, it is assumed to have the default value (0 for uint, false for bool, address(0) for address...). Explicitly initializing it with its default value is an anti-pattern and wastes gas.

        uint256 i = 0;

https://github.com/code-423n4/2023-10-brahma/blob/a6424230052fc47c4215200c19a8eef9b07dfccc/contracts/src/libraries/SafeHelper.sol#L107

# Issue 2

    uint8 public constant DIFFER_SAFE_MOD = 0;

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeModeratorOverridable.sol#L21

# Issue 3

There is no need to declare a state variable to a variable created in a constructor because it is already immutable and internal. Function  _onlyDelegateCall() will work without _self being a state variable just a constructor.

    address internal immutable _self; 
https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeEnabler.sol#L23

    constructor() {
        _self = address(this);
    }

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeEnabler.sol#L32C1-L34C6


# Issue 3

State variable only used once. Suggest adding it within the function to save gas. Since it is constant and there will be no modification

    string public constant VERSION = "1";

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeDeployer.sol#L23C1-L23C42

    function _genNonce(bytes32 _ownersHash, bytes32 _salt) private returns (uint256) {
        return uint256(keccak256(abi.encodePacked(_ownersHash, ownerSafeCount[_ownersHash]++, _salt, VERSION)));
    }

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/SafeDeployer.sol#L253-L255

Recommendation, similar for all instances

    function _genNonce(bytes32 _ownersHash, bytes32 _salt) private returns (uint256) {
        string VERSION = "1";
        return uint256(keccak256(abi.encodePacked(_ownersHash, ownerSafeCount[_ownersHash]++, _salt, VERSION)));
    }

Other instance
    
    string private constant _VERSION = "1.0";

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/PolicyValidator.sol#L28C1-L28C46

    function _domainNameAndVersion() internal pure override returns (string memory name, string memory version) {
        return (_NAME, _VERSION);
    }

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/PolicyValidator.sol#L174C1-L176C6

Another Instance

    string private constant _VERSION = "1.0";

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ExecutorPlugin.sol#L55C1-L55C46

    function _domainNameAndVersion() internal pure override returns (string memory name, string memory version) {
        return (_NAME, _VERSION);
    }

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ExecutorPlugin.sol#L159C1-L161C6

Another Instance

    string private constant _NAME = "ExecutorPlugin";

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/ExecutorPlugin.sol#L53C1-L53C54

# Issue 4

If multiple functions are 0 in a function, embrace bitwise or clauses

        if (trustedValidator.code.length == 0 && validatorSignature.length == 0) 

https://github.com/code-423n4/2023-10-brahma/blob/dd0b41031b199a0aa214e50758943712f9f574a0/contracts/src/core/PolicyValidator.sol#L135C1-L135C83

Recommendation

        if ((trustedValidator.code.length && validatorSignature.length) == 0) {
            //do something