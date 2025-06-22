# Audit Finding: Borrowing Fee Bypass via Low minNetDebt

## Summary

The contract initializes `minNetDebt` to `1800e18` (1800 MUSD), which significantly limits accessibility for average users. Although governance functions exist to lower this minimum, a flaw in the fee computation logic allows users to avoid borrowing fees entirely when the minimum is set below a certain threshold, undermining protocol revenue.

## Finding Description

During initialization, the contract sets the minimum net debt (`minNetDebt`) to 1800 MUSD:

```solidity
function initialize() external initializer {
    __Ownable_init(msg.sender);
    refinancingFeePercentage = 20;
    minNetDebt = 1800e18;
}
```

This high threshold effectively restricts borrowing to large-scale users. Governance has the ability to update this minimum via:

```solidity
function proposeMinNetDebt(uint256 _minNetDebt) external onlyGovernance {
    require(_minNetDebt >= MIN_NET_DEBT_MIN, "Minimum Net Debt must be at least $50.");
    proposedMinNetDebt = _minNetDebt;
    proposedMinNetDebtTime = block.timestamp;
    emit MinNetDebtProposed(proposedMinNetDebt, proposedMinNetDebtTime);
}

function approveMinNetDebt() external onlyGovernance {
    require(block.timestamp >= proposedMinNetDebtTime + 7 days,
        "Must wait at least 7 days before approving a change to Minimum Net Debt");
    require(proposedMinNetDebt >= MIN_NET_DEBT_MIN,
        "Minimum Net Debt must be at least $50.");
    minNetDebt = proposedMinNetDebt;
    emit MinNetDebtChanged(minNetDebt);
}
```

If governance reduces `minNetDebt` to a value between $50 and $200, a user can borrow without incurring any borrowing fees due to rounding errors in Solidity's integer arithmetic.

## Impact Explanation

**Impact: HIGH**

When `minNetDebt` is set to a value below 200 MUSD, the borrowing fee becomes effectively zero due to integer division truncation. As a result, users can bypass fees when opening troves, leading to direct loss of protocol revenue and undermining the fee model designed to support ecosystem incentives (e.g., `pcvContract` funding).

## Likelihood Explanation

**Likelihood: MEDIUM**

Governance has complete control over setting `minNetDebt`. While changes may be intended to increase accessibility, this flexibility creates an opportunity—intentional or accidental—for the protocol to operate with 0% borrowing fees. Without a safeguarded minimum fee, the vulnerability is realistically exploitable.

## Proof of Concept

Assume governance lowers the `minNetDebt` to `100e18`. A user then opens a trove:

```solidity
function openTrove(uint256 _debtAmount, address _upperHint, address _lowerHint)
    external payable override {}
```

Which routes to:

```solidity
_opentrove(msg.sender, msg.sender, _debtAmount, _upperHint, _lowerHint);
```

Borrowing fees are computed inside `_opentrove`:

```solidity
_triggerBorrowingFee(contractsCache.troveManager, contractsCache.musd, _debtAmount);
```

And the trigger calls:

```solidity
function _triggerBorrowingFee(ITroveManager _troveManager, IMUSD _musd, uint256 _amount) internal returns (uint) {
    uint256 fee = _troveManager.getBorrowingFee(_amount);
    _musd.mint(pcvAddress, fee);
    return fee;
}
```

Fee calculation logic:

```solidity
function getBorrowingFee(uint256 _debt) external pure override returns (uint) {
    return (_debt * BORROWING_FEE_FLOOR) / DECIMAL_PRECISION;
}
```

With constants:

```solidity
uint256 public constant DECIMAL_PRECISION = 1e18;
uint256 public constant BORROWING_FEE_FLOOR = ((DECIMAL_PRECISION * 5) / 1000); // = 5e15
```

Example calculation for 100 MUSD:

```
(100e18 * 5e15) / 1e18 = 5e17 wei = 0.5 MUSD
```

Since Solidity does not support decimals, the result is truncated to 0, and no fee is charged. This holds true for any borrowing amount under 200 MUSD.

## Recommendation

In `LiquityBase.sol`, revise the `BORROWING_FEE_FLOOR` constant to avoid fee truncation on small borrowing amounts:

**Current:**
```solidity
uint256 public constant BORROWING_FEE_FLOOR = ((DECIMAL_PRECISION * 5) / 1000); // 0.5%
```

**Proposed:**
```solidity
uint256 public constant BORROWING_FEE_FLOOR = ((DECIMAL_PRECISION * 5) / 10); // 50%
```

This change ensures non-zero fees are collected even on low-value troves. Additionally, revisit logic in `_adjustTrove` and `_refinance` to ensure consistent fee behavior across borrowing flows.
