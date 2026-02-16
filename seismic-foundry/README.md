# seismic-foundry

Develop, test, and deploy shielded contracts with sforge and scast — privacy-focused Foundry fork with encrypted transactions and TEE-aware development.

## Quick Start

```bash
# Install prerequisites (macOS)
brew install llvm pkg-config
rustup update

# Build from source
git clone https://github.com/SeismicSystems/seismic-foundry
cd seismic-foundry && cargo build --release

# Use shielded types in contracts
suint256 private_balance;  // Only readable inside TEE
mapping(address => suint256) balances;  // Shielded mapping
```

## Key Features

- **🔐 Transaction Privacy:** Encrypted submissions, no MEV exposure
- **🛡 Shielded Storage:** s-prefixed types (`suint256`, `saddress`, `sbool`) 
- **🔒 TEE Testing:** Simulated TEE environment for development
- **⚡ Full Compatibility:** Drop-in replacement for forge/cast workflows

## Privacy Advantages

| Feature | Standard Foundry | Seismic Foundry |
|---------|------------------|-----------------|
| Transaction visibility | Public calldata | **Encrypted until TEE execution** |
| Storage reads | All slots public | **Shielded slots return encrypted blobs** |
| MEV exposure | Full | **None (encrypted submission)** |

## Requirements

- Rust 1.88+ toolchain
- macOS: `brew install llvm pkg-config`

See [SKILL.md](./SKILL.md) for complete documentation.