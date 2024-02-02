## L-01 `fillOrder` does not cancel order while it should  

The `fillOrder` function expects to cancel the order at L380 in case  if the collateral amount of the `initPosId` is 0, however, it would not cancel the order since the order struct is saved to memory at L373:
```solidity
File: MarginTradingHook.sol
372:     function fillOrder(uint _orderId) external {
373:         Order memory order = __orders[_orderId];
...
379:         if (IPosManager(POS_MANAGER).getCollAmt(order.initPosId, marginPos.collPool) == 0) {
380:             order.status = OrderStatus.Cancelled; 
381:             return;
382:         }
```
Consider updating L380 to: `__orders[_orderId].status = OrderStatus.Cancelled;`

## L-02 Using of `safeApprove` could lead to DOS
In the `MoeSwapHelper` and `BaseMappingIdHook` contract `safeApprove` function is called in case if current allowance value is less than needed for future calls, however, this function would revert in case if current allowance is >0. So this could lead to potential DOS in future for tokens that has big supply, high number of decimals, and are actively utilized by protocol users.
```solidity 
File: MoeSwapHelper.sol
39:     function _ensureApprove(address _token, uint _amt) internal {
40:         if (IERC20(_token).allowance(address(this), ROUTER) < _amt) {
41:             IERC20(_token).safeApprove(ROUTER, type(uint).max);
42:         }
43:     }
```
Consider implementing the pattern `first approve to 0, then approve to the max` instead of using the `safeApprove` function.