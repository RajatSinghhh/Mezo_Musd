
# 🔍 Naming Convention Best Practice for Internal Functions in `checkContract.sol`

## 📄 Summary
Internal functions in Solidity should follow standard naming conventions to enhance code readability and maintainability. A common best practice is to prefix internal function names with an underscore (`_`).

## 📝 Finding Description
In the file `checkContract.sol`, the function `checkContract(address _account)` is declared as an internal function. However, it does not follow the conventional naming pattern for internal functions. As per widely accepted Solidity style guidelines, internal functions should be prefixed with an underscore (e.g., `_checkContract`) to clearly distinguish them from external/public interface functions and avoid confusion during development and audits.

## ⚠️ Impact
**Informational** – This issue does not affect the functionality of the contract but is important for maintaining code quality and consistency across the codebase.

## 📊 Likelihood
**Informational** – This is a stylistic issue and does not pose a risk by itself.

## 🧪 Proof of Concept
The following screenshot highlights the internal function in question, which is currently named without the standard underscore prefix:

![Screenshot 2025-05-01 154724.png](https://imagedelivery.net/wtv4_V7VzVsxpAFaxzmpbw/11f8fefc-6171-4825-8765-7785c36dab00/public)

## 💡 Recommendation
Update the function name to follow Solidity's naming convention for internal functions by adding an underscore prefix. For example:

```diff
- function checkContract(address _account) internal view returns (bool)
+ function _checkContract(address _account) internal view returns (bool)
```

By adopting this naming convention, the code becomes easier to audit and maintain, and aligns with community standards.
