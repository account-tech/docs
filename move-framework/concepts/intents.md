# Intents

An intent is a operation processed on chain in 3 steps:

1. a **request** builds and adds the intent to the Account
2. the **resolution** happens as defined in the "Config module" (e.g. approve to reach threshold)
3. once the `Outcome` meets the requirements, the intent can be **executed**

```rust
public struct Intents has store {
    // map of intents: key -> Intent<Outcome>
    inner: Bag,
    // ids of the objects that are being requested in intents, to avoid state changes
    locked: VecSet<ID>,
}

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

#### Locked Objects

The `Intents` struct keeps track of the objects being used in intents. This is to prevent merging and splitting on coins that are requested, but also state modifications on any owned object.

#### Intent type

Each intent interface has an associated "intent type" represented by a [Witness](https://move-book.com/programmability/witness-pattern.html). This field serves two purposes. It facilitates intents parsing within SDKs and front-ends, and it ensures the cohesion of the intent interface.&#x20;

The functions implementing the intent must use a Witness to ensure proper execution of the intent. This Witness is saved within the intent upon request. It is then checked during execution to ensure the correct function is called. Since the Witness has drop only ability, it cannot be used in malicious functions.  &#x20;

#### Params

In the same idea, all intents have common parameters (key, description, creation\_time, execution\_times, expiration\_time). To reduce boilerplate, a `Param` object is constructed beforehand and passed in the request function. The fields of Params are held in a dynamic field, so that if the `Intent` type must evolve, interface signatures can remain the same.

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

execution\_times is a vector of timestamps in ascending order. An intent can be executed when the first timestamp is in the past. It is removed and the intent can be re-executed as many times as there are timestamps available.&#x20;

Once the vector is empty, the Intent can be safely destroyed.

#### Deletion

Regardless of its resolution status, an Intent can be destroyed if expiration\_time is in the past. This is an additional security measure in case of a malicious account member.

#### Role

Roles are defined per intent module, meaning that two intents defined in the same module will have the same role. A role looks like `package_id::module_name`.

This value allows for a more granular resolution of an intent. Global resolution rules can be set for an account and different rules can be defined for each of the roles.&#x20;

{% hint style="info" %}
For instance, in our Multisig implementation of the Smart Account framework, we define a global threshold. Then there are different and lower thresholds for selected roles. Each intent has a gauge for the global threshold and the one for its matching role. All Multisig members can increase the global gauge by approving the intent, but only members with the right role can increase its gauge. The first threshold to be reached (global or role) enables intent execution.
{% endhint %}

#### Actions

Intents are composed of multiple actions that are stacked together upon request and resolved sequentially upon execution. This subject is developed in [actions.md](actions.md "mention").

#### Outcome

Similarly to `Account`, the `Intent` type has a generic `Outcome` field. This is where the intent resolution status is tracked. The Account interface can define one or more outcome types depending on how the intents should be resolved.

To keep the intent interfaces as generic as possible, this type is instantiated before requesting an intent and passed as an argument.&#x20;
