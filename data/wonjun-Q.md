[L-1] Missing event for an important operation
Important operations should trigger an event to allow being tracked off-chain.

Instances (1):

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L309


It should emit an event to notify external systems that the cap table has been restructured.


```
event CapTableRestructured(address indexed executor, address[] addressesWiped);

...

function restructureCapTable(address[] calldata helpers, address[] calldata addressesToWipe) public {
    ...

    emit CapTableRestructured(msg.sender, addressesToWipe);
}
```



[L-2] MinterApplied & MinterDenied events are missing an important parameter
The Frankencoin.sol contract has important functions; suggestMinter, denyMinter.
However, the callers of these functions are not published in emits.

Instances (2):

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L89
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L156

Add caller to MinterApplied and MinterDenied events.

```
event MinterApplied(address indexed caller, address indexed minter, uint256 applicationPeriod, uint256 applicationFee, string message);
event MinterDenied(address indexed caller, address indexed minter, string message);
```

Add msg.sender parameter in event-emits in suggestMinter() and denyMinter().

```
    emit MinterApplied(msg.sender, _minter, _applicationPeriod, _applicationFee, _message);
    emit MinterDenied(msg.sender, _minter, _message);
```

[N-1] Spellcheck

Instances (3):

the can -> they can
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L153

if -> is
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L24

creterion -> criterion
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L358

Consider using tools like the VSCode extension 'Code Spell Checker' or similar to help catch spelling errors during development.
