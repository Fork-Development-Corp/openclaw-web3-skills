# eth-readonly

Read-only Ethereum blockchain queries — blocks, transactions, balances, contracts, logs via RPC and Etherscan APIs.

## Quick Start

```bash
# Set up free public RPC
export ETH_RPC_URL="https://ethereum.publicnode.com"

# Get latest block number (no API key needed)
cast block-number --rpc-url $ETH_RPC_URL

# Check Vitalik's ETH balance  
cast balance vitalik.eth --rpc-url $ETH_RPC_URL
```

## Key Features

- **🔒 Read-only:** No wallet required, no transaction signing, completely safe
- **🌐 Public RPC:** Free endpoints + major providers (Infura, Alchemy, QuickNode)
- **📊 Etherscan integration:** Contract source, transaction history, gas tracker
- **⚡ Instant access:** Start exploring blockchain data immediately

## Requirements

- `cast` (Foundry) or `curl` for JSON-RPC calls
- No wallet or private keys needed - this is read-only!

See [SKILL.md](./SKILL.md) for complete documentation.