# Audit_Reports

# Foundry DeFi Stablecoin CodeHawks Audit Contest - Findings Report
# CodeHawks Escrow Contract - Competition Details - Findings Report

# Table of contents
- ## [Contest Summary](#contest-summary)
- ## [Results Summary](#results-summary)
- ## High Risk Findings
    - ### [H-01. getUsdValue function in DSCEngine.sol Contract Is Not Accurate for All Tokens](#H-01)
    - ### [H-02. Hardcoded Minimal_Health_Factor Bug in DSCEngine.sol Contract](#H-02)
- ## Medium Risk Findings
    - ### [M-01. block.timestamp may be unreliable in short term on L2s](#M-01)
    - ### [M-02. Hardcoded hearthbeat](#M-02)
    - ### [M-03. Lack of sequencer up-time check can lead to stale oracle prices](#M-03)



# <a id='contest-summary'></a>Contest Summary

### Sponsor: Cyfrin

### Dates: Jul 24th, 2023 - Aug 5th, 2023

[See more contest details here](https://www.codehawks.com/contests/cljx3b9390009liqwuedkn0m0)

# <a id='results-summary'></a>Results Summary

### Number of findings:
   - High: 2
   - Medium: 3
   - Low: 0


# High Risk Findings

## <a id='H-01'></a>H-01. getUsdValue function in DSCEngine.sol Contract Is Not Accurate for All Tokens            

### Relevant GitHub Links
	
https://github.com/REACH-black-panda/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/libraries/OracleLib.sol#L26-L32

## Summary
The `function getUsdValue` in the DSCEngine.sol contract is calculating the price of USD based WETH/WTBC decimals. However, the function does not take into account the decimals of some tokens like USDC or any token supported by chainlink that has less then 18 decimals. This means that the function will give a wrong price if a different token is used, such as USDC.
## Vulnerability Details
When user Redeem Collateral he might receive less then expected
## Impact
The `function getUsdValue` in the DSCEngine.sol contract could give users a wrong price for USDC. This could lead to users making bad financial decisions or losing money.
## Tools Used
Manual Review
## Recommendations
The `getUsdValue function` should be used with caution. Users should be aware that the function may not be accurate for all tokens.
Do not Hardcode it
## <a id='H-02'></a>H-02. Hardcoded Minimal_Health_Factor Bug in DSCEngine.sol Contract            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/main/src/DSCEngine.sol#L74

## Summary
The hardcoded Minimal_Health_Factor bug in the DSCEngine.sol contract can cause users to be liquidated even if they have sufficient collateral. This is because the contract hardcodes the Minimal_Health_Factor to 1e18, which is a very high value. This means that even if a user has 200% collateral, they can still be liquidated if the tokens used are USDC or DAI
## Vulnerability Details
The contract expect collateral to be 18 decimals but in discord channel it says that it can interact with any chainlink tokens some of them are 6 decimal like USDT or 2 decimals like DAI
## Impact
The hardcoded Minimal_Health_Factor bug can have a significant impact on users of the DSCEngine.sol contract. The user can be Immediately liquidated even if they have 200% collateral. This can be a significant financial loss, especially if the user has invested a large amount of money into the contract. Additionally, the liquidation process can be slow and expensive, which can further add to the user's losses.
## Tools Used
Manual Review
## Recommendations
Use a more sophisticated liquidation mechanism. The current liquidation mechanism is very simplistic and does not take into account the use of different collateral then WETH/WBTC. A more sophisticated liquidation mechanism would take into account the user's collateral and would only liquidate the user if they were truly insolvent.
		
# Medium Risk Findings

## <a id='M-01'></a>M-01. block.timestamp may be unreliable in short term on L2s            

### Relevant GitHub Links
	
https://github.com/REACH-black-panda/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/libraries/OracleLib.sol#L26-L32

## Summary
The oracle not checking the return timestamp by arbitrum in the `OracleLib.sol` contract can cause the contract to use stale data. This can have a number of implications, including inaccurate or outdated data, loss of funds, and denial of service.
## Vulnerability Details
Arbitrum treats .timestamp differently then EVM
## Impact
The oracle not checking the return timestamp by arbitrum bug can have a significant impact on users of the OracleLib.sol contract. If the contract is using stale data, the results it provides may be inaccurate or outdated. This can lead to problems for users of the contract, such as making bad financial decisions or losing money. Additionally, if the contract fails to execute transactions properly, users may lose funds. Finally, if the contract is unavailable, users may not be able to use it. This can be a problem for users who rely on the contract for important services.
## Tools Used
Manual Review
## Recommendations
The protocol should implement require(timestamp != 0) as shown below
```
        public
        view
        returns (uint80, int256, uint256, uint256, uint80)
    {
        (uint80 roundId, int256 answer, uint256 startedAt, uint256 updatedAt, uint80 answeredInRound) =
            priceFeed.latestRoundData();

        uint256 secondsSince = block.timestamp - updatedAt;
        if (secondsSince > TIMEOUT) revert OracleLib__StalePrice();
        + require(timestamp != 0,"Stale Price");
        return (roundId, answer, startedAt, updatedAt, answeredInRound);
    }

    function getTimeout(AggregatorV3Interface /* chainlinkFeed */ ) public pure returns (uint256) {
        return TIMEOUT;
    }
}
```
## <a id='M-02'></a>M-02. Hardcoded hearthbeat            

### Relevant GitHub Links
	
https://github.com/REACH-black-panda/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/libraries/OracleLib.sol#L26-L32

## Summary
OracleLib has a hardcoded 3h TIMEOUT
## Vulnerability Details
Hardcoded TIMEOUT might be to long for oracles with shorter heartbeat
## Impact
For some price feeds this might be not relevant due to a shorter heartbeat, e.g. https://data.chain.link/ethereum/mainnet/crypto-usd/eth-usd
## Tools Used
Mannual Review
## Recommendations
Change the TIMEOUT of the Heartbeat to match all chainlink tokens
## <a id='M-03'></a>M-03. Lack of sequencer up-time check can lead to stale oracle prices            

### Relevant GitHub Links
	
https://github.com/REACH-black-panda/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/libraries/OracleLib.sol#L26-L32

## Summary
The missing L2 Sequencer bug in the `OracleLib.sol` contract can cause the contract to fail to provide accurate or up-to-date data. This can have a number of implications, including inaccurate or outdated data, loss of funds, and denial of service.
## Vulnerability Details
There is no sequencer up-time check which can lead to stale price
## Impact
The missing L2 Sequencer bug can have a significant impact on users of the `OracleLib.sol` contract. If the contract is using stale data, the results it provides may be inaccurate or outdated. This can lead to problems for users of the contract, such as making bad financial decisions or losing money. Additionally, if the contract fails to execute transactions properly, users may lose funds. Finally, if the contract is unavailable, users may not be able to use it. This can be a problem for users who rely on the contract for important services.
## Tools Used
Manual Review
## Recommendations
Add a check to the `OracleLib` contract to ensure that the L2 sequencer is available before calling the `getPrice function`.




