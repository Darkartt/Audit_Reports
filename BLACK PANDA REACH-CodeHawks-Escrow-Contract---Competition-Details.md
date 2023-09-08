# CodeHawks Escrow Contract - Competition Details - Findings Report

# Table of contents
- ## [Contest Summary](#contest-summary)
- ## [Results Summary](#results-summary)
- ## High Risk Findings
    - ### [H-01. Escrow.sol Contract Is Vulnerable to Permit() Function in DAI Contract](#H-01)
- ## Medium Risk Findings
    - ### [M-01. Unable to use fee-on-transfer tokens](#M-01)

- ## Gas Optimizations / Informationals
    - ### [G-01. Compromised/bribed/keys lost arbiter may lock funds in the contract forever](#G-01)

# <a id='contest-summary'></a>Contest Summary

### Sponsor: Cyfrin

### Dates: Jul 24th, 2023 - Aug 5th, 2023

[See more contest details here](https://www.codehawks.com/contests/cljyfxlc40003jq082s0wemya)

# <a id='results-summary'></a>Results Summary

### Number of findings:
   - High: 1
   - Medium: 1
   - Low: 0
  - Gas/Info: 1

# High Risk Findings

## <a id='H-01'></a>H-01. Escrow.sol Contract Is Vulnerable to Permit() Function in DAI Contract            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2023-07-escrow/blob/main/src/Escrow.sol

## Summary
If DAI is deposited into the contract, the buyer can use the `permit()` function on the DAI contract to withdraw the DAI without the approval of the Escrow contract. This could be exploited by malicious actors to steal funds from the Escrow contract.
## Vulnerability Details
Some tokens (DAI, RAI, GLM, STAKE, CHAI, HAKKA, USDFL, HNY) have a `permit()` implementation that does not follow `EIP2612`. Tokens that do not support permit may not revert, which could lead to the execution of later lines of code in unexpected scenarios.
## Impact
The `permit()` issue in the `Escrow.sol` contract could allow buyer to steal funds from the contract. Additionally, the issue could damage the reputation of the Escrow contract and make it less likely that sellers and buyers will use it in the future.
## Tools Used
Manual Review
## Recommendations
The Escrow.sol contract should not accept DAI deposits. If the contract must accept DAI deposits, then the contract should be updated to have a `permit()` function. Additionally, the contract could be updated to only accept tokens that do not have a `permit()` function.

		
# Medium Risk Findings

## <a id='M-01'></a>M-01. Unable to use fee-on-transfer tokens            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2023-07-escrow/blob/65a60eb0773803fa0be4ba72defaec7d8567bccc/src/Escrow.sol#L32-L51

https://github.com/Cyfrin/2023-07-escrow/blob/65a60eb0773803fa0be4ba72defaec7d8567bccc/src/Escrow.sol#L119-L128

## Summary
The documentation doesn't describe how "vetting" of tokens will work, but the mechanism is not deployed on chain, which means that technically users are able to use any ERC20 token. The only documentation restriction is that ERC777 are not supported. fee-on-transfer tokens are not mentioned, yet they will not work, because first in `EscrowFactory` `price` amount of tokens is transferred from user to `Escrow`, then in the constructor `balanceOf(address(this))` is checked to be at least `price`. In case fee-on-transfer tokens it will not be possible, as the `balanceOf` will diminish after transfer to `Escrow`. There is a workaround for it - directly sending tokens to `Escrow` address to be deployed. This way, seller will get amount - fees - less than expected.

## Vulnerability Details
A malicious seller can top up the newly created `Escrow` ahead of time to bypass the restriction of `Escrow.constructor` of token amount:
```
        if (tokenContract.balanceOf(address(this)) < price) revert Escrow__MustDeployWithTokenBalance();
```

This will make the constructor logic work, however later when either confirming the receipt or resolving a dispute, will make seller receive less tokens than expected.

## Impact
a) not possible to use fee-on-transfer tokens
b) making user receive less tokens than expected when fee-on-transfer tokens are used

## Tools Used
Manual analysis

## Recommendations
Check account token balances before transfering them and after, and calculate proper amount based on it.



# Gas Optimizations / Informationals

## <a id='G/I-01'></a>G/I-01. Compromised/bribed/keys lost arbiter may lock funds in the contract forever            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2023-07-escrow/blob/65a60eb0773803fa0be4ba72defaec7d8567bccc/src/Escrow.sol#L102-L106

https://github.com/Cyfrin/2023-07-escrow/blob/65a60eb0773803fa0be4ba72defaec7d8567bccc/src/Escrow.sol#L109

## Summary
The state machine of the Escrow does not allow to change the state from `Disputed` otherwise than by calling `resolveDispute()`, which can be only called by arbiter. That means that compromised, bribed, or one with keys lost cannot resolve the dispute, locking the funds in the smart contract forever.

## Vulnerability Details
First step, anyone can initiate a dispute:
https://github.com/Cyfrin/2023-07-escrow/blob/65a60eb0773803fa0be4ba72defaec7d8567bccc/src/Escrow.sol#L102-L106
```
    function initiateDispute() external onlyBuyerOrSeller inState(State.Created) {
        if (i_arbiter == address(0)) revert Escrow__DisputeRequiresArbiter();
        s_state = State.Disputed;
        emit Disputed(msg.sender);
    }

```

Then, in order to resolve a dispute, it has to be called by arebiter and the state has to be `Disputed`:
https://github.com/Cyfrin/2023-07-escrow/blob/65a60eb0773803fa0be4ba72defaec7d8567bccc/src/Escrow.sol#L109
```
    function resolveDispute(uint256 buyerAward) external onlyArbiter nonReentrant inState(State.Disputed)
```
In case that arbiter never calls it, the funds are locked forever.

## Impact
Funds lost in case that arbiter does not resolve dispute.

## Tools Used
Manual analysis.

## Recommendations
Consider making dispute auto-resolvable after specific amount of time. For example after a month since starting a dispute both buyer and seller are able to get half of the escrowed funds and arbiter does not receive anything, because they did not do what they should.
