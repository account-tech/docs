# Smart Accounts

Smart Accounts are represented by shared objects, defined in the `account.move` module. They are generic in terms of:&#x20;

* composability (they can be shared, owned or wrapped)
* ownership (rules can be anything)
* intent resolution logic (types are abstracted)
* extension (any dependent module can add dynamic fields with assets and data)
* additional data (the framework does not require specific fields)

These aspects are defined in the module implementing a smart account, we call it the "Config Module".

```rust
public struct Account<Config> has key, store {
    id: UID,
    // arbitrary data that can be proposed and added by members
    // first field is a human readable name to differentiate the multisig accounts
    metadata: Metadata,
    // ids and versions of the packages this account is using
    // idx 0: account_protocol, idx 1: account_actions optionally
    deps: Deps,
    // open intents, key should be a unique descriptive name
    intents: Intents,
    // config can be anything (e.g. Multisig, coin-based DAO, etc.)
    config: Config,
}
```

Custom types are used for all the Account fields to minimize risks. Only strictly needed accessors are defined to reduce vector attacks. &#x20;

The `Account`  type has the store ability to enable unforeseen use cases. This way these objects can be wrapped instead of being shared.&#x20;

`Metadata` wraps a VecMap which can never be borrowed mutably (only created and read) and contains arbitrary (key, value) pairs. `Deps` encapsulates a vector of `Dep` and a flag that will be explained in [dependencies.md](dependencies.md "mention"). `Intents` wraps a Bag (heterogeneous map) of `Intent` and tracks the owned objects being used in intents.

`Config` is a generic type defined in the module implementing the Account type and representing the ownership rules and permissions for this Account type. It can be literally any data struct.

The framework also provide an interface to add and interact with dynamic fields. This is detailed in [managed-assets.md](managed-assets.md "mention").

Last but not least, an Auth type is provided for authenticating addresses as "authorized members" of the Account. It is a hot potato that is delivered according to the account interface. This struct gives access to privileged operations within a single transaction, such as creating an intent or opening a Vault.
