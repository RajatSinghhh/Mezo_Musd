
# 🔍 Risk of Initialization Attack Due to Delayed Invocation of `initialize()` Function

## 📄 Summary
The `initialize()` function in upgradeable contracts must be executed **immediately** after deploying the proxy. If left uninitialized, malicious actors can exploit this delay to hijack ownership or privileges — a severe vulnerability known as an *Initialization Attack*.

## 📝 Finding Description
In `BorrowerOperations.sol` at line 110, the following `initialize()` function is defined:

```solidity
function initialize() external initializer {
    __Ownable_init(msg.sender);
    refinancingFeePercentage = 20;
    minNetDebt = 1800e18;
}
```

Although the function uses OpenZeppelin’s `initializer` modifier — which ensures it can only be executed once — **it still remains externally callable until it is initialized**. If this function is not invoked immediately upon deployment of the proxy contract, an attacker could call it first and set themselves as the contract owner via `__Ownable_init(msg.sender)`, effectively gaining control of the contract.

This is a textbook case of an *Initialization Attack*, which has occurred in production systems and led to critical asset loss and control takeovers.

## ⚠️ Impact
**High** – Unauthorized access to `initialize()` could lead to full administrative control being granted to an attacker, allowing manipulation of contract parameters or draining of funds.

## 📊 Likelihood
**High** – If initialization is delayed or forgotten during deployment, this vulnerability is trivially exploitable by anyone observing the contract state.

## 🧪 Proof of Concept
The function below is publicly accessible until initialized:

```solidity
function initialize() external initializer {
    __Ownable_init(msg.sender); // attacker becomes the owner
    refinancingFeePercentage = 20;
    minNetDebt = 1800e18;
}
```

If the deployer forgets or delays the initialization step, any actor can take control by calling `initialize()` and setting `msg.sender` as the owner.

## 💡 Recommendation
Ensure `initialize()` is **called immediately after proxy deployment**, before the contract is exposed to the public. Alternatively, consider enforcing role-based access control during initialization.

Additional best practices:
- Automate the initialization in the deployment script to reduce human error.
- Verify that the deployment pipeline does not expose the uninitialized proxy on-chain before calling `initialize()`.

By adhering to these practices, the risk of Initialization Attacks can be effectively mitigated.
