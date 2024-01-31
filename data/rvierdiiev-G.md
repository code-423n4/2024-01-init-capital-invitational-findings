## G-01. Move some event from InitCore to PosManager.
### Description
To decrease bytecode size of `InitCore` contract you can move some events emitting to PosManager.

https://github.com/code-423n4/2024-01-init-capital-invitational/blob/main/contracts/core/InitCore.sol#L165
https://github.com/code-423n4/2024-01-init-capital-invitational/blob/main/contracts/core/InitCore.sol#L217
https://github.com/code-423n4/2024-01-init-capital-invitational/blob/main/contracts/core/InitCore.sol#L230
https://github.com/code-423n4/2024-01-init-capital-invitational/blob/main/contracts/core/InitCore.sol#L245
https://github.com/code-423n4/2024-01-init-capital-invitational/blob/main/contracts/core/InitCore.sol#L267
https://github.com/code-423n4/2024-01-init-capital-invitational/blob/main/contracts/core/InitCore.sol#L285


```soldity
emit CreatePosition(msg.sender, posId, _mode, _viewer);

emit SetPositionMode(_posId, _mode);

emit Collateralize(_posId, _pool, amtColl);

emit Decollateralize(_posId, _pool, _to, amtDecoll);

emit CollateralizeWLp(_posId, _wLp, _tokenId, amtColl);

emit DecollateralizeWLp(_posId, _wLp, _tokenId, _to, amtDecoll);
```

## G-02. Remove check inside `InitCore.callback` function.
### Description
The [check inside `InitCore.callback` function](https://github.com/code-423n4/2024-01-init-capital-invitational/blob/main/contracts/core/InitCore.sol#L519) is not needed, because `InitCore` contract even don't have `coreCallback` function. You can decrease bytecode size by removing the check.

## G-03. Move mode check to PosManager.
### Description
To decrease size of InitCore contract, you can [move 0 mode check](https://github.com/code-423n4/2024-01-init-capital-invitational/blob/main/contracts/core/InitCore.sol#L163) to the PosManager.

## G-04. Remove arrays check.
### Description
`InitCore._validateFlash` function [does arrays check](https://github.com/code-423n4/2024-01-init-capital-invitational/blob/main/contracts/core/InitCore.sol#L573). This line can be removed and there will be no problems with that even if sizes will be different.