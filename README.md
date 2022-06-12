(remember to `git submodule update --init --recursive` after cloning)

# Token with revocable transfer restrictions

## Summary
* Based on solmate code. 
* Adds authorizer `mint`.
* Users can `burn` their tokens.
* Requires authentication on `transfer`, `transferFrom` and `mint`.
* Can remove authentication later.
* Editable `script/deploy.sol` to set token name,symbol & owner.

**The Auth contract differs from solmate's vanilla Auth** : `authority` is immutable. This saves a SLOAD.

**The contract is structured** as inheriting from `ERC20` and `RolesAuthority`. It defines itself as its own authority, so authenticated methods call `this.canCall` to check authorization.

**The owner is given in the constructor** and the owner can mint, and transfer at will. Once the proper authorizations have been setup, the owner should be set to `0`.

We decide on the following roles:

| Role | Can call                   |
|------|----------------------------|
| 0    | `transfer`, `transferFrom` |
| 1    | `mint`                     |

**To set roles**, owner should call (once):
```soldity
token.setRoleCapability(0,address(token),Token.transfer.selector,true);
token.setRoleCapability(0,address(token),Token.transferFrom.selector,true);
token.setRoleCapability(1,address(token),Token.mint.selector,true);
```
**This will be done automatically when calling `deploy.sol`**

**To give transfer rights** to an address `addr`, owner should call:
```solidity
token.setUserRole(addr,0,true);
```

**To enable transfers for everyone**, owner should call:
```
token.setPublicCapability(address(token),Token.transfer.selector,true);
token.setPublicCapability(address(token),Token.transferFrom.selector,true);
```
Compared to regular transfers, there will then be 1 extra SLOAD (loading the boolean `isCapabilityPublic(address(token), Token.transfer*.selector)`).

**To give mint rights**, to an addresss `addr`, owner should call
```solidity
token.setUserRole(addr,1,true); // for mint
```

**To remove itself**, owner should call (⚠️ There is no going back ⚠):
```solidity
token.setOwner(address(0));
```




