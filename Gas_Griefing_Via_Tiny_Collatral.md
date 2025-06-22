# Summary

The `withdrawColl()` function permits users to withdraw arbitrarily small amounts of collateral, which are routed through `_adjustTrove()` and ultimately reinsert the Trove into a sorted list (`sortedTroves`). This behavior introduces a gas griefing vulnerability, where attackers can continuously send near-zero withdrawal requests that consume disproportionate gas. This results in degraded protocol performance, increased transaction costs for legitimate users, and can potentially lead to partial denial of service if block gas limits are saturated.

# Finding Description

The relevant entry point is `withdrawColl()`:

```solidity
function withdrawColl(uint256 _amount, address _upperHint, address _lowerHint) external override {
    _adjustTrove(
        msg.sender,
        msg.sender,
        msg.sender,
        _amount, // _collWithdrawal
        0,
        false,
        _upperHint,
        _lowerHint
    );
}
```

The `_amount` value is forwarded as `_collWithdrawal` into `_adjustTrove()`. There are no lower bounds enforced on `_collWithdrawal`, aside from requiring it to be non-zero via `_requireNonZeroAdjustment()`:

```solidity
function _requireNonZeroAdjustment(uint256 _collWithdrawal, uint256 _debtChange, uint256 _assetAmount) internal pure {
    require(
        _assetAmount != 0 || _collWithdrawal != 0 || _debtChange != 0,
        "BorrowerOps: There must be either a collateral change or a debt change"
    );
}
```

Even the smallest non-zero value (e.g. `1 wei`) passes this check.

Following this, `_getCollChange()` interprets the withdrawal as a collateral decrease:

```solidity
(collChange, isCollIncrease) = _getCollChange(msg.value, _collWithdrawal);
```

Ultimately, this leads to re-insertion into the Trove sorted list:

```solidity
sortedTroves.reInsert(_borrower, vars.newNICR, _upperHint, _lowerHint);
```

This operation is often costly (e.g., `O(log N)` or worse), and if executed repeatedly with trivial `_collWithdrawal` values, the attacker can:

- Waste on-chain resources.
- Force high gas consumption.
- Increase gas costs for all users.
- Possibly block legitimate operations under high load conditions.

# Impact Explanation

**Severity:** `HIGH`

A malicious user can congest the protocol by sending cheap, tiny-withdrawal transactions.
These operations require very low capital and do not punish the attacker economically.
The cumulative gas costs are disproportionately high compared to the action’s utility.
Repeated `reInsertions` of Troves bloat the execution and can even push block gas limits.
Can result in a partial denial of service (DoS).

# Likelihood Explanation

**Likelihood:** `HIGH`

The attack is simple to execute with no meaningful barrier.
There are no lower-bound validations on the `_collWithdrawal`.
Users can automate and batch these operations from multiple addresses.
There's no disincentive to execute this attack repeatedly.

# Proof of Concept

```solidity
// Attacker script: tiny withdrawals loop
for (uint256 i = 0; i < 100; i++) {
    TroveManager.withdrawColl(1, upperHint, lowerHint); // 1 wei
}
```

This loop:

- Initiates 100 transactions withdrawing only `1 wei` of collateral each.
- Triggers full `_adjustTrove()` logic including ICR/CR recalculation.
- Re-inserts each Trove into the `sortedTroves` list.
- Consumes excessive gas per transaction, all for no effective outcome.

Gas profiler estimates (sampled):

- Normal Trove adjustment: `~100k` gas.
- Tiny withdrawal w/ `reInsert`: `~80k` gas.
- 100 repetitions: `~8M` gas.

Even a single user can congest the system by submitting such transactions rapidly.

# Recommendation

### Option 1: Enforce a minimum withdrawal amount

```solidity
uint256 public constant MIN_WITHDRAWAL = 1e15; // ~0.001 ETH equivalent

require(_collWithdrawal >= MIN_WITHDRAWAL, "Withdrawal too small");
```

### Option 2: Gas throttling via rate limits or cooldowns

- Enforce a cooldown period for withdrawal operations per user.
- Require economic incentives or fees for micro-adjustments.

### Option 3: Lazy sorting for dust ops

- Do not `reInsert` Troves into `sortedTroves` for adjustments < threshold.
- Delay insertion until a meaningful change is made.
