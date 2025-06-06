# Dependencies

Each Account explicitly declares and migrates to new versions of package dependencies that are authorized to modify their state. This approach ensures that only authorized code can interact with the account.

### Deps

By default, the Account must have `AccountProtocol` and `AccountConfig` as its first dependencies, with the latter being the package that defines the Account type. Optionally, one can add `AccountActions`, which provides built-in actions and intents, and any other packages.

A dependency is defined by a name, a package ID, and a version.

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

Dependencies are tracked in the custom `Deps` field of the Account.

```rust
public struct Deps has copy, drop, store {
    inner: vector<Dep>,
    // can community extensions be added
    unverified_allowed: bool,
}
```

### Extensions

The `Deps` struct includes an `unverified_allowed` flag. By default, an Account can only add verified packages as dependencies. The account.tech team curates a collection of these verified packages in the `Extensions` shared object.

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

By enabling unverified dependencies, account members can add arbitrary packages as dependencies. This allows developers to use their own code to interact with an Account.

### VersionWitness

In Move, there is no native way to determine which package, module, or function is being called. `TxContext` provides very limited information, so we developed a new pattern to track package versions and their addresses.

The "Version Witness" is a Witness that can be instantiated anywhere within a package and guarantees provenance by wrapping the package ID.

```rust
public struct VersionWitness has copy, drop {
    // package id where the witness has been created
    package_addr: address,
}
```

In a package dependent on account.tech, such a module must be defined:

* A Witness is defined for each package version, similar to the UpgradeCap version
* The latest Witness is used in a `current()` package-only function that returns a `VersionWitness`
* This type is then used in certain functions and checked against the Account dependencies

```rust
module my_module::version;

use account_protocol::version_witness::{Self, VersionWitness};

// define a new version struct for each new version of the package
public struct V1() has drop;
// public struct V2() has drop; when upgrading

public(package) fun current(): VersionWitness {
    version_witness::new(V1()) // modify with the new version struct
}
```

With this design, it is guaranteed that the VersionWitness for a given package can only be instantiated within that same package, effectively replicating the Witness pattern at the package level.
