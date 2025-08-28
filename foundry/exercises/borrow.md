# Borrow Exercises

In this exercise, you'll learn how to borrow tokens from Aave V3.

The starter code for this exercise is provided in `foundry/src/exercises/Borrow.sol`

Solution is provided in `foundry/src/solutions/Borrow.sol`

## Task 1 - Approximate the maximum amount of token that can be borrowed

```solidity
function approxMaxBorrow(address token) public view returns (uint256) {
    uint256 price = oracle.getAssetPrice(token);
    uint256 decimals = IERC20Metadata(token).decimals();

    (,, uint256 availableToBorrowUsd,,,) = pool.getUserAccountData(address(this));

    return availableToBorrowUsd * (10 ** decimals) / price;
}
```

Implement the `approxMaxBorrow(` function. This function will approximate the maximum amount of token that you can borrow.

- `token` is the token to borrow

> Hints
>
> - Call `oracle.getAssetPrice` to get the USD price of `token`
> - Call `pool.getUserAccountData` to get the maximum USD amount that can be borrowed from Aave
> - Approximate amount of `token` that can be borrowed is the max USD value that can be borrowed divide by the price of `token`

## Task 2 - Get the health factor of this contract

```solidity
function getHealthFactor() public view returns (uint256) {
    (,,,,, uint256 healthFactor) = pool.getUserAccountData(address(this));
    return healthFactor;
}
```

Get the health factor of this contract

> Hint - Call `pool.getUserAccountData`

## Task 3 - Borrow token from Aave V3

```solidity
function borrow(address token, uint256 amount) public {
    pool.borrow({
        asset: token,
        amount: amount,

        interestRateMode: 2,
        referralCode: 0,
        onBehalfOf: address(this)
    });
}
```

Borrow `token` from Aave V3

- `token` is the token to borrow
- `amount` is the amount of `token` to borrow

> Hint - Call `pool.borrow`

## Task 4 - Get variable debt balance of this contract

```solidity
function getVariableDebt(address token) public view returns (uint256) {
    IPool.ReserveData memory reserve = pool.getReserveData(token);
    return IERC20(reserve.variableDebtTokenAddress).balanceOf(address(this));
}
```

Get variable debt balance of this contract.

> Hints
>
> - Call `pool.getReserveData` to get the address of variable debt token
> - Query the balance of variable debt token for this contract to get the current debt that this contract owes to Aave

## Test

```shell
forge test --fork-url $FORK_URL --fork-block-number $FORK_BLOCK_NUM --match-path test/Borrow.test.sol -vvv
```
