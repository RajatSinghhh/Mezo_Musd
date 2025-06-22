# Mezo-Musd Audit – Findings Summary

This document catalogs key vulnerabilities identified in the audit of the protocol’s smart contract system. Detailed individual reports are provided in the same directory, offering code references, reproduction steps, risk evaluations, and remediation guidance.

---

## 📑 Table of Contents

| ID | Finding Title                                                         | Severity<sup>†</sup> | Link                                                 |
| -- | --------------------------------------------------------------------- | -------------------- | ---------------------------------------------------- |
| 1  | Borrowing Fee Bypass via Low `minNetDebt`                             | Medium               | [Report 1](./Borrow_Fee_Bypass.md)      |
| 2  | Compiler Version Should Be Pinned                                     | Low                  | [Report 2](./Compiler_Version_Should_Be_Pinned.md)  |
| 3  | `withdrawColl()` Enables Dust Withdrawals Causing Gas Griefing        | Medium                 | [Report 3](./Gas_Griefing_Via_Tiny_Collatral.md) |
| 4  | Risk of Initialization Attack Due to Delayed `initialize()` Call      | High                 | [Report 4](./Initialization_Attack_Due_To_Delayed_Call.md)  |
| 5  | Internal Function Naming Convention Violations in `checkContract.sol` | Informational        | [Report 5](./Internal_Function_Naming_Convention.md)         |

> <sup>†</sup> *Severity levels are preliminary; see individual reports for final impact scoring.*

---

## 1 Borrowing Fee Bypass via Low `minNetDebt`

* **Summary:** By manipulating `minNetDebt`, users can trigger borrowing operations with negligible debt, avoiding meaningful fee deductions.
* 👉 [Full report](./Borrow_Fee_Bypass.md)

## 2 Compiler Version Should Be Pinned

* **Summary:** Floating compiler versions may introduce inconsistencies or unexpected behaviors across environments.
* 👉 [Full report](./Compiler_Version_Should_Be_Pinned.md)

## 3 `withdrawColl()` Enables Dust Withdrawals Causing Gas Griefing

* **Summary:** Near-zero collateral withdrawals repeatedly reinsert Troves into the sorted list, degrading performance and opening doors to gas griefing.
* 👉 [Full report](./Gas_Griefing_Via_Tiny_Collatral.md)

## 4 Risk of Initialization Attack Due to Delayed `initialize()` Call

* **Summary:** Contracts that delay calling `initialize()` are vulnerable to front-running or unauthorized configuration.
* 👉 [Full report](./Initialization_Attack_Due_To_Delayed_Call.md)

## 5 Internal Function Naming Convention Violations in `checkContract.sol`

* **Summary:** Best practices recommend prefixing internal/private functions with an underscore to improve code clarity and maintainability.
* 👉 [Full report](./Internal_Function_Naming_Convention.md)

---

### 📂 Directory Structure

```
.
├── Borrow_Fee_Bypass.md
├──Compiler_Version_Should_Be_Pinned.md
├──Gas_Griefing_Via_Tiny_Collatral.md
├──Initialization_Attack_Due_To_Delayed_Call.md
├──Internal_Function_Naming_Convention.md

└── README.md   ← (you are here)
```

### ✍️ Contributing

If additional vulnerabilities or architectural flaws are identified, please consider submitting them securely or via pull request.

---

© <?= date('Y') ?> <?= $ownerName ?? "AuditDrop" ?> – All rights reserved.
