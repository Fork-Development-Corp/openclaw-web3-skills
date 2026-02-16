# OpenClaw Ethereum Skills

A comprehensive collection of skills for Ethereum development, node operations, and privacy-focused blockchain interactions. These skills provide agents with deep expertise in the Ethereum ecosystem, from basic client operations to advanced privacy technologies.

## Skills Overview

### Core Ethereum Skills

**[eth-readonly](./eth-readonly/)** - Read-only Ethereum blockchain queries via RPC and Etherscan APIs. Safe exploration of blocks, transactions, balances, and contracts without any wallet required.

**[eth-node](./eth-node/)** - Manage Ethereum execution client nodes (reth, geth) — start, stop, sync status, peers, logs, configuration. Critical for node operators.

**[foundry](./foundry/)** - Complete Ethereum development toolkit using forge, cast, anvil, and chisel. Build, test, deploy, and debug Solidity contracts with industry-standard tooling.

### Privacy-Focused Skills

**[seismic-foundry](./seismic-foundry/)** - Develop, test, and deploy shielded contracts with sforge and scast. Adds encrypted transaction submission and TEE-aware development to the standard Foundry workflow.

**[seismic-reth](./seismic-reth/)** - Privacy-focused Ethereum node with encrypted transactions, shielded storage, and TEE attestation. Enables truly private blockchain interactions.

## Key Features

- **🔐 Privacy-First:** Seismic skills provide transaction-level privacy and shielded storage
- **🛠 Production-Ready:** Battle-tested tools used by Ethereum developers worldwide
- **🔒 Security-Focused:** Mandatory key management best practices and security warnings
- **🍎 macOS-Optimized:** Primary support for macOS development environments
- **🔄 Comprehensive:** From basic queries to advanced contract deployment and debugging

## Installation & Usage

Each skill includes detailed installation instructions and usage examples. Start with:

1. **[eth-readonly](./eth-readonly/)** - Safe blockchain exploration (no wallet needed)
2. **[foundry](./foundry/)** - Full development toolkit (includes wallet/signing)
3. **[eth-node](./eth-node/)** - Node management and operations

For privacy-focused development, explore the Seismic skills after mastering the core toolkit.

## Security & Best Practices

- **Never expose private keys in commands or logs**
- **Always verify chain ID before transactions** 
- **Use encrypted keystores for key management**
- **Bind RPC endpoints to localhost only**
- **Implement proper gas estimation and limits**

## Contributing

These skills are maintained by the OpenClaw community. Each skill follows the OpenClaw skill specification with proper metadata, user-invocable flags, and dependency declarations.

## License

MIT License - See individual skill directories for specific licensing information.

---

**Tip the maintainers:** apexfork.eth