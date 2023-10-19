Gas Optimizations:

    1. contracts/src/core/AddressProvider.sol
       ===> Line No. 139 - 141
       Change _ onlyGov() to a modifier or an inline function to save on function calling gas.

	function _onlyGov() internal view {
        	if (msg.sender != governance) revert NotGovernance(msg.sender);
    	}

	===> Line No. 147 – 149
       Change _notNull() to a modifier or an inline function to save on function calling gas.

    	function _notNull(address addr) internal pure {
       	 if (addr == address(0)) revert NullAddress();
    	}


    2. contracts/src/core/ConsoleFallbackHandler.sol
	===> Line No. 94  //@audit magic number 10, and why 10?
	(address[] memory array,) = safe.getModulesPaginated(SENTINEL_MODULES, 10);


    3. src/core/SafeEnabler.sol
       ===>Line No. 48
       use two if statements with revert custom error instead of require with the && operator.
       
       require(module != address(0) && module != _SENTINEL_MODULES, "GS101"); 
       

    4. src/core/PolicyValidator.sol

       ===> Line No. 135 
       (a). Use nested if statements to save on gas instead of the && operator
       (b). Remove magic numbers. What is the zero for? Declare as a const or private const.

       if(trustedValidator.code.length == 0) {
           if(validatorSignature.length == 0) {
               // ...
          }
       }
       instead of:
       if(trustedValidator.code.length == 0 && validatorSignature.length == 0)  

	and,

	===>Line No. 162, 164-166 
	Magic numbers; “4” and “8”. Declare them as private constants.
	if (length < 8) revert InvalidSignatures();
 	uint32 sigLength = uint32(bytes4(_signatures[length - 8:length – 4]));
	expiryEpoch = uint32(bytes4(_signatures[length – 4:length]));
	validatorSignature = _signatures[length - 8 - sigLength:length - 8];


    5. src/core/TransactionValidator.sol
       ===>Line No. 165  Use nested if statements to save on gas instead of the && operator
       
       if (_from == _to) {
           if( _value == 0) {
               if(_operation == Enum.Operation.Call) {
                   //do stuff….
               }
             }
        }

        instead of this:
        if (_from == _to && _value == 0 && _operation == Enum.Operation.Call) {


    6. contracts/src/libraries/SafeHelper.sol
       ===>Lint No. 114   why not revert first in this if/else statement? might save gas!
       Check if (_txns[i].callType == Types.CallType.STATICCALL) { first 

        uint256 i = 0;
        do {
            // Enum.Operation.Call is 0
            uint8 call = uint8(Enum.Operation.Call);
            if (_txns[i].callType == Types.CallType.DELEGATECALL) {
                call = uint8(Enum.Operation.DelegateCall);
            } else if (_txns[i].callType == Types.CallType.STATICCALL) {
                revert InvalidMultiSendCall(i);   //@audit why not revert first in this if/else statement? might save gas!
            }
            ..
	..
    }