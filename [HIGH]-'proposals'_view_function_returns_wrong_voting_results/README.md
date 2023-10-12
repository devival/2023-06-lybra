# Original link
https://github.com/code-423n4/2023-06-lybra-findings/issues/943
# Lines of code

https://github.com/code-423n4/2023-06-lybra/blob/7b73ef2fbb542b569e182d9abf79be643ca883ee/contracts/lybra/governance/LybraGovernance.sol#L120-L122


# Vulnerability details

## Impact
Detailed description of the impact of this finding.

Even though, the votes are calculated correctly, the `proposals` view function returns wrong voting results returning `forVotes` results as `againstVotes` amount. 

This would negatively impact the users understanding of the proposals voting results and lead to confusions.

## Proof of Concept
Provide direct links to all referenced code in GitHub. Add screenshots, logs, or any other relevant proof that illustrates the concept.

The Governance `COUNTING_MODE()` is set Bravo as follows:
```
    function COUNTING_MODE() public override view virtual returns (string memory){
          return "support=bravo&quorum=for,abstain";
    }
``` 
[LybraGovernance.sol#L156](https://github.com/code-423n4/2023-06-lybra/blob/7b73ef2fbb542b569e182d9abf79be643ca883ee/contracts/lybra/governance/LybraGovernance.sol#L156)

This refers to the vote options `0 = Against, 1 = For, 2 = Abstain`, as in GovernorBravo.
[Openzepping#Governor](https://docs.openzeppelin.com/contracts/4.x/api/governance#IGovernor-COUNTING_MODE--), 


The total amount of votes are stored in `proposalData[proposalId].supportVotes[X]`, where X, as abovementioned, can be `0 = Against, 1 = For, 2 = Abstain`:
```
    struct ProposalExtraData {
        mapping(address => Receipt)  receipts;
        mapping(uint8 => uint256) supportVotes;
        uint256 totalVotes;
    }

    mapping (uint256 => ProposalExtraData) public proposalData;
```
[LybraGovernance.sol#L27](https://github.com/code-423n4/2023-06-lybra/blob/7b73ef2fbb542b569e182d9abf79be643ca883ee/contracts/lybra/governance/LybraGovernance.sol#L27)

Finally, this is the `proposals` function which returns `forVotes` as a total amount of votes against, which corresponds to `proposalData[proposalId].supportVotes[0]`:
```
    function proposals(uint256 proposalId) external view returns (uint256 id, address proposer, uint256 eta, uint256 startBlock, uint256 endBlock, uint256 forVotes, uint256 againstVotes, uint256 abstainVotes, bool canceled, bool executed) {
        id = proposalId;
        //...
        
        forVotes =  proposalData[proposalId].supportVotes[0];
        againstVotes =  proposalData[proposalId].supportVotes[1];
        abstainVotes =  proposalData[proposalId].supportVotes[2];
        //...
    }
```
[LybraGovernance.sol#L120-L122](https://github.com/code-423n4/2023-06-lybra/blob/7b73ef2fbb542b569e182d9abf79be643ca883ee/contracts/lybra/governance/LybraGovernance.sol#L120-L122)

## Tools Used
Manual Review

## Recommended Mitigation Steps
In [LybraGovernance.sol#L120-L122](https://github.com/code-423n4/2023-06-lybra/blob/7b73ef2fbb542b569e182d9abf79be643ca883ee/contracts/lybra/governance/LybraGovernance.sol#L120-L122) replace 
```
        forVotes =  proposalData[proposalId].supportVotes[0];
        againstVotes =  proposalData[proposalId].supportVotes[1];
```
with
```
        againstVotes =  proposalData[proposalId].supportVotes[0];
        forVotes =  proposalData[proposalId].supportVotes[1];
```






## Assessed type

Other