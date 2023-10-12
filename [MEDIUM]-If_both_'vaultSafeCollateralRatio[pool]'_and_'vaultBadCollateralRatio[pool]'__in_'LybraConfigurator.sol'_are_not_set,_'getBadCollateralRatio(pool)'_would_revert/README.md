# Original link
https://github.com/code-423n4/2023-06-lybra-findings/issues/926
# Lines of code

https://github.com/code-423n4/2023-06-lybra/blob/7b73ef2fbb542b569e182d9abf79be643ca883ee/contracts/lybra/configuration/LybraConfigurator.sol#L339
https://github.com/code-423n4/2023-06-lybra/blob/7b73ef2fbb542b569e182d9abf79be643ca883ee/contracts/lybra/pools/base/LybraPeUSDVaultBase.sol#L128


# Vulnerability details

## Impact

The `liquidation` function in all contracts inheriting `LybraPeUSDVaultBase.sol` would revert until both `vaultSafeCollateralRatio[pool]` and `vaultBadCollateralRatio[pool]` are set. Even though, according to the documentation, the default safe and bad collateral ratios should be 160 and 150 respectively.

Contracts affected by `LybraPeUSDVaultBase` abstract: `LybraWbETHVault.sol`, `LybraWstETHVault.sol`, `LybraRETHVault.sol`.

## Proof of Concept

In `getBadCollateralRatio` function in `LybraConfigurator.sol`, if `vaultBadCollateralRatio[pool]` equals `0`, the function would return `vaultSafeCollateralRatio[pool] - 1e19`.
```
    function getBadCollateralRatio(address pool) external view returns(uint256) {
        if(vaultBadCollateralRatio[pool] == 0) return vaultSafeCollateralRatio[pool] - 1e19;
        return vaultBadCollateralRatio[pool];
    }
```
However, if `vaultSafeCollateralRatio[pool]` is not manually set through `setSafeCollateralRatio` function, it will equal `0`, which will make `vaultSafeCollateralRatio[pool] - 1e19` cause a revert due to the possible underflow.

[LybraConfigurator.sol#L339](https://github.com/code-423n4/2023-06-lybra/blob/7b73ef2fbb542b569e182d9abf79be643ca883ee/contracts/lybra/configuration/LybraConfigurator.sol#L339)

As a consequence, `liquidation` function in all contracts inheriting `LybraPeUSDVaultBase.sol` would revert after calling `getBadCollateralRatio`. 

[LybraPeUSDVaultBase.sol#L128](https://github.com/code-423n4/2023-06-lybra/blob/7b73ef2fbb542b569e182d9abf79be643ca883ee/contracts/lybra/pools/base/LybraPeUSDVaultBase.sol#L128)

## Tools Used
Manual Review

## Recommended Mitigation Steps
Replace `vaultSafeCollateralRatio[pool] - 1e19` with `150 * 1e18` in `getBadCollateralRatio` function of `LybraConfigurator.sol`. 
[LybraConfigurator.sol#L339](https://github.com/code-423n4/2023-06-lybra/blob/7b73ef2fbb542b569e182d9abf79be643ca883ee/contracts/lybra/configuration/LybraConfigurator.sol#L339)






## Assessed type

Under/Overflow