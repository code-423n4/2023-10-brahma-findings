Summary
Missing zero address check can set registry to zero address

Vulnerability Details
The `_registry` parameter in the `setRegistry()` function in the AddressProvider.sol file is missing the zero address check which could allow the treasury to be set to zero address.

Vulnerable Code:
src/core/AddressProvider.sol:: L97
Few other instances:
src/core/ExecutorPlugin.sol::L68 
--> takes the exec request, does not checks for 0 address, same gets passed into L86
src/code/PolicyValidator.sol::L54 - account, to

Impact
Registry address can be set to 0 address.

Tools Used
Manual Review

Remediation Steps
Use the _notnull() helper function to ensured that `_registry` address is not a zero address.