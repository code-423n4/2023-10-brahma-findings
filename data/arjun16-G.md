Variables: No need to explicitly initialize variables with default values

If a variable is not set/initialized, it is assumed to have the default value(0 for uint and uint types, False for bool, address(0) for address....). Explicitly initializing it with its default value is an anti-pattern and wastes gas.

Instance:

   src/core/SafeModeratorOverridable.sol:
    21:         uint8 public constant DIFFER_SAFE_MOD = 0;

I suggest removing explicit initialization for default values.





