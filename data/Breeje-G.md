# Gas Optimization

| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [GAS-01] | Use unchecked keyword for few calculations | 4 |
| [GAS-02] | Unnecessary function declared | 1 |

### [GAS-01] Use unchecked keyword for few calculations

There are instances when their is a required check above the calculation which makes it impossible to underflow or overflow. Such calculations should be marked unchecked to save gas. These are the instances which the Bot missed:

*Instance (4):*
```solidity
File: ERC20.sol

132:    _approve(sender, msg.sender, currentAllowance - amount);

156:    _balances[sender] -= amount;
157:    _balances[recipient] += amount;

```

```solidity
File: Frankencoin.sol

286:    _mint(msg.sender, _amount - reserveLeft);

```

### [GAS-02] Unnecessary function declared 

In `Ownable.sol`, there is a `onlyOwner` modifier which is used at many places in `Position.sol`. The modifier makes an unnecessary call to `requireOwner` function which costs more gas. Instead, the one line logic of `requireOwner` should be placed inline in `onlyOwner` modifier to save gas for jumping to the function everytime.

```solidity
File: Ownable.sol

      function requireOwner(address sender) internal view { 
          if (owner != sender) revert NotOwner();
      }

      modifier onlyOwner() {
          requireOwner(msg.sender);
          _;
      }

```