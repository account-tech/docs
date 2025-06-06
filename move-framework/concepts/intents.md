# Intents

An intent is an operation processed on-chain in three steps:

1. A **request** builds and adds the intent to the Account
2. The **resolution** happens as defined in the "Config module" (e.g., approve to reach threshold)
3. Once the `Outcome` meets the requirements, the intent can be **executed**

```rust
public struct Intents has store {
    // map of intents: key -> Intent<Outcome>
    inner: Bag,
    // ids of the objects that are being requested in intents, to avoid state changes
    locked: VecSet<ID>,
}
```

The `Intents` struct tracks the objects being used in intents. This prevents merging and splitting operations on coins that are requested, as well as state modifications on any owned object.

The inner bag holds the `Intent`s that may have different Outcome types.

```rust
public struct Intent<Outcome> has store {
    // type of the intent, checked against the witness to ensure correct execution
    type_: TypeName,
    // name of the intent, serves as a key, should be unique
    key: String,
    // what this intent aims to do, for informational purpose
    description: String,
    // address of the account that created the intent
    account: address,
    // address of the user that created the intent
    creator: address,
    // timestamp of the intent creation
    creation_time: u64,
    // proposer can add a timestamp_ms before which the intent can't be executed
    // can be used to schedule actions via a backend
    // recurring intents can be executed at these times
    execution_times: vector<u64>,
    // the intent can be deleted from this timestamp
    expiration_time: u64,
    // role for the intent 
    role: String,
    // heterogenous array of actions to be executed in order
    actions: Bag,
    // Generic struct storing vote related data, depends on the config
    outcome: Outcome
}
```

#### Intent type

Each intent interface has an associated "intent type" represented by a [Witness](https://move-book.com/programmability/witness-pattern.html). This field serves two purposes: it facilitates intent parsing within SDKs and front-ends, and it ensures the cohesion of the intent interface.

The functions implementing the intent must use a Witness to ensure proper execution of the intent. This Witness is saved within the intent upon request and is then checked during execution to ensure the correct function is called. Since the Witness has only the `drop` ability, it cannot be used in malicious functions.

#### Role

Roles are defined per intent module, meaning that two intents defined in the same module will have the same role. A role follows the format `package_id::module_name`.

Some of the roles can be divided by "bucket" or "managed asset". For instance, each Vault has a different role with the form: `package_id::module_name::vault_name`.

This value allows for more granular resolution of an intent. Global resolution rules can be set for an account, and different rules can be defined for each of the roles.

{% hint style="info" %}
For instance, in our Multisig implementation of the Smart Account framework, we define a global threshold. Then there are different and lower thresholds for selected roles. Each intent has a gauge for the global threshold and one for its matching role. All Multisig members can increase the global gauge by approving the intent, but only members with the appropriate role can increase its gauge. The first threshold to be reached (global or role) enables intent execution.
{% endhint %}

#### Params

Similarly, all intents have common parameters (key, description, creation\_time, execution\_times, expiration\_time). To reduce boilerplate, a `Params` object is constructed beforehand and passed to the request function. The fields of Params are held in a dynamic field, so that if the `Intent` type must evolve, interface signatures can remain the same.

```rust
public struct Params has key, store {
    id: UID,
}

public struct ParamsFieldsV1 has copy, drop, store {
    key: String,
    description: String,
    creation_time: u64,
    execution_times: vector<u64>,
    expiration_time: u64,
}
```

#### Recurring intents

`execution_times` is a vector of timestamps in ascending order. An intent can be executed when the first timestamp is in the past. It is removed and the intent can be re-executed as many times as there are timestamps available.

Once the vector is empty, the Intent can be safely destroyed.

#### Deletion

Regardless of its resolution status, an Intent can be destroyed if `expiration_time` is in the past. This is an additional security measure in case of a malicious account member.

#### Actions

Intents are composed of multiple actions that are stacked together upon request and resolved sequentially upon execution. This subject is developed in the following section.

{% content-ref url="actions.md" %}
[actions.md](actions.md)
{% endcontent-ref %}

#### Outcome

Similarly to `Account`, the `Intent` type has a generic `Outcome` field. This is where the intent resolution status is tracked. The Account interface can define one or more outcome types depending on how the intents should be resolved.

To keep the intent interfaces as generic as possible, this type is instantiated before requesting an intent and passed as an argument.
