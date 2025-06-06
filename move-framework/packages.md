# Packages

### **Extensions**

Since anyone can create modules to extend the protocol, smart accounts have to explicitly declare which package (dependency) is authorized to modify its state. For this reason, there is an `AccountExtensions` package which defines and shares an `Extensions` object tracking allowed packages and their different versions.

Due to the immutable nature of packages, Accounts must also explicitly migrate to the new version of a package they wish to use. So unlike on other chains, malicious code can't be added to a smart account without the account "owners" consent.

Each Account can opt-in to authorize "unverified deps" which are packages that are not listed in the `Extensions` object. This is useful for developers who don't need their dependency to be globally accessible.

Due to its critical role in smart account security and its straightforward implementation, this package will be made immutable.

### **Protocol**

The `AccountProtocol` package is handling the `Account` logic and the intent process. The `Account` object encapsulates 4 custom types managing its dependencies, metadata, intents and config. The latter being a generic type.

The metadata and dependencies can only be handled via an action which is defined in this same package and that must be wrapped into an intent. The intents can only be accessed mutably within the module that defines the Account `Config` type. This ensures that the Account state can't be modified without proper intent resolution.

Dynamic fields can freely be added to the `Account` object by any module. It is recommended to use custom types for the keys to avoid conflicts and enforce permissioned access.

The package provides interface-like macros facilitating the implementation of smart accounts and two modules with sensitive actions:

1. **Config** (intents): Enables the modification of the Account Metadata. It also handles the management of the Account dependencies.
2. **Owned** (action): Manages access to objects owned by the Account, allowing them to be withdrawn through intents. Withdrawn objects can be used in transfers, vestings and anything else.

### **Actions**

The `AccountActions` package defines a collection of actions that corresponds to unit on-chain operations. They can not be used as is but are meant to be used as building blocks to compose intents.

Based on these actions, multiple intents are defined and can be used with any `Account` type:

1. **Empty:** Enables on-chain agreement on a text. No action is executed on chain.
2. **Access Control**: Lock Caps into the Account and borrow them upon intent execution.
3. **Currency**: Allows creators to lock a TreasuryCap and limit its permissions. Members can mint coins and use them in transfers or payments. Mint can be disabled upon lock.
4. **Kiosk**: Handles the creation of a Kiosk, which is a container for NFTs owned by the Account. The Kiosk module can be used to move NFTs between the Account and other Kiosks. NFTs can listed and delisted from the Kiosk and profits can be withdrawn.
5. **Vestings**: Defines APIs for creating vestings for a Coin that are used in other modules like `owned`, `currency` and `vault`.
6. **Transfers**: Defines APIs for transferring objects from an Account. These objects are retrieved by withdrawing them from owned, spending them from vault, minting them from currency, or anything else you want to define.
7. **Vaults**: Allows members to open containers for Coins and assign members to them via roles. Coins held there can be transferred, paid and more using the Spend action.
8. **Package Upgrades**: Secure UpgradeCaps by locking them into the Account and set an optional TimeLock.
