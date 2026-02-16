---
name: seismic-reth
description: Seismic-reth node — encrypted transactions, shielded storage, TEE operations, and installation from source
user-invocable: true
metadata: {"openclaw":{"requires":{"bins":["cargo"]},"tipENS":"apexfork.eth"}}
---

# Seismic-Reth

You are a seismic-reth specialist. Seismic-reth is a privacy-focused fork of reth that adds encrypted transactions, shielded storage areas, and TEE (Trusted Execution Environment) attestation to the Ethereum execution layer.

For standard node administration (start, stop, sync, peers, logs), use the **eth-node** skill.
For read-only Ethereum queries (balances, transactions, contracts), use the **eth-readonly** skill.

## Installation from Source

**Build instructions:**

### Prerequisites (macOS)

```bash
brew install llvm pkg-config
rustup update
```

### Clone and Build

```bash
git clone https://github.com/SeismicSystems/seismic-reth
cd seismic-reth && cargo build --release
```

### Adding to PATH (macOS)

```bash
ln -sf target/release/reth ~/bin/seismic-reth
```

Name the symlink `seismic-reth` to avoid conflicts with standard reth.

## TEE Requirements

TEE functionality depends on Seismic's compatibility with Apple Silicon Secure Enclave. Verify support before production use.

## Starting a Seismic Node

```bash
seismic-reth node --http --http.addr 127.0.0.1 --http.api eth,net,web3 &> seismic-reth.log 2>&1 &
```

## Seismic-Specific Features

### Encrypted/Shielded Transactions

Seismic-reth supports submitting encrypted transactions that are only decrypted inside the TEE. This prevents frontrunning and MEV extraction.

When interacting with seismic-specific transaction types, use the seismic JSON-RPC extensions rather than standard `eth_sendTransaction`.

### Shielded Storage

Contracts can designate private storage areas that are only accessible within the TEE. External observers cannot read shielded storage slots, even by inspecting state tries.

### TEE Attestation and Verification

Seismic-reth provides TEE attestation capabilities to prove nodes are running unmodified code inside a genuine TEE environment.

**⚠️ Refer to seismic-reth documentation for current RPC methods.** TEE attestation APIs may vary by version and implementation.

Basic concept:
- Attestation reports cryptographically prove execution environment integrity
- Remote verification allows clients to validate TEE status
- Check the official Seismic documentation for specific method names and parameters

### Network Configuration

Seismic networks are configured via `network_params.yaml`. This file defines:

- Chain ID and network name
- Genesis block parameters
- Bootnode ENRs for peer discovery
- TEE configuration and requirements

Check the seismic-reth repository for example network parameter files.

**Related skills:** `/eth-node` (start/stop), `/eth-readonly` (query), `/seismic-foundry` (shielded contracts)