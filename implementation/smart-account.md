# Smart Account

To write the interface of a new smart account type, you need multiple elements that we define here. A smart account type is first defined by its "config" which represents its ownership and intent resolution rules. Note: the macros presented here are located in [account_interface.move](https://github.com/account-tech/move-framework/blob/main/packages/protocol/sources/interfaces/account_interface.move).

### Types

`ConfigWitness`

This is a simple Witness (drop-only struct) that is used to ensure certain functions are called only from the module defining the account `Config`. For example, accessing the fields of an account mutably can be done only from the module defining its config. You can find the "config-only" functions [here](https://github.com/account-tech/move-framework/blob/6d0f47384ec417bbeb52ffdabc201419a38b3002/packages/protocol/sources/account.move#L308).

`Config`

This is a generic type defining the configuration—or rules—of an `Account`. It can be any struct.

```rust
public struct Multisig has copy, drop, store {
    // members and associated data
    members: vector<Member>,
    // global threshold
    global: u64,
    // role name with role threshold
    roles: vector<Role>,
}
```

`Outcome`

Another generic type, representing the current resolution state of an intent. It can also be any struct. A single `Account<Config>` implementation can have a multitude of outcome types thanks to the intent Bag!

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

### Functions

`new_account`

To create a new account with the config you defined, you can use this macro. You pass the instantiated config, a `VersionWitness`, a `ConfigWitness`, and a way to initialize the dependencies.

```rust
public macro fun create_account<$Config, $CW: drop>(
    $config: $Config,
    $version_witness: VersionWitness,
    $config_witness: $CW,
    $ctx: &mut TxContext,
    $init_deps: || -> Deps,
): Account<$Config>
```

`authenticate`

This function is used to give admin privileges to the transaction sender. Several functions require an Auth to be called, such as the ones to call commands and request intents in the `AccountActions` package.

You can use any function to grant permission. In the Multisig, we assert that the caller is a member of the Account, for example.

```rust
public macro fun create_auth<$Config, $CW: drop>(
    $account: &Account<$Config>,
    $version_witness: VersionWitness,
    $config_witness: $CW,
    $grant_permission: ||,
): Auth
```

`empty_outcome`

Define functions to create and return the outcome types you defined. These types are passed in the request intent functions. This way we keep the intent interfaces as generic as possible.

`resolve_intent`

This macro is used to mutably access the outcome of an intent. Just like the other macros, it can only be called within the "Config module" which must be a dependency of the Account.

```rust
public macro fun resolve_intent<$Config, $Outcome, $CW: drop>(
    $account: &mut Account<$Config>,
    $key: String,
    $version_witness: VersionWitness,
    $config_witness: $CW,
    $modify_outcome: |&mut $Outcome|,
)
```

`execute_intent`

After validating the outcome of an intent, this macro returns an Executable hot potato wrapping the intent and enforcing proper execution.

```rust
public macro fun execute_intent<$Config, $Outcome, $CW: drop>(
    $account: &mut Account<$Config>,
    $key: String,
    $clock: &Clock,
    $version_witness: VersionWitness,
    $config_witness: $CW,
    $validate_outcome: |$Outcome|,
): Executable<$Outcome>
```

`join` `leave` `send_invite`

Additionally, you can provide ways for bearers of a `User` object to add or remove an account address to their object. On-chain `Invite`s can also be sent to facilitate discovery (e.g., Invites are sent to newly added members in a Multisig).

{% hint style="success" %}
Use our [multisig](https://github.com/account-tech/move-registry/blob/main/packages/multisig/sources/multisig.move) and [dao](https://github.com/account-tech/move-registry/blob/main/packages/dao/sources/dao.move) implementations as references!
{% endhint %}
