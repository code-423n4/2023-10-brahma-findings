This codebase was evaluated following the user flow. The provided documentation to proceed as such were very helpful and accurate, allowing readers to understand the deep mechanism of the protocol's codebase. 
Although this codebase would be considered medium to small size, the entry points are rather restricted which makes it more robust and safe by design.

There is a guardian address that is optional on the Console account however mandatory on sub accounts, in the context of this protocol, the guardian is actually the `SafeModerator` contract for subs and `SafeModeratorOverridable` for the Console accounts. Now the contracts are audited, so the current deployments of accounts and sub accounts are safe. 

The protocol governance has the freedom to change any of the contracts, if this event was to happen, it be would considered a best practice for the new contracts to go through another audit before being deployed.

This protocol offers a new innovative way to introduce hierarchy, delegation and flexibility among safe addresses. The codebase itself could gain clarity with more consistent natspec and comments through the code. For example mappings are sometimes made clearer with the new annotation however this methodology is not preserved through the codebase. Keep in mind that, these are just details, and that overall this protocol in generally very well documented and explained. 

### Time spent:
30 hours