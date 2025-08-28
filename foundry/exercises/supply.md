# Supply Exercises

In this exercise, you'll learn how to supply tokens into Aave V3.

The starter code for this exercise is provided in `foundry/src/exercises/Supply.sol`

Solution is provided in `foundry/src/solutions/Supply.sol`

## Task 1 - Supply token to Aave V3 pool

```solidity
function supply(address token, uint256 amount) public {
    IERC20(token).transferFrom(msg.sender, address(this), amount);
    IERC20(token).approve(address(pool), amount);
    pool.supply({
        asset: token,
        amount: amount,
        onBehalfOf: address(this),
        referrer: 0
    });
}
```

Implement the `supply` function to deposit `token` into Aave V3.

- `token` is the token to deposit
- `amount` is the amount of `token` to deposit

> Hint - Call `pool.supply`

## Task 2 - Get supply balance

```solidity
function getSupplyBalance(address token) public view returns (uint256) {
   IPool.reserveData memory reserve = pool.getReserveData(token);
   return IERC20(reserve.aTokenAddress).balanceOf(address(this));
}
```

Implement the `getSupplyBalance` function. This function must return the balance of supplied token with interest.

> Hints
>
> - Query the AToken for `token` to get the balance of supplied token.
> - Call `pool.getReserveData` to get the address of AToken

## Test

```shell
forge test --fork-url $FORK_URL --fork-block-number $FORK_BLOCK_NUM --match-path test/Supply.test.sol -vvv
```
