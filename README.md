# Mezo-Musd Audit – Findings Summary

This document catalogs key vulnerabilities identified in the audit of the protocol’s smart contract system. Detailed individual reports are provided in the same directory, offering code references, reproduction steps, risk evaluations, and remediation guidance.

---

## 📑 Table of Contents

| ID | Finding Title                                                         | Severity<sup>†</sup> | Link                                                 |
| -- | --------------------------------------------------------------------- | -------------------- | ---------------------------------------------------- |
| 1  | Borrowing Fee Bypass via Low `minNetDebt`                             | Medium               | [Report 1](./finding-1-borrowing-fee-bypass.md)      |
| 2  | Compiler Version Should Be Pinned                                     | Low                  | [Report 2](./finding-2-compiler-version-pinning.md)  |
| 3  | `withdrawColl()` Enables Dust Withdrawals Causing Gas Griefing        | High                 | [Report 3](./finding-3-withdrawcoll-gas-griefing.md) |
| 4  | Risk of Initialization Attack Due to Delayed `initialize()` Call      | High                 | [Report 4](./finding-4-delayed-initialize-attack.md) |
| 5  | Internal Function Naming Convention Violations in `checkContract.sol` | Informational        | [Report 5](./finding-5-naming-convention.md)         |
| 6  | Unprotected `initialize()` Function Vulnerable to Re-initialization   | High                 | [Report 6](./finding-6-unprotected-initialize.md)    |

> <sup>†</sup> *Severity levels are preliminary; see individual reports for final impact scoring.*

---

## 1 Borrowing Fee Bypass via Low `minNetDebt`

* **Summary:** By manipulating `minNetDebt`, users can trigger borrowing operations with negligible debt, avoiding meaningful fee deductions.
* 👉 [Full report](./finding-1-borrowing-fee-bypass.md)

## 2 Compiler Version Should Be Pinned

* **Summary:** Floating compiler versions may introduce inconsistencies or unexpected behaviors across environments.
* 👉 [Full report](./finding-2-compiler-version-pinning.md)

## 3 `withdrawColl()` Enables Dust Withdrawals Causing Gas Griefing

* **Summary:** Near-zero collateral withdrawals repeatedly reinsert Troves into the sorted list, degrading performance and opening doors to gas griefing.
* 👉 [Full report](./finding-3-withdrawcoll-gas-griefing.md)

## 4 Risk of Initialization Attack Due to Delayed `initialize()` Call

* **Summary:** Contracts that delay calling `initialize()` are vulnerable to front-running or unauthorized configuration.
* 👉 [Full report](./finding-4-delayed-initialize-attack.md)

## 5 Internal Function Naming Convention Violations in `checkContract.sol`

* **Summary:** Best practices recommend prefixing internal/private functions with an underscore to improve code clarity and maintainability.
* 👉 [Full report](./finding-5-naming-convention.md)

## 6 Unprotected `initialize()` Function Vulnerable to Re-initialization

* **Summary:** Lack of access control on `initialize()` allows anyone to reconfigure the contract after deployment.
* 👉 [Full report](./finding-6-unprotected-initialize.md)

---

### 📂 Directory Structure

```
.
├── finding-1-borrowing-fee-bypass.md
├── finding-2-compiler-version-pinning.md
├── finding-3-withdrawcoll-gas-griefing.md
├── finding-4-delayed-initialize-attack.md
├── finding-5-naming-convention.md
├── finding-6-unprotected-initialize.md
└── README.md   ← (you are here)
```

### ✍️ Contributing

If additional vulnerabilities or architectural flaws are identified, please consider submitting them securely or via pull request.

---

© <?= date('Y') ?> <?= $ownerName ?? "AuditDrop" ?> – All rights reserved.
