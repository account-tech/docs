# Intent and Action

You can define new actions and intents in addition to the ones provided by the framework. The new smart account could provide new features that would need new intents (e.g. an intent to configurate this specific `Account<Config>` object).

You might also want to make certain functions in your own packages executable by smart accounts, such as Multisigs. For this to work, you have to define these functions as intents.

### Action

An action should be unopionated. It represents the smallest unit of an on-chain operation. For instance, in the case of a "Mint" action, we only mint and return the coin, another action will take care of using it.

All actions are implemented in the same way. Let's go through the "Transfer" action module.

After defining the action struct as explained in [this section](../move-framework/concepts/actions.md), there must be three functions.&#x20;

1. `new_` should instantiate the action directly within an Intent
2. `do_` takes an Executable to read the action and execute required operations
3. `delete_` removes the action struct from an Expired hot potato and unpacks it

By requiring an Intent, Executable and Expired, we ensure that an action can only be executed within an intent process.

```rust
/// Action used in combination with other actions (like WithdrawAction) to transfer objects to a recipient.
public struct TransferAction has store {
    // address to transfer to
    recipient: address,
}

/// Creates a TransferAction and adds it to an intent.
public fun new_transfer<Outcome, IW: drop>(
    intent: &mut Intent<Outcome>,
    recipient: address,
    intent_witness: IW,
) {
    intent.add_action(TransferAction { recipient }, intent_witness);
}

/// Processes a TransferAction and transfers an object to a recipient.
public fun do_transfer<Outcome: store, T: key + store, IW: drop>(
    executable: &mut Executable<Outcome>, 
    object: T,
    intent_witness: IW,
) {
    let action: &TransferAction = executable.next_action(intent_witness);
    transfer::public_transfer(object, action.recipient);
}

/// Deletes a TransferAction from an expired intent.
public fun delete_transfer(expired: &mut Expired) {
    let TransferAction { .. } = expired.remove_action();
}
```

### Intent

Intent interfaces necessitate only two functions. The framework also provides macros for facilitated implementation. You can find them [here](https://github.com/account-tech/move-framework/blob/main/packages/protocol/sources/interfaces/intent_interface.move).&#x20;

Actions compose intents. They are stacked and executed sequentially, in the order they have been added. In this example, the object is first withdrawn, then transferred. Notice that we have to loop over the execute function since we need to declare the object type being transferred upfront. This is abstracted in the SDK.

`build_intent!` allows to create, compose and add an intent to an account in one call.

`process_intent!` helps to read the Executable to process the action assuming the action function called matches the next action to be processed in the Executable (here Withdraw then Transfer).&#x20;

```rust
public fun request_withdraw_and_transfer<Config, Outcome: store>(
    auth: Auth,
    account: &mut Account<Config>, 
    params: Params,
    outcome: Outcome,
    object_ids: vector<ID>,
    recipients: vector<address>,
    ctx: &mut TxContext
) {
    account.verify(auth);
    params.assert_single_execution();
    assert!(object_ids.length() == recipients.length(), EObjectsRecipientsNotSameLength);

    intent_interface::build_intent!(
        account,
        params,
        outcome,
        b"".to_string(),
        version::current(),
        WithdrawAndTransferIntent(),
        ctx,
        |intent, iw| object_ids.zip_do!(recipients, |object_id, recipient| {
            owned::new_withdraw(intent, account, object_id, iw);
            acc_transfer::new_transfer(intent, recipient, iw);
        })
    );
}

/// Executes a WithdrawAndTransferIntent, transfers an object owned by the account. Can be looped over.
public fun execute_withdraw_and_transfer<Config, Outcome: store, T: key + store>(
    executable: &mut Executable<Outcome>, 
    account: &mut Account<Config>, 
    receiving: Receiving<T>,
) {
    account.process_intent!(
        executable,
        version::current(),
        WithdrawAndTransferIntent(),
        |executable, iw| {
            let object = owned::do_withdraw(executable, account, receiving, iw);
            acc_transfer::do_transfer(executable, object, iw);
        }
    );
}
```

### Composability

Intents are high level functions that are meant to be called by end users. Their implementation ensure proper execution. In particular, the "Intent Witness" enforce the "execute" function matching the previously called "request" function to be called for executing the intent. Intents are not composable functions as we usually see on Sui.

Composability happens at a lower level, with actions. Any actions can be grouped together to compose an intent. But again, this takes place in Move - not within PTBs - to ensure intended execution.&#x20;

Not all actions are meant to be composed with others though. For instance, we don't see any use case that would require composing an "ConfigDeps" action with another one. In this case, there is no need to define an interface for the action which can be directly processed within the intent interface. Make sure not to forget the function to delete the action struct.

```rust
/// Creates an intent to update the dependencies of the account by directly adding the action
public fun request_config_deps<Config, Outcome: store>(
    auth: Auth,
    account: &mut Account<Config>, 
    params: Params,
    outcome: Outcome,
    extensions: &Extensions,
    names: vector<String>,
    addresses: vector<address>,
    versions: vector<u64>,
    ctx: &mut TxContext
) {
    account.verify(auth);
    params.assert_single_execution();
    
    let deps = deps::new_inner(extensions, account.deps(), names, addresses, versions);

    account.build_intent!(
        params,
        outcome, 
        b"".to_string(),
        version::current(),
        ConfigDepsIntent(),   
        ctx,
        |intent, iw| intent.add_action(ConfigDepsAction { deps }, iw),
    );
}

/// Executes an intent updating the dependencies of the account by mutating the deps field of the account
public fun execute_config_deps<Config, Outcome: store>(
    executable: &mut Executable<Outcome>,
    account: &mut Account<Config>,  
) {
    account.process_intent!(
        executable, 
        version::current(),   
        ConfigDepsIntent(), 
        |executable, iw| {
            let ConfigDepsAction { deps } = executable.next_action<_, ConfigDepsAction, _>(iw);
            *account.deps_mut(version::current()).inner_mut() = *deps;
        }
    ); 
} 

/// Deletes the ConfigDepsAction from an expired intent
public fun delete_config_deps(expired: &mut Expired) {
    let ConfigDepsAction { .. } = expired.remove_action();
}
```
