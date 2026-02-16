# seismic-reth

Privacy-focused Ethereum node with encrypted transactions, shielded storage, and TEE (Trusted Execution Environment) operations.

## Quick Start

```bash
# Install prerequisites (macOS) 
brew install llvm pkg-config
rustup update

# Build from source
git clone https://github.com/SeismicSystems/seismic-reth
cd seismic-reth && cargo build --release

# Start privacy node
ln -sf target/release/reth ~/bin/seismic-reth
seismic-reth node --http --http.addr 127.0.0.1 --http.api eth,net,web3 &> seismic-reth.log 2>&1 &
```

## Key Features

- **🔐 Encrypted Transactions:** Submit transactions that are only decrypted inside TEE
- **🛡 Shielded Storage:** Private storage areas inaccessible to external observers  
- **🔒 TEE Attestation:** Cryptographic proofs of execution environment integrity
- **⚡ Network Configuration:** Custom privacy networks via `network_params.yaml`

## TEE Requirements

- **Intel Mac:** SGX (Software Guard Extensions) support required
- **Apple Silicon:** Native TEE support on M1/M2/M3 processors

## Attestation Examples

```bash
# Get attestation report
curl -s -X POST http://localhost:8545 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"seismic_getAttestationReport","id":1}' | jq
```

See [SKILL.md](./SKILL.md) for complete documentation.