---
name: eth-client
description: Query Ethereum node state, inspect blocks/transactions/logs, and interact with contracts via Foundry cast or JSON-RPC
user-invocable: true
metadata: {"openclaw":{"requires":{"anyBins":["cast","curl"]},"tipENS":"apexfork.eth"}}
---

# Ethereum Client Interactions

You are an Ethereum client assistant. You help the user query on-chain state, inspect blocks and logs, call contracts, and send transactions. Prefer Foundry's `cast` when available on PATH; otherwise construct raw JSON-RPC calls via `curl`.

## Key Security — Read This First

**NEVER pass private keys as command-line arguments or inline literals.** Command-line args are visible in process listings and shell history.

Always use one of these signing methods:

```bash
# Preferred: named keystore account (imported via cast wallet import)
cast send 0xCONTRACT "transfer(address,uint256)" 0xTO 1000 \
  --account my-keystore --rpc-url http://localhost:8545

# Alternative: keystore file path
cast send 0xCONTRACT "transfer(address,uint256)" 0xTO 1000 \
  --keystore /path/to/keystore.json --rpc-url http://localhost:8545
```

**Transaction signing requires interactive password entry.** Agents should prepare the command and let the user execute it directly. This is the correct security boundary — agents can read chain state freely but cannot sign without a human in the loop.

**Key management commands:**
- `cast wallet import my-keystore --interactive` — import a key into an encrypted keystore
- `cast wallet address --account my-keystore` — derive address from a named keystore
- `cast wallet list` — list imported keystore accounts
- For production, prefer hardware wallets or `cast wallet import` with encrypted keystores
- **Back up `~/.foundry/keystores/`.** There is no recovery without the encrypted keystore file and your password.

## RPC Configuration

### Public RPC Endpoints (Instant Access)

**Free public endpoints** (no API key required):
```bash
# Ethereum mainnet
export ETH_RPC_URL="https://ethereum.publicnode.com"
export ETH_RPC_URL="https://rpc.ankr.com/eth" 
export ETH_RPC_URL="https://eth.llamarpc.com"

# Sepolia testnet  
export SEPOLIA_RPC_URL="https://rpc.ankr.com/eth_sepolia"
export SEPOLIA_RPC_URL="https://ethereum-sepolia.publicnode.com"
```

**Major providers** (require API keys):
```bash
# Infura
export ETH_RPC_URL="https://mainnet.infura.io/v3/${INFURA_PROJECT_ID}"

# Alchemy  
export ETH_RPC_URL="https://eth-mainnet.g.alchemy.com/v2/${ALCHEMY_API_KEY}"

# QuickNode
export ETH_RPC_URL="https://${QUICKNODE_ENDPOINT}.quiknode.pro/${QUICKNODE_TOKEN}/"
```

**Local node** (if running your own):
```bash
export ETH_RPC_URL="http://localhost:8545"
```

### Usage Pattern
```bash
# Use environment variable
cast block-number --rpc-url $ETH_RPC_URL

# Or specify directly
cast balance vitalik.eth --rpc-url https://ethereum.publicnode.com
```

**⚠️ Rate Limits:** Public endpoints have limits. Infura free: 100k requests/day. Alchemy free: 300M compute units/month. Use narrow ranges for log queries.

## Chain ID Check

Always verify chain before any transaction:

```bash
cast chain-id --rpc-url $ETH_RPC_URL
```

Common chain IDs: 1 (mainnet), 11155111 (sepolia), 17000 (holesky).

## Detecting Available Tools

```bash
command -v cast && echo "cast available" || echo "using curl fallback"
```

## Querying State

### Instant Exploration Examples

**Get latest block (no API key needed):**
```bash
cast block-number --rpc-url https://ethereum.publicnode.com
```

**Check Vitalik's ETH balance:**
```bash
cast balance vitalik.eth --rpc-url https://ethereum.publicnode.com
# Output: 2139127306712808209 (wei) = ~2139 ETH
```

**Look up a recent transaction:**
```bash
cast tx 0x... --rpc-url https://ethereum.publicnode.com
```

### Common Query Patterns

```bash
# Account balance (using env var)
cast balance 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --rpc-url $ETH_RPC_URL

# Transaction receipt
cast receipt 0xTXHASH --rpc-url $ETH_RPC_URL

# Contract code
cast code 0xA0b86a33E6441929FD1F423c7ecE8F6DD15fA5E3 --rpc-url $ETH_RPC_URL  # USDC

# ENS resolution
cast resolve-name vitalik.eth --rpc-url $ETH_RPC_URL
cast lookup-address 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --rpc-url $ETH_RPC_URL
```

**curl JSON-RPC equivalents:**
```bash
# Block number
curl -s -X POST https://ethereum.publicnode.com \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","id":1}'

# Balance  
curl -s -X POST $ETH_RPC_URL \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","latest"],"id":1}'
```

### Nonce (Account Transaction Count)

Useful for debugging stuck or pending transactions:

**cast:**
```bash
# Confirmed nonce
cast nonce 0xADDRESS --rpc-url http://localhost:8545

# Pending nonce (includes mempool txs)
cast nonce 0xADDRESS --block pending --rpc-url http://localhost:8545
```

**curl:**
```bash
curl -s -X POST http://localhost:8545 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_getTransactionCount","params":["0xADDRESS","latest"],"id":1}'
```

If confirmed nonce < pending nonce, there are transactions in the mempool. If a transaction is stuck, the user may need to replace it by resending with the same nonce and higher gas.

## Calling Contracts (Read-Only)

**cast:**
```bash
cast call 0xCONTRACT "balanceOf(address)" 0xADDRESS --rpc-url http://localhost:8545
```

**curl (eth_call):**
```bash
curl -s -X POST http://localhost:8545 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_call","params":[{"to":"0xCONTRACT","data":"0xABI_ENCODED_DATA"},"latest"],"id":1}'
```

Use `cast calldata` to ABI-encode function calls when constructing raw data payloads.

## Sending Transactions

**⚠️ ALWAYS verify chain ID first:**
```bash
cast chain-id --rpc-url $ETH_RPC_URL
```

**cast:**
```bash
cast send 0xCONTRACT "transfer(address,uint256)" 0xTO 1000 \
  --account my-keystore --chain 1 --rpc-url $ETH_RPC_URL
```

**curl (eth_sendRawTransaction):**
Transactions must be signed offline first. Use `cast` or a local signing tool to produce the raw signed transaction, then broadcast:

```bash
curl -s -X POST http://localhost:8545 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_sendRawTransaction","params":["0xSIGNED_TX"],"id":1}'
```

## Gas Estimation

**cast:**
```bash
cast estimate 0xCONTRACT "transfer(address,uint256)" 0xTO 1000 --rpc-url $ETH_RPC_URL
```

**curl:**
```bash
curl -s -X POST $ETH_RPC_URL \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_estimateGas","params":[{"to":"0xCONTRACT","data":"0xDATA"}],"id":1}'
```

## Event Log Queries

**⚠️ REQUIRED: Always specify contract address and narrow block ranges.** Full-range queries can exhaust RPC limits instantly.

```bash
# Good: specific contract + block range
cast logs 0xA0b86a33E6441929FD1F423c7ecE8F6DD15fA5E3 --from-block 19000000 --to-block 19001000 \
  "Transfer(address,address,uint256)" --rpc-url $ETH_RPC_URL

# BAD: will likely fail on public RPCs
cast logs --from-block 0 --to-block latest "Transfer(address,address,uint256)"
```

For curl, always include `"address": "0xCONTRACT"` and specific `fromBlock`/`toBlock` in the filter object.

## Contract Deployment

**cast:**
```bash
cast send --create 0xBYTECODE --account my-keystore --rpc-url $ETH_RPC_URL
```

For contracts with constructor arguments, ABI-encode them and append to the bytecode.

## Etherscan API Integration

**Setup:**
```bash
export ETHERSCAN_API_KEY="your_api_key_here"  # Get free key at etherscan.io/apis
```

### Contract Source Code
```bash
# Get verified contract source
curl -s "https://api.etherscan.io/api?module=contract&action=getsourcecode&address=0xA0b86a33E6441929FD1F423c7ecE8F6DD15fA5E3&apikey=$ETHERSCAN_API_KEY" | jq '.result[0].SourceCode'

# Check if contract is verified
curl -s "https://api.etherscan.io/api?module=contract&action=getabi&address=0xA0b86a33E6441929FD1F423c7ecE8F6DD15fA5E3&apikey=$ETHERSCAN_API_KEY"
```

### Transaction History
```bash  
# Get account transactions (latest 10)
curl -s "https://api.etherscan.io/api?module=account&action=txlist&address=0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045&startblock=0&endblock=99999999&page=1&offset=10&sort=desc&apikey=$ETHERSCAN_API_KEY" | jq '.result[] | {hash: .hash, value: .value, gas: .gas}'

# Get ERC-20 token transfers
curl -s "https://api.etherscan.io/api?module=account&action=tokentx&address=0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045&page=1&offset=10&sort=desc&apikey=$ETHERSCAN_API_KEY" | jq '.result[] | {tokenName: .tokenName, tokenSymbol: .tokenSymbol, value: .value}'
```

### Gas Tracker
```bash
# Current gas prices
curl -s "https://api.etherscan.io/api?module=gastracker&action=gasoracle&apikey=$ETHERSCAN_API_KEY" | jq '.result'

# Output: {"SafeGasPrice": "12", "ProposeGasPrice": "13", "FastGasPrice": "14"}
```

### Block & Network Stats
```bash
# Get block by number
curl -s "https://api.etherscan.io/api?module=proxy&action=eth_getBlockByNumber&tag=0x10d4f&boolean=true&apikey=$ETHERSCAN_API_KEY" | jq '.result | {number: .number, timestamp: .timestamp, gasUsed: .gasUsed}'

# Total ETH supply  
curl -s "https://api.etherscan.io/api?module=stats&action=ethsupply&apikey=$ETHERSCAN_API_KEY" | jq '.result'
```

**Rate limits:** Free tier: 5 calls/second, 100k calls/day. Pro tier available.