#GAS OPTMIZATION REPORT

## G-1 Use modifier insted of require
its more cheaper use a modifier insted of require itself. (althoug modifiers increase the deployment cost its still worth it )

```
contract Equity.sol line 270
 require(msg.sender == address(zchf), "caller must be zchf");

insted create a modifier

modifier onlyZchf(){
 require(msg.sender == address(zchf), "caller must be zchf");
}
```

