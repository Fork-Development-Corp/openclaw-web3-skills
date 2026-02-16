# eth-client

Query Ethereum node state, inspect blocks/transactions/logs, and interact with contracts via Foundry cast or JSON-RPC.

## Quick Start

```bash
# Check if cast is available
command -v cast && echo "cast available" || echo "using curl fallback"

# Get block number
cast block-number --rpc-url http://localhost:8545

# Get account balance  
cast balance 0xADDRESS --rpc-url http://localhost:8545
```

## Key Features

- **Security-first:** Mandatory chain-id verification, encrypted keystores only
- **Dual compatibility:** Foundry cast preferred, curl JSON-RPC fallback
- **RPC safety:** Rate limiting warnings for public endpoints
- **Log filtering:** Required address and block range limits

## Requirements

- `cast` (Foundry) or `curl` for JSON-RPC calls
- For transactions: encrypted keystore via `cast wallet import --interactive`

See [SKILL.md](./SKILL.md) for complete documentation.