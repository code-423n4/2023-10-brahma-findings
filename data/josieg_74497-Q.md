QA reports (Low risk and Non-critical):

    1. contracts/src/core/registries/PolicyRegistry.sol
       ===> Line No. 67 -68

       emit event, Line No. 67 after line No. 68, not before, unless you want to show status before policy commit update

    function _updatePolicy(address account, bytes32 policyCommit) internal {
        emit UpdatedPolicyCommit(account, policyCommit, commitments[account]);
        commitments[account] = policyCommit;
    }


    2. /src/core/AddressProvider.sol
       ===> Line No. 52 – 57 

       emit event, Line No. 55 after line No. 56, not before.

	function setGovernance(address _newGovernance) external {
       	 _notNull(_newGovernance);
       	 _onlyGov();
       	 emit GovernanceTransferRequested(governance, _newGovernance);
       	 pendingGovernance = _newGovernance;
    	}

	and,

	===> Line No. 62 – 69 
	emit event, Line No. 66 after line No. 67, not before, unless you want to show the before and after

	function acceptGovernance() external {
        	if (msg.sender != pendingGovernance) {
        		revert NotPendingGovernance(msg.sender);
        	}
       	 emit GovernanceTransferred(governance, msg.sender);
       	 governance = msg.sender;
       	delete pendingGovernance;
    	}