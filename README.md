# BatchLimitOrderRouter

## Overview

The **BatchLimitOrderRouter** is a smart contract designed to execute batch limit orders for token swaps on decentralized exchanges (DEXs) using the Uniswap V2 router interface. It provides the ability to batch-process multiple limit orders while ensuring conditions like price range, minimum output, and order deadlines are met.

---

## Features

- **Limit Orders**: Execute token swaps with specified price ranges, minimum output amounts, and deadlines.
- **Batch Processing**: Execute multiple orders in a single transaction to save gas.
- **Price Validation**: Ensures the price of the swap is within the user-defined range.
- **Allowance and Balance Check**: Validates if the user has approved and holds enough tokens for the swap.
- **Emergency Rescue**: Allows the owner to recover stuck tokens in the contract.

---

## Contract Details

### Constructor

The constructor initializes the contract with a reference to the Uniswap V2 router:

```solidity
constructor(address _router) {
    router = IUniswapV2Router(_router);
}
```

- `_router`: Address of the Uniswap V2 router contract.

---

### Core Functionality

#### 1. **Order Struct**

The `Order` struct defines the parameters for a limit order:

- `maker`: Address of the order creator.
- `tokenIn`: Address of the token to swap from.
- `tokenOut`: Address of the token to swap to.
- `amountIn`: Amount of `tokenIn` to swap.
- `minAmountOut`: Minimum acceptable amount of `tokenOut`.
- `deadline`: Timestamp after which the order expires.
- `lowPrice`: Minimum acceptable price for the swap.
- `highPrice`: Maximum acceptable price for the swap.

#### 2. **Order Execution**

The `executeOrder` function performs the following steps for an individual order:

- Validates the order parameters (e.g., deadline, allowance, balance).
- Checks if the price is within the defined range using the `getAmountsOut` function of the router.
- Transfers the tokens to the contract.
- Approves the router to spend the tokens.
- Executes the swap using the `swapExactTokensForTokens` function of the router.

#### 3. **Batch Order Execution**

The `executeBatchOrders` function allows batch execution of multiple orders:

```solidity
function executeBatchOrders(Order[] memory orders)
    external
    onlyOwner
    returns (bool[] memory results)
{
    results = new bool[](orders.length);

    for (uint256 i = 0; i < orders.length; i++) {
        results[i] = executeOrder(orders[i]);
    }

    return results;
}
```

- Takes an array of `Order` structs as input.
- Executes each order using the `executeOrder` function.
- Returns a boolean array indicating the success or failure of each order.

#### 4. **Emergency Rescue**

The `rescueTokens` function allows the owner to recover tokens stuck in the contract:

```solidity
function rescueTokens(address token, uint256 amount) external onlyOwner {
    IERC20(token).transfer(owner(), amount);
}
```

---

## Events

### OrderExecuted

Emitted for each order execution, whether successful or not:

```solidity
event OrderExecuted(
    address indexed maker,
    address indexed tokenIn,
    address indexed tokenOut,
    uint256 amountIn,
    uint256 amountOut,
    bool success,
    string reason
);
```

---

## Usage

1. **Deploy the Contract**
   - Deploy the contract by providing the address of the Uniswap V2 router.

2. **Submit Orders**
   - Define an array of `Order` structs with desired parameters.
   - Call the `executeBatchOrders` function to execute the orders.

3. **Handle Emergency**
   - Use the `rescueTokens` function to recover any stuck tokens in the contract.

---

## Dependencies

- [OpenZeppelin Contracts](https://github.com/OpenZeppelin/openzeppelin-contracts): Used for `IERC20` and `Ownable` functionality.

---

## Security Considerations

- Ensure the router address is a valid Uniswap V2 router.
- Only the owner can execute batch orders or rescue tokens.
- Use appropriate gas limits when executing a large batch of orders.

---

## License

This project is licensed under the [MIT License](LICENSE).

---

## Contact

For any queries or contributions, feel free to reach out via GitHub or email.