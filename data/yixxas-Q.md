## QA - Low-01

3% Quorum seems to be too low. Being a qualified user has significant privileges. A qualified user can perpetually veto the addition of a new minter and can also restructure the system during dire situations. Considering that chain delegation is used, it is not difficult to reach a 3% quorum. In fact, there can be many delegates who will reach this threshold.

For example, if we only have 500 votes evenly distributed across 5 users A,B,C,D,E.
Total votes is 9000.

A delegates to B who delegates to C who delegates to D who delegates to E.

In this case, C,D,E all have enough votes from their delegatees. C has 300, D has 400, E has 500. Because votes are stacked cumulatively and the delegate inherits all from delegatee without actually losing any votes of itself, quorum should be set to a higher amount.

8-10% seems a better choice of QUORUM.