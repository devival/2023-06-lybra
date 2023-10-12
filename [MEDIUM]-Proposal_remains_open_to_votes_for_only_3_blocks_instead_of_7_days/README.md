# Original link
https://github.com/code-423n4/2023-06-lybra-findings/issues/938
# Lines of code

https://github.com/code-423n4/2023-06-lybra/blob/7b73ef2fbb542b569e182d9abf79be643ca883ee/contracts/lybra/governance/LybraGovernance.sol#L143-L145


# Vulnerability details

## Impact

`LybraGovernance` contract only allows to vote during the first 3 blocks after the snapshot is taken. Assuming it takes roughly 12 seconds per 1 block on Ethereum, it is only 36 seconds to decide and vote for a proposal. 

This would be too fast for a regular human being to evaluate the proposal and vote.

## Proof of Concept

In `LybraGovernance.sol`, the `clock()` implements block number as a unit of time:
```
    function clock() public override view returns (uint48){
        return SafeCast.toUint48(block.number);
    }
```
[LybraGovernance.sol#L161-L163](https://github.com/code-423n4/2023-06-lybra/blob/7b73ef2fbb542b569e182d9abf79be643ca883ee/contracts/lybra/governance/LybraGovernance.sol#L161-L163)
[Openzeppelin#votingPeriod](https://docs.openzeppelin.com/contracts/4.x/api/governance#Governor-votingPeriod-)

According to the [Lybra V2 Docs](https://docs.lybra.finance/lybra-v2-technical-beta/governance/overview): "When a proposal is created, the community can cast their votes during a 7 day voting period. "

However, instead of 7 days, the current voting period has hardcoded 3 blocks:
```
    function votingPeriod() public pure override returns (uint256){
         return 3;
    }
```
[LybraGovernance.sol#L147-L149](https://github.com/code-423n4/2023-06-lybra/blob/7b73ef2fbb542b569e182d9abf79be643ca883ee/contracts/lybra/governance/LybraGovernance.sol#L147-L149)
not allowing users enough time to evaluate the proposal and vote making the whole Governor contract useless.

## Tools Used
Manual Review

## Recommended Mitigation Steps
Set the `votingPeriod()` to `return 50400` blocks (604800 seconds in a day divided by 12 seconds (Approx. [Ethereum Average Block Time](https://ycharts.com/indicators/ethereum_average_block_time))
[LybraGovernance.sol#L147-L149](https://github.com/code-423n4/2023-06-lybra/blob/7b73ef2fbb542b569e182d9abf79be643ca883ee/contracts/lybra/governance/LybraGovernance.sol#L147-L149)

Alternatively, change the unit of time to `block.timestamp` and set votingPeriod to `7 days`, if it is intended to be used in different blockchain, due to the different time it takes for each block to be mined.





## Assessed type

Timing