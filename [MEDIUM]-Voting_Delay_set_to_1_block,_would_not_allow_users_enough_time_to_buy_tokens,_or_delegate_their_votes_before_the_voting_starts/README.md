# Original link
https://github.com/code-423n4/2023-06-lybra-findings/issues/939
# Lines of code

https://github.com/code-423n4/2023-06-lybra/blob/7b73ef2fbb542b569e182d9abf79be643ca883ee/contracts/lybra/governance/LybraGovernance.sol#L147-L149


# Vulnerability details

## Impact
Detailed description of the impact of this finding.

With a voting delay set to 1 block, users would not have enough time to buy tokens, or delegate their votes.

## Proof of Concept
Provide direct links to all referenced code in GitHub. Add screenshots, logs, or any other relevant proof that illustrates the concept.

In `LybraGovernance.sol`, the `clock()` implements block number as a unit of time:
```
    function clock() public override view returns (uint48){
        return SafeCast.toUint48(block.number);
    }
```
[LybraGovernance.sol#L161-L163](https://github.com/code-423n4/2023-06-lybra/blob/7b73ef2fbb542b569e182d9abf79be643ca883ee/contracts/lybra/governance/LybraGovernance.sol#L161-L163)

This means, if `votingDelay` function returns `1`, the delay since the proposal is submitted until voting power is fixed and voting starts equals 1 block. [Openzeppelin#votingDelay](https://docs.openzeppelin.com/contracts/4.x/api/governance#Governor-votingDelay-)


## Tools Used
Manual Review

## Recommended Mitigation Steps
Set `votingDelay` to at least 7200 blocks (~1 day) [Ethereum Average Block Time](https://ycharts.com/indicators/ethereum_average_block_time))



## Assessed type

Timing