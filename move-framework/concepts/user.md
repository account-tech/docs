# User

The User object is a key-only object indexing the accounts a user is following or is member of. There can be only one User object per address.

```rust
public struct User has key {
    id: UID,
    // account type to list of accounts that the user has joined
    accounts: VecMap<String, vector<address>>,
}
```

Each `Account` implementation can define functions to add and remove an account address within this object. The key corresponds to the TypeName of the `Config` type, the value is the list of the saved account addresses.

An `Invite` can be created and sent to the owner of a User object that will be able to add the provided address to their list of accounts.

This feature removes the need for off-chain indexing.
