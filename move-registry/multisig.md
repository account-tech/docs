# Multisig

The Multisig solution is a smart account implementation developed and maintained by the account.tech core team. It leverages the framework's core features including intents and actions, while providing additional functionality for configuration management.

You can find the source code [here](https://github.com/account-tech/move-registry/tree/main/packages/multisig).

### Config

For this specific smart account, we defined a `Multisig` struct wrapping:

* &#x20;`Member`s designed by their address, a weight (by default 1) and a list of Roles
* the global threshold&#x20;
* a vector of `Role`s composed of a name and a matching threshold

This constitutes the rules or ownership of a `Account<Multisig>`.

```rust
public struct Multisig has copy, drop, store {
    // members and associated data
    members: vector<Member>,
    // global threshold
    global: u64,
    // role name with role threshold
    roles: vector<Role>,
}

public struct Member has copy, drop, store {
    addr: address,
    // voting power of the member
    weight: u64,
    // roles that have been attributed
    roles: VecSet<String>,
}

public struct Role has copy, drop, store {
    // role name: package_id::module_name::opt_name
    name: String,
    // threshold for the role
    threshold: u64,
}
```

### Outcome

For the multisig, there is a single outcome type which is `Approval` tracking the resolution state of the intents requested and executed by such an account.&#x20;

```rust
public struct Approvals has copy, drop, store {
    // sum of the weights of members who approved the intent
    total_weight: u64,
    // sum of the weights of members who approved and have the role
    role_weight: u64, 
    // who has approved the intent
    approved: VecSet<address>,
}
```

### Process

**Authentication**

* Any member can create an Auth
* Auth enables intent creation, vault operations, etc.

**Approval Process**

* Members can approve or revoke approvals
* Approvals modify the intent's outcome

**Execution Conditions**

An intent can be executed when either:

* The global threshold is reached
* The role-specific threshold is met (e.g., `WithdrawAndTransferIntent` requires @account\_action::owned\_intents role)

### Intent

The package provides an additional intent to modify the multisig config. Upon execution, the intent replaces the config field of the account by the new one.&#x20;

{% hint style="warning" %}
It basically resets the multisig config with the new values provided.
{% endhint %}

### Fees

A fee module enables the team to collect fees upon multisig creation.
