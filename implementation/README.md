# Implementation

In the following sections, we detail how to implement your own smart account type as well as your own intents and actions, using our macros.

Smart accounts can power (almost) any kind of app. We implemented a Multisig and a DAO version, but it could be anything you can dream of! Think AI agent, shops, gaming accounts, possibilities are endless. account.tech just gives you predefined building primitives that you can leverage in your own Move packages.

In addition to making our new `Account` standard modular (use the Config you want), we also made it easily extensible (add any functionality). We show you how it's done in the following sections.

If you want to use or fork one of the existing implementations, you can find them in our [move-registry](https://github.com/account-tech/move-registry).&#x20;

<figure><img src="../.gitbook/assets/intent_flow.excalidraw.png" alt=""><figcaption></figcaption></figure>
