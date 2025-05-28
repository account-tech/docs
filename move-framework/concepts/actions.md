# Actions

Actions represent the concept of an elementary unit of execution on Sui, such as withdrawing an object or transferring it. Multiple actions can be stacked together to compose an intent that will process each one sequentially.

In the diagram below, you can see the processing flow of an intent composed of four actions. The first action withdraws (receives) an object from the account and returns it. The following action uses it as an input and transfers it to a recipient. The two subsequent actions perform the same operation with another object.

<figure><img src="../../.gitbook/assets/intent_composition.excalidraw (1).png" alt=""><figcaption></figcaption></figure>

In Move, this is implemented as simple structs with a unique `store` ability. Each Action defines its own interface for adding it to an intent, executing it, and deleting it.

```rust
public struct WithdrawAction has store {
    // the owned object we want to access
    object_id: ID,
}

public struct TransferAction has store {
    // address to transfer to
    recipient: address,
}
```

Each action type implements three core functions:
- **Add**: Creates and adds the action to an intent
- **Execute**: Performs the action's operation during intent execution
- **Delete**: Cleans up the action when the intent is destroyed
