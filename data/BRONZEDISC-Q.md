## QA
---

### Function Visibility [1]

- Order of Functions: Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private. Within a grouping, place the view and pure functions last.

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol

```solidity
// place these external functions before public ones
83:    function isValidSignature(bytes32 _dataHash, bytes calldata _signature) external view returns (bytes4) {
91:    function getModules() external view returns (address[] memory) {
104:    function simulate(address targetContract, bytes calldata calldataPayload)
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModerator.sol

```solidity
// place these view external functions for last
33:    function checkTransaction(
70:    function checkAfterExecution(bytes32 txHash, bool success) external view override {
```

---

### natSpec missing [2]

Some functions are missing @params or @returns. Specification Format.” These are written with a triple slash (///) or a double asterisk block(/** ... */) directly above function declarations or statements to generate documentation in JSON format for developers and end-users. It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). These comments contain different types of tags:
- @title: A title that should describe the contract/interface @author: The name of the author (for contract, interface) 
- @notice: Explain to an end user what this does (for contract, interface, function, public state variable, event) 
- @dev: Explain to a developer any extra details (for contract, interface, function, state variable, event) 
- @param: Documents a parameter (just like in doxygen) and must be followed by parameter name (for function, event)
- @return: Documents the return variables of a contract’s function (function, public state variable)
- @inheritdoc: Copies all missing tags from the base function and must be followed by the contract name (for function, public state variable)
- @custom…: Custom tag, semantics is application-defined (for everywhere)

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ExecutorPlugin.sol

```solidity
60:    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol

```solidity
32:    event SafeProxyCreationFailure(address indexed singleton, uint256 indexed nonce, bytes initializer);
33:    event ConsoleAccountDeployed(address indexed consoleAddress);
34:    event SubAccountDeployed(address indexed subAccountAddress, address indexed consoleAddress);
35:    event PreComputeAccount(address[] indexed owners, uint256 indexed threshold);
42:    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}

// @return
110:    function _setupConsoleAccount(address[] memory _owners, uint256 _threshold, bool _policyHashValid)
168:    function _setupSubAccount(address[] memory _owners, uint256 _threshold, address _consoleAccount)
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeDeployer.sol

```solidity
32:    event SafeProxyCreationFailure(address indexed singleton, uint256 indexed nonce, bytes initializer);
33:    event ConsoleAccountDeployed(address indexed consoleAddress);
34:    event SubAccountDeployed(address indexed subAccountAddress, address indexed consoleAddress);
35:    event PreComputeAccount(address[] indexed owners, uint256 indexed threshold);
42:    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}

// @return missing
110:    function _setupConsoleAccount(address[] memory _owners, uint256 _threshold, bool _policyHashValid)
168:    function _setupSubAccount(address[] memory _owners, uint256 _threshold, address _consoleAccount)
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProviderService.sol

```solidity
27:    constructor(address _addressProvider) {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/WalletRegistry.sol

```solidity
29:    event RegisterWallet(address indexed wallet);
20:    event RegisterSubAccount(address indexed wallet, address indexed subAccount);
29:    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/ExecutorRegistry.sol

```solidity
23:    event RegisterExecutor(address indexed _subAccount, address indexed _owner, address indexed _executor);
24:    event DeRegisterExecutor(address indexed _subAccount, address indexed _owner, address indexed _executor);
29:    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/registries/PolicyRegistry.sol

```solidity
19:    event UpdatedPolicyCommit(address indexed account, bytes32 policyCommit, bytes32 oldPolicyCommit);
24:    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/PolicyValidator.sol

```solidity
30:    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/AddressProvider.sol

```solidity
21:    event RegistryInitialised(address indexed registry, bytes32 indexed key);
22:    event AuthorizedAddressInitialised(address indexed authorizedAddress, bytes32 indexed key);
23:    event GovernanceTransferRequested(address indexed previousGovernance, address indexed newGovernance);
24:    event GovernanceTransferred(address indexed previousGovernance, address indexed newGovernance);
43:    constructor(address _governance) {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/ConsoleFallbackHandler.sol

```solidity
29:    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}

// @dev missing
91:    function getModules() external view returns (address[] memory) {

// @return missing
104:    function simulate(address targetContract, bytes calldata calldataPayload)
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModerator.sol

```solidity
17:    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeEnabler.sol

```solidity
19:    event EnabledModule(address module);
20:    event ChangedGuard(address guard);
32:    constructor() {
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/SafeModeratorOverridable.sol

```solidity
23:    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}
```

https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/src/core/TransactionValidator.sol

```solidity
// @param missing
54:    constructor(address _addressProvider) AddressProviderService(_addressProvider) {}
```

