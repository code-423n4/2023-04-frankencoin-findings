## Suggestion to modify `>=` to `>`

In the `bid()` function, if the remaining time is less than 30 minutes, increase the time to allow other bidders more opportunities to bid.

```
uint256 earliestEnd = block.timestamp + 30 minutes;
if (earliestEnd >= challenge.end) {//@audit
    // bump remaining time like ebay does when last minute bids come in
    // An attacker trying to postpone the challenge forever must increase the bid by 0.5%
    // every 30 minutes, or double it every three days, making the attack hard to sustain
    // for a prolonged period of time.
    challenge.end = earliestEnd;
}

```

However, the `>=` operator can be replaced with `>`, because when `block.timestamp + 30 minutes == challenge.end`, there is no need to update the value of `challenge.end` again, since `earliestEnd` is already equal to `challenge.end`.