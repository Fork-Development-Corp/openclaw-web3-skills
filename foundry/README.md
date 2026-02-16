# foundry

Ethereum development with Foundry — build, test, deploy, and debug Solidity contracts using forge, cast, anvil, and chisel.

## Quick Start

```bash
# Install Foundry
curl -L https://foundry.paradigm.xyz | bash
foundryup
echo 'export PATH="$HOME/.foundry/bin:$PATH"' >> ~/.zshrc

# Create new project
forge init my-project && cd my-project

# Build and test
forge build
forge test -vvv

# Start local network
anvil --silent
```

## Key Features

- **Complete toolkit:** forge (build/test), cast (interact), anvil (local chain), chisel (REPL)
- **Secure key management:** Encrypted keystores, never expose private keys
- **Advanced testing:** Fuzzing, forking, gas reports, coverage analysis
- **Production deployment:** Script-based deployment, contract verification

## Requirements

- Rust toolchain (installed automatically via foundryup)
- Git (for dependencies)

See [SKILL.md](./SKILL.md) for complete documentation including guided installation.