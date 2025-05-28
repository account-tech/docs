# User

The User object is a key-only object that indexes the accounts a user is following or is a member of. There can be only one User object per address.

```rust
public struct User has key {
    id: UID,
    // account type to list of accounts that the user has joined
    accounts: VecMap<String, vector<address>>,
}
```

Each `Account` implementation can define functions to add and remove an account address within this object. The key corresponds to the TypeName of the `Config` type, and the value is the list of saved account addresses.

An `Invite` can be created and sent to the owner of a User object, enabling them to add the provided address to their list of accounts.

This feature eliminates the need for off-chain indexing.
