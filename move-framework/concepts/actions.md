# Actions

Actions represent the concept of an elementary unit of execution on Sui, such as the action of withdrawing an object or transferring it. Multiple actions can be stacked together to compose an intent that will go over each one sequentially.

In the diagram below, you can see the processing flow of an intent composed of 4 actions. Here the first action withdraws (receives) an object from the account and returns it. The following action uses it as an input and transfers it to a recipient. The 2 actions coming after are doing the same thing with another object.

<figure><img src="../../.gitbook/assets/intent_composition.excalidraw (1).png" alt=""><figcaption></figcaption></figure>

In Move, this is translated as simple structs with a unique `store` ability. Each Action defines its own interface for adding it to an intent, executing and deleting it.

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
