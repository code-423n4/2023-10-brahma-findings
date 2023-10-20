## Validators may not be able to verify policy commits quickly Enough 

The volume of transactions, may impede the speed at which a trusted validator makes sure the transaction being submitted follow the policy commit, this might be common if the policy commit EIP 712 Digest is written vaguely and the Validator has to be able to interpret what the policy commit allows or disallows. This process might hold up other pending transactions and cause a delay. This is as a result of their being a single Trusted Validator as specified in the ```AddressRegistry```
This can even cause more problem in which a front-running exploit can occur, if the transaction needed to be carried out by the actors in a sub account require quick and immediate approval, a User not necessarily malicious can actually get their transaction included in the mempool and executed first, thereby changing the expected result the sub account transaction intend to achieve

## Operators of a Sub Account can create and Deploy their own Sub account

This is possible as The Gnosis Safe is being set up that way, if an Owner of a console can deploy their own Sub account then the "Technical" owner of a sub account can also deploy their own sub account. But for any Transaction that involves moving out assets or any transactions not included in the policy hash, the trusted validator will not allow it. But Just as a Note the Operators can deploy their own Sub Account

## Operators can Loosen the threshold requirements and Remove other Operators 

This is also possible as Operators essentially are owners of the sub Account, Removing a threshold and removing another owner can only be done with a safe transaction, and a number of operators have that chance to remove an operator themselves without the Console Account Knowing about it, they can also reduce the threshold to any amount they wish.

## Zero Check not Available when registering Policy Commits

A zero check for address of the account should be put in place to avoid further computations, in the ```PolicyRegistry``` contract, when updating polices, since we know a zero address wont pass those if and else statements to avoid further work, just put a check whether the address being passed is a zero address or not