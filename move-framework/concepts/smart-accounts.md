# Smart Accounts

Smart Accounts are represented by shared objects, defined in the `account.move` module. They are generic in terms of:&#x20;

* **Composability** - they can be shared, owned, or wrapped
* **Ownership** - rules can be customized to any specification
* **Intent resolution logic** - types are abstracted for flexibility
* **Extension** - any dependent module can add dynamic fields with assets and data
* **Additional data** - the framework does not require specific fields

These aspects are defined in the module implementing a smart account, which we call the "Config Module".

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

Custom types are used for all Account fields to minimize security risks. Only strictly necessary accessors are defined to reduce potential attack vectors.

The `Account` type has the `store` ability to enable unforeseen use cases. This allows these objects to be wrapped instead of being shared.

`Metadata` wraps a VecMap which can never be borrowed mutably (only created and read) and contains arbitrary (key, value) pairs. `Deps` encapsulates a vector of `Dep` and a flag that will be explained in the [dependencies](dependencies.md) section. `Intents` wraps a Bag (heterogeneous map) of `Intent` and tracks the owned objects being used in intents.

`Config` is a generic type defined in the module implementing the Account type and represents the ownership rules and permissions for this Account type. It can be any data structure.

The framework also provides an interface to add and interact with dynamic fields. This is detailed in the [managed assets](managed-assets.md) section.

Finally, an `Auth` type is provided for authenticating addresses as "authorized members" of the Account. It is a hot potato that is delivered according to the account interface. This struct provides access to privileged operations within a single transaction, such as creating an intent or opening a Vault.
