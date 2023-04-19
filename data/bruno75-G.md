For state variables, `x += y` consumes more gas than `x = x + y`.

Instances 1:
- Code: https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L196
- Mitigation: Use `minted = minted + amount;`

Instance 2:
- Code: https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L295
- Mitigation: Use `challengedAmount = challengedAmount + size;`
