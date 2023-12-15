# First Flight #5: Santa's List - Findings Report

# Table of contents
- ## [Contest Summary](#contest-summary)
- ## [Results Summary](#results-summary)
- ## High Risk Findings
    - ### [H-01. SantasList__UnrestrictedStatusChange](#H-01)
    - ### [H-02. SantasList__UnauthorizedBurn](#H-02)
    - ### [H-03. SantasList__DoubleCollect](#H-03)
- ## Medium Risk Findings
    - ### [M-01. NFT Ownership Ambiguity in Hard Fork Scenarios](#M-01)



# <a id='contest-summary'></a>Contest Summary

### Sponsor: First Flight #5

### Dates: Nov 30th, 2023 - Dec 7th, 2023

[See more contest details here](https://www.codehawks.com/contests/clpba0ama0001ywpabex01hrp)

# <a id='results-summary'></a>Results Summary

### Number of findings:
   - High: 3
   - Medium: 1
   - Low: 0


# High Risk Findings

## <a id='H-01'></a>H-01. SantasList__UnrestrictedStatusChange            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2023-11-Santas-List/blob/main/src/SantasList.sol#L121-L125

## Summary
In the `checkList() `function of the Santa's List smart contract it could allow anyone to drain the contract of Santa tokens. This vulnerability is caused by the function lacking access control, allowing anyone to call it and give themselves an `EXTRA_NICE` status, which grants them the ability to collect a large number of tokens.
## Vulnerability Details
The `checkList()` function in the Santa's List smart contract does not implement any access control measures, allowing anyone to call it and assign themselves an `EXTRA_NICE` status. This status grants users the ability to collect a large number of Santa tokens by repeatedly calling the `collectPresent()` function. As a result, anyone can exploit this vulnerability to drain the contract of Santa tokens.
## Impact
An attacker could exploit this vulnerability to repeatedly call the collectPresent() function with the `EXTRA_NICE` status, draining the contract of Santa tokens. This could lead to a severe depletion of the contract's resources and disrupt the protocol's economy.
## Tools Used
Manual Review
## Recommendations
Implement an `onlySanta` modifier to restrict the ability to call the `checkList()` function.
```
function checkList(address person, Status status) external onlySanta {
        s_theListCheckedOnce[person] = status;
        emit CheckedOnce(person, status);
    }
```
## <a id='H-02'></a>H-02. SantasList__UnauthorizedBurn            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2023-11-Santas-List/blob/main/src/SantasList.sol#L172-L176

## Summary
A critical vulnerability has been discovered in the `buyPresent() function` of the `SantasList.sol` smart contract that could allow an attacker to burn any user's Santa tokens without their consent. This vulnerability is caused by the function not verifying that the token owner has approved the burn transaction. This vulnerability could be exploited to steal Santa tokens from other users.
## Vulnerability Details
An attacker could exploit this vulnerability by calling the `buyPresent() function` with the address of the victim as the presentReceiver parameter. This would cause the victim's Santa tokens to be burned, even if they had not approved the transaction.
## Impact
This vulnerability could allow an attacker to:

**Steal Santa tokens from other users

**Disrupt the operation of the protocol

## Tools Used
Manual Review
## Recommendations
Modify the buyPresent() function to verify that the token owner has approved the burn transaction.
For example, the function can be modified to :
```
function buyPresent(address presentReceiver) external {
  // Verify that the token owner has approved the burn transaction
  require(i_santaToken.allowance(presentReceiver, address(this)) >= 1, "Burn not approved");

  // Burn the token
  i_santaToken.burn(presentReceiver);

  // Mint a new NFT for the present receiver
  _mintAndIncrement();
}
```
## <a id='H-03'></a>H-03. SantasList__DoubleCollect            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2023-11-Santas-List/blob/main/src/SantasList.sol#L147-L166

## Summary
A critical vulnerability has been discovered in the `collectPresent() function` of the `SantasList.sol` smart contract that could allow an attacker to reclaim their presents multiple times. This vulnerability is caused by the function not tracking whether the user has previously collected a present. This vulnerability could be exploited to mint an unlimited number of Santa tokens.
## Vulnerability Details
An attacker could exploit this vulnerability by sending or burning their NFT and then calling the `collectPresent() function` again. This would allow the attacker to mint a new NFT, even though they had already collected a present.
## Impact
This vulnerability could allow an attacker to:

**Mint an unlimited number of Santa tokens

**Disrupt the operation of the protocol

**Take control of affected systems

## Tools Used
Manual Review
## Recommendations
Modify the `collectPresent() function` to store a flag indicating whether the user has already collected a present

Example of how it can be integrated :

First, we need to create the variable :
```solidity
byte8 bool s_alreadyCollected;
```
Then we ned to modify `collectPressent()`
```
function collectPresent() external {
if (block.timestamp < CHRISTMAS_2023_BLOCK_TIME) {
revert SantasList__NotChristmasYet();
}
if (balanceOf(msg.sender) > 0) {
revert SantasList__AlreadyCollected();
}
if (s_theListCheckedOnce[msg.sender] == Status.NICE && s_theListCheckedTwice[msg.sender] == Status.NICE) {
if (s_alreadyCollected[msg.sender]) {
revert SantasList__AlreadyCollected();
}
_mintAndIncrement();
s_alreadyCollected[msg.sender] = true;
return;
} else if (
s_theListCheckedOnce[msg.sender] == Status.EXTRA_NICE
&& s_theListCheckedTwice[msg.sender] == Status.EXTRA_NICE
) {
if (s_alreadyCollected[msg.sender]) {
revert SantasList__AlreadyCollected();
}
_mintAndIncrement();
i_santaToken.mint(msg.sender);
s_alreadyCollected[msg.sender] = true;
return;
}
revert SantasList__NotNice();
}
```
		
# Medium Risk Findings

## <a id='M-01'></a>M-01. NFT Ownership Ambiguity in Hard Fork Scenarios            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2023-11-Santas-List/blob/main/src/SantasList.sol#L187-L190

## Summary
The current implementation of the `tokenURI` function in the NFT contract doesn't consider the potential impact of hard forks on NFT ownership. During a hard fork, the blockchain splits into two separate chains, and the current `tokenURI` function would return the same URI for both chains, potentially leading to confusion and ownership disputes.
## Vulnerability Details
This vulnerability could lead to confusion and potential ownership disputes among NFT holders if the contract doesn't handle hard forks appropriately.
## Impact
This vulnerability poses a severe threat to the integrity of NFT ownership. If not addressed, it could lead to:

* Ownership Disputes: Conflicting URI responses for the same NFT across different chains could lead to ownership disputes and potential legal ramifications.

* Market Disruption: Uncertainty regarding NFT ownership could disrupt NFT markets, potentially causing value fluctuations and eroding trust in the project.

* User Confusion: The lack of clear ownership identification could confuse NFT holders and hinder their ability to manage their assets effectively.
## Tools Used
Manual Review
## Recommendations
Incorporate chain ID verification into the `tokenURI` function. This can be achieved by adding the following line:
```
require(1 == chain.chainId);
```




