## [N-01] Use a modifier for access control

Consider using a modifier to implement access control instead of inlining the condition/requirement in the function’s body.

There are 5 instances of this issue, in 2 files:

    File: src/core/AddressProvider.sol	

    52: function setGovernance(address _newGovernance) external {
    53:     _notNull(_newGovernance);
    54:     _onlyGov();

    62: function acceptGovernance() external {
    63:     if (msg.sender != pendingGovernance) {
    64:         revert NotPendingGovernance(msg.sender);
    65:     }

    77: function setAuthorizedAddress(bytes32 _key, address _authorizedAddress, bool _overrideCheck) external {
    78:     _onlyGov();

    97: function setRegistry(bytes32 _key, address _registry) external {
    98:     _onlyGov();

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol

    File: src/core/registries/WalletRegistry.sol	

    49: function registerSubAccount(address _wallet, address _subAccount) external {
    50:     if (msg.sender != AddressProviderService._getAuthorizedAddress(_SAFE_DEPLOYER_HASH)) revert InvalidSender();

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol

## [N-02] Lack of address(0) checks in the constructor

Zero-address check should be used in the constructors, to avoid the risk of setting smth as address(0) at deploying time.

There are 2 instances of this issue, in 2 files:

    File: src/core/AddressProvider.sol	

    43: constructor(address _governance) {

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol

    File: src/core/AddressProviderService.sol		

    27: constructor(address _addressProvider) {

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol

## [N-03] According to the syntax rules, use *=> mapping (* instead of *=> mapping(* using spaces as keyword

There is 1 instance of this issue, in 1 file:

    File: src/core/ExecutorPlugin.sol	

    58: mapping(address account => mapping(address executor => uint256 nonce)) public executorNonce;

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol

## [N-04] Assembly Codes Specific – Should Have Comments

Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Assembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.

There is 1 instance of this issue, in 1 file:

    File: src/core/SafeEnabler.sol	

    71: assembly {

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol

## [N-05] Remove commented out code

There is 1 instance of this issue, in 1 file:

    File: src/core/ConsoleFallbackHandler.sol	

    21: //keccak256("SafeMessage(bytes message)");

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol