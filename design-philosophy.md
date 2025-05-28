---
description: >-
  We are committed to building products aligned with the original Web3 ethos we
  cherish: transparency, accessibility and collaboration. Our stack reflects
  these principles and more on the technical side
---

# Design Philosophy

### **Modularity**

Our smart accounts are designed as shared objects with a unified interface that handles core logic and manages the intent creation and execution flow. The system's strength lies in the flexibility of account rules and intent resolution, which can be configured to meet specific requirements.

Configurations are versatile structures that can be customized to fit any requirement. By default, we provide Multisig configuration as well as coin-based and NFT-based DAO configurations. These preset options define specific rules and parameters such as thresholds, roles, members, and weights for Multisig accounts, or quorum requirements, voting rules, and governing assets for DAOs.

The true power of our system lies in its adaptability – any type of configuration with custom rules can be implemented without requiring a core protocol upgrade.

### **Compatibility**

Smart Accounts seamlessly interact with owned objects, mirroring the functionality of standard Sui accounts. Account.tech extends this capability by providing custom interfaces for effortless interaction with standard Sui objects and straightforward integration with custom objects. Our roadmap includes comprehensive support for all known objects on Sui, along with extended capabilities for enhanced functionality.

Beyond basic compatibility, our interfaces introduce essential policies missing from Sui's core framework. These include critical security features such as upgrade time locks and currency supply controls, providing built-in protection that traditionally requires custom development.

Front-end applications leveraging our protocol, such as our Multisig platform, can utilize the Move interfaces provided for each object type. This enables the creation of user-friendly experiences that abstract away the complexities of managing multiple objects from different packages.

### **Extensibility**

Account.tech serves as fundamental security infrastructure for the Sui ecosystem, built on principles of modularity and extensibility that enable seamless integration at any level of complexity.

On-chain organizations can immediately secure their assets and operations through our ready-to-use applications such as Multisig and DAO platforms, requiring zero additional code. For development teams, our SDK provides programmatic access to these capabilities, allowing them to extend functionality within their applications and backends without direct blockchain interaction.

Advanced users can deeply integrate with our protocol by embedding custom actions and intents directly into their smart contracts through our comprehensive interfaces. This flexibility also enables innovators to create entirely new configurations and build secure, intuitive platforms on top of our infrastructure while maintaining enterprise-grade security standards.

### **Trustless**

Smart Accounts explicitly declare and manage package dependencies authorized to modify their state. This approach ensures a trustless environment by preventing the execution of unauthorized or potentially malicious code without explicit consent from account members.
