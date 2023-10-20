# QA Report

## Remarks
- Code test coverage seems comprehensive
- Arbitrum compatibility is accounted for by keeping the pragma version at 0.8.19 in each contract
- Code seems to be well & highly documented
- Flow diagrams were provided to showcase the different execution flows possible


## Low risk: **`Solady`** could be bumped to a more recent version

The **`solady`** version used in this protocol is 0.0.113. As of today, the latest version is 0.0.129.
Updating to a more recent version would be a good idea because it would include various fixes and optimizations.

## Non critical: No implementation provided in if and else-if blocks in **`PolicyRegistry.updatePolicy`**

There is 1 empty if and 2 empty else-if blocks in **`PolicyRegistry.updatePolicy`**. It seems like there should be some side effects happening
in there judging by the comments provided, however that is not the case, so I thought it's worth pointing out.

## Non critical: Naming of **`_policyHashValid`** in **`SafeDeployer`** is slighly deceiving and inconsistent

The name used for the **`_policyHashValid`** boolean variable is slightly deceiving. It only checks
if the **`policyCommit`** is set or not but does not do any validation per se. This is used in **`deployConsoleAccount`** and **`_setupConsoleAccount`**. A more appropriate name would be **`_hasPolicyHash`** or something along those lines.

## Non critical: Naming of **`InvalidCommitment`** event in **`SafeDeployer`** is slightly deceiving and inconstistent

The name used for this event is slightly deceiving and inconsistent. It's only used inside of the **`deploySubAccount`** function and to check
whether the **`_policyCommit`** argument is a non-null. A similar use can be found in **`PolicyValidator.isPolicySignatureValid`** where the same check is done but the emitted event is called **`NoPolicyCommit`**.

## Non critical: Naming of **`PolicyCommitInvalid`** event in **`PolicyRegistry`** is slightly deceiving and inconsistent

The name used for this event is slightly deceiving as it not used to do any validation other than checking whether
there is a **`policyCommit`** or not in **`PolicyRegistry.updatePolicy`**. A better name would be **`NoPolicyCommit`**. That's also how it's named in **`PolicyValidator.sol`**.
