---
name: foundry
description: Ethereum development with Foundry — build, test, deploy, and debug Solidity contracts using forge, cast, anvil, and chisel
user-invocable: true
metadata: {"openclaw":{"requires":{"anyBins":["forge","cast","curl"]},"tipENS":"apexfork.eth"}}
---

# Foundry — Ethereum Development Toolkit

You are a Foundry specialist. You help the user build, test, deploy, and debug Solidity smart contracts using the four Foundry tools: **forge** (build/test/deploy), **cast** (chain interaction), **anvil** (local devnet), and **chisel** (Solidity REPL).

For node administration (start, stop, sync, peers), use the **eth-node** skill.
For lightweight on-chain queries without a full Foundry install, use the **eth-client** skill.

## Pre-Flight Check

Before doing anything else, check whether Foundry is installed:

```bash
command -v forge && forge --version || echo "FOUNDRY_NOT_INSTALLED"
```

If Foundry is **not installed**, do NOT proceed with any other section. Instead, walk the user through the guided installation below. Each step requires explicit user confirmation before executing.

## Guided Installation

**Only run this section if the pre-flight check shows Foundry is not installed.**

Walk the user through each step one at a time. Explain what the command does, then ask for confirmation before running it. Never batch these together.

### Step 1: Download the Foundry installer

Explain to the user: "This downloads an installer script from Paradigm (the Foundry maintainers) at `foundry.paradigm.xyz`. It will add `foundryup` to `~/.foundry/bin/`. The script does not install the tools yet — that happens in the next step."

Ask the user for confirmation, then run:
```bash
curl -L https://foundry.paradigm.xyz | bash
```

### Step 2: Install the Foundry toolchain

Explain to the user: "`foundryup` downloads pre-built binaries for `forge`, `cast`, `anvil`, and `chisel` into `~/.foundry/bin/`. These are the actual tools."

Ask the user for confirmation, then run:
```bash
~/.foundry/bin/foundryup
```

### Step 3: Verify PATH

Check that the tools are reachable:
```bash
~/.foundry/bin/forge --version
~/.foundry/bin/cast --version
```

If the shell can't find `forge` without the full path, add to shell profile:
```bash
echo 'export PATH="$HOME/.foundry/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### Step 4: Confirm

```bash
forge --version
cast --version
anvil --version
chisel --version
```

Only proceed to the rest of this skill's capabilities once all four tools report their versions.

---

## Key Management

This is the canonical key management pattern for all Ethereum skills. Private keys never appear in shell arguments, command history, environment variables, or skill text.

### Import a Key into an Encrypted Keystore

```bash
cast wallet import my-wallet --interactive
```

This prompts for the private key, then a password to encrypt it. The keystore file is saved to `~/.foundry/keystores/my-wallet` using AES-encrypted Ethereum keystore v3 format.

### Use by Name

Every command that signs a transaction uses `--account`:

```bash
cast send ... --account my-wallet
forge script ... --account my-wallet
forge create ... --account my-wallet
```

The password is prompted at runtime. The agent never stores, logs, or passes raw private keys.

### Generate a New Wallet

**Run this directly in your terminal, not through an agent.** The raw private key appears in output and cannot be redacted from conversation history, logs, or provider telemetry.

```bash
cast wallet new
```

Import the key immediately:

```bash
cast wallet import fresh-wallet --interactive
```

### List Saved Keystores

```bash
ls ~/.foundry/keystores/
```

### Derive Address from Keystore

```bash
cast wallet address --account my-wallet
```

### Back Up Your Keystore

`~/.foundry/keystores/` is the single point of failure. If this directory is lost, your keys are gone — there is no recovery without the encrypted keystore file and your password. Back it up to a secure, offline location.

---

## Project Scaffolding

### New Project

```bash
forge init my-project
cd my-project
```

This creates:

```
my-project/
├── foundry.toml          # Project configuration
├── src/                   # Contract source files
│   └── Counter.sol
├── test/                  # Solidity tests
│   └── Counter.t.sol
├── script/                # Deployment scripts
│   └── Counter.s.sol
└── lib/                   # Dependencies (git submodules)
```

### Add Dependencies

```bash
forge install OpenZeppelin/openzeppelin-contracts
forge install transmissions11/solmate
```

Dependencies are added as git submodules in `lib/`. Remappings are managed in `foundry.toml` or `remappings.txt`:

```
@openzeppelin/=lib/openzeppelin-contracts/
@solmate/=lib/solmate/src/
```

### Configuration (foundry.toml)

Key settings:

```toml
[profile.default]
src = "src"
out = "out"
libs = ["lib"]
solc_version = "0.8.28"
optimizer = true
optimizer_runs = 200
via_ir = false
ffi = false

[rpc_endpoints]
local = "http://127.0.0.1:8545"
mainnet = "${ETH_RPC_URL}"
sepolia = "${SEPOLIA_RPC_URL}"
```

---

## Building

```bash
forge build
```

Artifacts go to `out/`. To inspect contract sizes:

```bash
forge build --sizes
```

To force a clean rebuild:

```bash
forge clean && forge build
```

---

## Testing

### Run All Tests

```bash
forge test
```

### Verbosity Levels

```bash
forge test -vv      # Show logs from emit
forge test -vvv     # Show execution traces for failing tests
forge test -vvvv    # Show execution traces for all tests
forge test -vvvvv   # Show everything including setup traces
```

### Run Specific Tests

```bash
forge test --match-test testTransfer
forge test --match-contract CounterTest
forge test --match-path test/Counter.t.sol
```

### Watch Mode

```bash
forge test --watch
```

### Writing Tests

Tests are Solidity contracts that inherit from `forge-std/Test.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {Test, console} from "forge-std/Test.sol";
import {Counter} from "../src/Counter.sol";

contract CounterTest is Test {
    Counter public counter;

    function setUp() public {
        counter = new Counter();
        counter.setNumber(0);
    }

    function test_Increment() public {
        counter.increment();
        assertEq(counter.number(), 1);
    }

    function testFuzz_SetNumber(uint256 x) public {
        counter.setNumber(x);
        assertEq(counter.number(), x);
    }

    function test_RevertWhen_Unauthorized() public {
        vm.prank(address(0xdead));
        vm.expectRevert("not owner");
        counter.setNumber(999);
    }
}
```

### Key Test Cheatcodes

| Cheatcode | Purpose |
|-----------|---------|
| `vm.prank(addr)` | Next call comes from `addr` |
| `vm.startPrank(addr)` | All subsequent calls from `addr` until `stopPrank` |
| `vm.deal(addr, amt)` | Set ETH balance |
| `vm.warp(timestamp)` | Set `block.timestamp` |
| `vm.roll(number)` | Set `block.number` |
| `vm.expectRevert(...)` | Assert next call reverts |
| `vm.expectEmit(...)` | Assert next event emission |
| `vm.snapshot()` / `vm.revertTo(id)` | State snapshots |
| `vm.createFork(rpcUrl)` | Fork a live network in-test |
| `vm.label(addr, "name")` | Label addresses in traces |
| `console.log(...)` | Print to test output (use `-vv`) |

### Gas Reports

```bash
forge test --gas-report
```

### Coverage

```bash
forge coverage
forge coverage --report lcov   # For IDE integration
```

### Fork Testing

Test against live network state:

```bash
forge test --fork-url http://127.0.0.1:8545
forge test --fork-url http://127.0.0.1:8545 --fork-block-number 19000000
```

---

## Local Development Network (Anvil)

### Start a Local Chain

```bash
anvil --silent
```

Defaults to `http://127.0.0.1:8545` with 10 pre-funded accounts (10,000 ETH each). Use `--silent` for cleaner output in production scripts.

### Common Options

```bash
anvil --port 8546                          # Custom port
anvil --accounts 5                         # Fewer test accounts
anvil --block-time 12                      # Auto-mine every 12 seconds
anvil --fork-url http://127.0.0.1:8545     # Fork from live node
anvil --fork-block-number 19000000         # Fork at specific block
anvil --chain-id 31337                     # Custom chain ID
```

### Anvil Control via RPC

```bash
# Mine a block manually
cast rpc anvil_mine --rpc-url http://127.0.0.1:8545

# Set account balance
cast rpc anvil_setBalance 0xADDRESS 0xDE0B6B3A7640000 --rpc-url http://127.0.0.1:8545

# Snapshot and revert
cast rpc evm_snapshot --rpc-url http://127.0.0.1:8545
cast rpc evm_revert 0x1 --rpc-url http://127.0.0.1:8545

# Impersonate an account (useful for fork testing)
cast rpc anvil_impersonateAccount 0xWHALE --rpc-url http://127.0.0.1:8545
```

---

## Solidity REPL (Chisel)

### Start

```bash
chisel
```

### Usage

```
➜ uint256 x = 42;
➜ x * 2
Type: uint256
└ Data: 84
➜ address(this).balance
Type: uint256
└ Data: 0
➜ !source          // Show generated source
➜ !clear           // Reset session
➜ !quit            // Exit
```

Chisel is useful for quickly testing ABI encoding, type conversions, math, and Solidity syntax without creating a project.

---

## Deployment

### Using forge create (Simple)

```bash
forge create src/Counter.sol:Counter \
  --account my-wallet \
  --rpc-url http://127.0.0.1:8545
```

With constructor arguments:

```bash
forge create src/Token.sol:Token \
  --constructor-args "MyToken" "MTK" 1000000 \
  --account my-wallet \
  --rpc-url http://127.0.0.1:8545
```

### Using forge script (Recommended)

Scripts give you full control over multi-step deployments:

```solidity
// script/Deploy.s.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {Script, console} from "forge-std/Script.sol";
import {Counter} from "../src/Counter.sol";

contract DeployScript is Script {
    function run() external {
        vm.startBroadcast();

        Counter counter = new Counter();
        counter.setNumber(42);
        console.log("Counter deployed at:", address(counter));

        vm.stopBroadcast();
    }
}
```

Run it:

```bash
# Dry run (no broadcast)
forge script script/Deploy.s.sol --rpc-url http://127.0.0.1:8545

# Broadcast transactions
forge script script/Deploy.s.sol \
  --rpc-url http://127.0.0.1:8545 \
  --account my-wallet \
  --broadcast
```

### Contract Verification

**Etherscan:**
```bash
forge verify-contract 0xDEPLOYED_ADDRESS src/Counter.sol:Counter \
  --chain-id 1 \
  --etherscan-api-key $ETHERSCAN_KEY
```

**Sourcify (decentralized verification):**
```bash
forge verify-contract 0xDEPLOYED_ADDRESS src/Counter.sol:Counter \
  --chain-id 1 \
  --verifier sourcify
```

---

## Debugging

### Debug a Transaction

```bash
forge debug --debug src/Counter.sol --sig "increment()"
```

Opens an interactive debugger with stepping, memory inspection, and stack traces.

### Debug a Failed Transaction on a Fork

```bash
cast run 0xTXHASH --rpc-url http://127.0.0.1:8545
```

Replays the transaction locally and shows the execution trace.

### Inspect ABI Encoding

```bash
# Encode a function call
cast calldata "transfer(address,uint256)" 0xRECIPIENT 1000

# Decode calldata
cast 4byte-decode 0xCALLDATA

# Decode ABI-encoded output
cast abi-decode "balanceOf(address)(uint256)" 0xOUTPUT
```

### Inspect Storage Layout

```bash
forge inspect src/Counter.sol:Counter storage-layout
```

---

## Cast Quick Reference

Beyond wallet management (above), `cast` handles all chain interaction:

```bash
# Read state
cast block-number --rpc-url http://127.0.0.1:8545
cast balance 0xADDRESS --rpc-url http://127.0.0.1:8545
cast call 0xCONTRACT "totalSupply()" --rpc-url http://127.0.0.1:8545
cast code 0xCONTRACT --rpc-url http://127.0.0.1:8545
cast storage 0xCONTRACT 0 --rpc-url http://127.0.0.1:8545

# Gas price
cast gas-price --rpc-url http://127.0.0.1:8545
cast base-fee --rpc-url http://127.0.0.1:8545

# Transaction info
cast tx 0xTXHASH --rpc-url http://127.0.0.1:8545
cast receipt 0xTXHASH --rpc-url http://127.0.0.1:8545

# ENS resolution
cast resolve-name vitalik.eth --rpc-url http://127.0.0.1:8545
cast lookup-address 0xADDRESS --rpc-url http://127.0.0.1:8545

# Unit conversion
cast to-wei 1.5 ether          # 1500000000000000000
cast from-wei 1000000000 gwei  # 1.0
cast to-hex 255                # 0xff
cast to-dec 0xff               # 255

# Nonce (useful for stuck transactions)
cast nonce 0xADDRESS --rpc-url http://127.0.0.1:8545

# Chain ID
cast chain-id --rpc-url http://127.0.0.1:8545
```

---

## Workflow Summary

| Stage | Command | Notes |
|-------|---------|-------|
| Start local chain | `anvil --silent` | Pre-funded test accounts |
| Create project | `forge init` | Includes starter contract + test |
| Add dependencies | `forge install org/repo` | Git submodules in lib/ |
| Build | `forge build` | Artifacts in out/ |
| Test | `forge test -vvv` | Traces on failure |
| Test coverage | `forge coverage` | Line/branch coverage |
| Gas analysis | `forge test --gas-report` | Per-function gas costs |
| Test against live state | `forge test --fork-url $RPC` | Fork testing |
| Quick experiments | `chisel` | Solidity REPL |
| Deploy locally | `forge script --broadcast` | To anvil |
| Deploy to network | `forge script --broadcast --account my-wallet` | Signs with keystore |
| Verify | `forge verify-contract` | Etherscan or Sourcify |
| Debug | `forge debug` or `cast run` | Interactive debugger |

