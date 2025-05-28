# Managed Assets

Unlike owned objects, managed assets are deliberately deposited into an Account and handled with custom logic. Smart Account objects have the same capabilities as standard Sui accounts to receive and transfer objects (via intents), but they provide additional functionality.

The framework defines "managed assets" as objects attached as _dynamic object fields_ to an account, and "managed data" as arbitrary structs that can be associated with assets using _dynamic fields_.

{% hint style="info" %}
Both are differentiated to enable object discovery. We recommend that developers define custom types to use as dynamic (object) field keys to prevent abuse.
{% endhint %}

Using these dynamic (object) field wrappers, developers can define interfaces with custom behavior for specific assets, containers, or other objects.

For example, the framework defines Vaults as managed containers (Bag) where only Coins can be added. It also allows developers to deposit and lock UpgradeCaps with a TimeLock. In this case, upgrade execution must wait for the specified delay. Additionally, the framework provides rules for TreasuryCaps that are not available in the core Sui Framework, such as disabling minting or setting a fixed supply.

With managed assets, developers define how an Account interacts with different object and struct types, providing more control than with owned objects.
