# Dependencies

Each Account explicitly declares and migrates to new versions of package dependencies which are allowed to modify their state. This approach ensures only authorized code can interact with it.

### Deps

By default, the Account must have `AccountProtocol` and `AccountConfig` as first dependencies. The latter being the package defining the Account type. Optionally, one can add `AccountActions` which provides built-in actions and intents, and any other package.

A dependency is defined by a name, a package id and a version.

```rust
public struct Dep has copy, drop, store {
    // name of the package
    name: String,
    // id of the package
    addr: address,
    // version of the package
    version: u64,
}
```

They are tracked in the custom `Deps` field of the Account.

```rust
public struct Deps has copy, drop, store {
    inner: vector<Dep>,
    // can community extensions be added
    unverified_allowed: bool,
}
```

### Extensions

In this struct, there is an `unverified_allowed` flag. This is because, by default, an Account can only add verified packages as dependencies. The account.tech team curates a collection of these in the `Extensions` shared object.

```rust
/// A list of verified and whitelisted packages
public struct Extensions has key {
    id: UID,
    inner: vector<Extension>,
}

/// A package with a name and all authorized versions
public struct Extension has copy, drop, store {
    name: String,
    history: vector<History>,
}

/// The address and version of a package
public struct History has copy, drop, store {
    addr: address,
    version: u64,
}
```

By allowing unverified dependencies, members of an account can add arbitrary packages as dependencies. This allows developers to use their own code to interact with an Account.
