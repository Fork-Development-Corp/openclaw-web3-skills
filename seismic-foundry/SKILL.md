---
name: seismic-foundry
description: Seismic Foundry — develop, test, and deploy shielded contracts with sforge and scast
user-invocable: true
metadata: {"openclaw":{"requires":{"anyBins":["sforge","scast","cargo"]},"tipENS":"apexfork.eth"}}
---

# Seismic Foundry

You are a Seismic Foundry specialist. Seismic Foundry is a privacy-focused fork of Foundry that adds encrypted transaction submission, shielded storage access, and TEE-aware contract development to the standard Forge/Cast workflow.

For read-only Ethereum queries (balances, transactions, contracts), use the **eth-readonly** skill.
For node administration (start, stop, sync, peers), use the **eth-node** skill.
For seismic-reth node setup and TEE operations, use the **seismic-reth** skill.

## Installation from Source

### Prerequisites

**macOS:**
```bash
brew install llvm pkg-config
rustup update
```

**General:**
- **Rust 1.88+** (MSRV) — install or update via `rustup update`
- **Cargo** — included with Rust

### Clone and Build
```bash
git clone https://github.com/SeismicSystems/seismic-foundry
cd seismic-foundry && cargo build --release
```

The build produces `scast` and `sforge` in `seismic-foundry/target/release/`.

### Adding to PATH

macOS:
```bash
ln -sf "$(pwd)/target/release/scast" /usr/local/bin/scast
ln -sf "$(pwd)/target/release/sforge" /usr/local/bin/sforge
```

Linux (user-local, no sudo):
```bash
mkdir -p ~/.local/bin
ln -sf "$(pwd)/target/release/scast" ~/.local/bin/scast
ln -sf "$(pwd)/target/release/sforge" ~/.local/bin/sforge
```

## scast — Shielded Cast

`scast` mirrors `cast` but routes transactions through Seismic's encrypted submission path. Observers cannot see calldata or value until execution inside the TEE.

### Sending a Shielded Transaction
```bash
scast send 0xCONTRACT "transfer(address,uint256)" 0xTO 1000 \
  --account my-keystore --rpc-url http://localhost:8545
```

### Reading Shielded Storage

Standard `eth_getStorageAt` returns encrypted blobs for shielded slots. Use `scast` to decrypt within a TEE context:
```bash
scast storage 0xCONTRACT --slot 0 --rpc-url http://localhost:8545
```

### Querying Standard State

For non-shielded reads, `scast` behaves identically to `cast`:
```bash
scast block-number --rpc-url http://localhost:8545
scast balance 0xADDRESS --rpc-url http://localhost:8545
```

## sforge — Shielded Forge

`sforge` extends `forge` with Seismic's privacy primitives for contract development.

### Shielded Storage in Contracts

Seismic uses **s-prefixed types** (`suint256`, `saddress`, `sbool`, etc.) for storage variables that are only readable inside the TEE. These use `CLOAD`/`CSTORE` opcodes instead of `SLOAD`/`SSTORE`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract PrivateBalance {
    suint256 totalDeposited;           // shielded — not readable outside TEE
    mapping(address => suint256) balances;  // shielded mapping

    function deposit() external payable {
        balances[msg.sender] += suint256(msg.value);
        totalDeposited += suint256(msg.value);
    }

    function getMyBalance() external view returns (suint256) {
        return balances[msg.sender];
    }
}
```

**Constraints on shielded types:**
- Cannot be declared `public` (prevents accidental exposure via auto-generated getters)
- Cannot be `constant` or `immutable` (values would be visible in bytecode)
- Cannot be emitted in events (would leak confidential data on-chain)
- Each shielded variable consumes a full storage slot
- Uses `CSTORE`/`CLOAD` opcodes instead of `SSTORE`/`SLOAD`; cross-writes are rejected at runtime
- **Gas cost:** `CSTORE`/`CLOAD` operations cost more gas than standard storage due to encryption overhead

See the [Seismic Solidity docs](https://docs.seismic.systems) for the latest type reference.

### Building and Testing
```bash
sforge build
sforge test
```

Tests run in a simulated TEE environment so shielded storage behaves correctly during development.

### Deploying a Shielded Contract
```bash
sforge create src/PrivateBalance.sol:PrivateBalance \
  --account my-keystore --rpc-url http://localhost:8545
```

### Script-Based Deployment
```bash
sforge script script/Deploy.s.sol --broadcast \
  --account my-keystore --rpc-url http://localhost:8545
```

## 🔐 Privacy Advantages vs Standard Foundry

**Key differentiators:** Seismic Foundry provides true transaction privacy and shielded storage that standard Ethereum cannot offer.

| Feature | forge/cast | sforge/scast | Privacy Benefit |
|---------|-----------|--------------|-----------------|
| Transaction visibility | Public calldata | **Encrypted until TEE execution** | No frontrunning, MEV protection |
| Storage reads | All slots public | **Shielded slots return encrypted blobs** | Private balances, hidden state |
| Testing environment | Standard EVM | **Simulated TEE with shielded storage** | Test privacy locally |
| Contract types | Standard Solidity | **Adds s-prefixed types** (`suint256`, `saddress`, `sbool`) | Type-safe confidential data |
| MEV exposure | Full | **None (encrypted submission)** | Protected from sandwich attacks |

**Related skills:** `/eth-node` (start/stop), `/eth-readonly` (query), `/seismic-reth` (TEE operations)