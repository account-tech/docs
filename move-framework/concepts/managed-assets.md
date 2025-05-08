# Managed Assets

As opposed to owned objects, managed assets are deliberately deposited into an Account and handled with a custom logic. Smart Account objects have the same capability as standard Sui accounts to receive and transfer objects (via intents), but they have more power.

The framework defines "managed assets" as objects being attached as _dynamic object fields_ to an account, and "managed data" as arbitrary structs that can be associated to assets using _dynamic fields._

{% hint style="info" %}
Both are differentiated to allow discovery for objects. We recommend developers to define custom types to use as dynamic (object) fields key to prevent abuse.
{% endhint %}

Using these dynamic (object) field wrappers, one can define an interface with custom behavior for a specific asset, container or anything else.

For example, the framework defines Vaults as managed containers (Bag) where only Coins can be added. It also allows developers to deposit and lock UpgradeCaps and set a TimeLock. In this case, the upgrade execution will have to wait for the delay. Finally the framework provides additional rules for TreasuryCaps not available in the core Sui Framework, like disabling minting or setting a fixed supply.

With managed assets, developers define how an Account interacts with different object and struct types, unlike with owned objects.&#x20;
