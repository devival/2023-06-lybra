# Original link
https://github.com/code-423n4/2023-06-lybra-findings/issues/920
# Lines of code

https://github.com/code-423n4/2023-06-lybra/blob/7b73ef2fbb542b569e182d9abf79be643ca883ee/contracts/lybra/pools/LybraWbETHVault.sol#L10


# Vulnerability details

## Impact
Detailed description of the impact of this finding.

Executing `getAssetPrice()` will always fail which makes all functions calling it fail as well. This includes `depositEtherToMint` which will not allow to deposit any funds to the contract. 


## Proof of Concept
Provide direct links to all referenced code in GitHub. Add screenshots, logs, or any other relevant proof that illustrates the concept.

In `depositEtherToMint` of `LybraWBETHVault.sol` and `LybraRETHVault.sol`there is the line of code calling `getAssetPrice()`:
```
        _mintPeUSD(msg.sender, msg.sender, mintAmount, getAssetPrice());
```
[LybraWbETHVault.sol#L28](https://github.com/code-423n4/2023-06-lybra/blob/7b73ef2fbb542b569e182d9abf79be643ca883ee/contracts/lybra/pools/LybraWbETHVault.sol#L28) 
[LybraRETHVault.sol#L35](https://github.com/code-423n4/2023-06-lybra/blob/7b73ef2fbb542b569e182d9abf79be643ca883ee/contracts/lybra/pools/LybraRETHVault.sol#L35)
`getAssetPrice()` function calls nonexistent function `exchangeRatio()` in WBETH contract to get the exchange rate of WBETH. This will make the call to always fail.
```
   function getAssetPrice() public override returns (uint256) {
        return (_etherPrice() * IWBETH(address(collateralAsset)).exchangeRatio()) / 1e18;
    }
```
[LybraWbETHVault.sol#L35](https://github.com/code-423n4/2023-06-lybra/blob/7b73ef2fbb542b569e182d9abf79be643ca883ee/contracts/lybra/pools/LybraWbETHVault.sol#L35)

## Tools Used
Manual Review

## Recommended Mitigation Steps
Replace `exchangeRatio()` with `exchangeRate()` in the following instances:
`IWBETH` interface [LybraWbETHVault.sol#L10](https://github.com/code-423n4/2023-06-lybra/blob/7b73ef2fbb542b569e182d9abf79be643ca883ee/contracts/lybra/pools/LybraWbETHVault.sol#L10)

`getAssetPrice()` function [LybraWbETHVault.sol#L35](https://github.com/code-423n4/2023-06-lybra/blob/7b73ef2fbb542b569e182d9abf79be643ca883ee/contracts/lybra/pools/LybraWbETHVault.sol#L35)

`IRETH` interface [LybraRETHVault.sol#L10](https://github.com/code-423n4/2023-06-lybra/blob/7b73ef2fbb542b569e182d9abf79be643ca883ee/contracts/lybra/pools/LybraRETHVault.sol#L10)

`getAssetPrice()` function in [LybraRETHVault.sol#L47](https://github.com/code-423n4/2023-06-lybra/blob/7b73ef2fbb542b569e182d9abf79be643ca883ee/contracts/lybra/pools/LybraRETHVault.sol#L47)


## Assessed type

Error