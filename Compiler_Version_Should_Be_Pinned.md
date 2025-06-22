
# 🔍 Compiler Version Should Be Pinned to a Specific Version

## 📄 Summary
Smart contract compiler versions should be explicitly pinned to a specific version to ensure consistent and deterministic builds. Using a floating version (e.g., `^0.8.24`) may lead to unexpected behavior due to changes in minor compiler releases.

## 📝 Finding Description
In the smart contract, the Solidity compiler version is set using a caret (`^`) as follows:

```solidity
pragma solidity ^0.8.24;
```

This declaration allows the contract to compile with any version of Solidity from `0.8.24` up to, but not including, `0.9.0`. While this may appear flexible, it can introduce unintended issues if a newer compiler version includes subtle changes or optimizations that alter contract behavior. Pinned versions improve reproducibility and auditability.

## ⚠️ Impact
**Low** – Although this does not directly introduce a vulnerability, it can cause future inconsistencies or compilation errors that are hard to trace.

## 📊 Likelihood
**Medium** – Developers may unintentionally compile with different Solidity versions, especially in multi-developer teams or CI/CD environments.

## 🧪 Proof of Concept
The following compiler version directive is used in the codebase:

```solidity
pragma solidity ^0.8.24;
```

This should be updated to:

```solidity
pragma solidity 0.8.24;
```

## 💡 Recommendation
Update the compiler version directive to use a fixed version instead of a range. This ensures consistency across development, testing, and deployment environments.

```diff
- pragma solidity ^0.8.24;
+ pragma solidity 0.8.24;
```

This change promotes deterministic builds and reduces the risk of future compatibility issues.
