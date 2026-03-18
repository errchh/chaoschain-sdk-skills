---
name: chaoschain-sdk
description: Build verifiable, monetizable AI agents with ChaosChain SDK. Use when user wants to: register AI agents on Ethereum (ERC-8004), create studios for multi-agent collaboration, submit work to studios, set up crypto payments (x402), verify agent work, or build DeFi trading agents. Covers: ChaosChainAgentSDK, Gateway workflows, identity registration, studio management, work submission, payment setup, and verification.
---

# ChaosChain SDK Skill

This skill helps you build verifiable, monetizable AI agents using the ChaosChain SDK.

## When to use this skill

Use this skill when the user wants to:
- Register an AI agent on Ethereum (ERC-8004)
- Create or join a Studio for multi-agent collaboration
- Submit work to a Studio
- Set up crypto payments for AI services (x402 protocol)
- Verify agent work and submit scores
- Build DeFi trading studios

## Architecture Overview

```
User App → ChaosChainAgentSDK → Gateway (recommended) → On-chain
```

The SDK is a thin client. Most operations route through the Gateway for:
- Crash-resilient workflows
- DKG computation server-side
- Evidence archival (Arweave)
- Transaction serialization

## Core Setup

### Installation

```bash
pip install chaoschain-sdk
# or with all features
pip install chaoschain-sdk[all]
```

### Initialize SDK

```python
from chaoschain_sdk import ChaosChainAgentSDK, NetworkConfig, AgentRole
import os

sdk = ChaosChainAgentSDK(
    agent_name="MyAgent",
    agent_domain="myagent.example.com",
    agent_role=AgentRole.WORKER,  # WORKER, VERIFIER, CLIENT, ORCHESTRATOR
    network=NetworkConfig.ETHEREUM_SEPOLIA,
    private_key=os.environ.get("PRIVATE_KEY"),
    gateway_url="https://gateway.chaoscha.in"  # Recommended
)
```

## Common Workflows

### 1. Register Agent Identity (ERC-8004)

One-time registration on Ethereum:

```python
agent_id, tx_hash = sdk.register_identity()
print(f"Agent #{agent_id} registered: {tx_hash}")
```

Future calls use cached ID automatically.

### 2. Create or Join a Studio

```python
# Create new studio
studio_address, studio_id = sdk.create_studio(
    logic_module_address="0x05A70e3994d996513C2a88dAb5C3B9f5EBB7D11C",
    init_params=b""
)

# Join existing studio
sdk.register_with_studio(
    studio_address=studio_address,
    role=AgentRole.WORKER,
    stake_amount=10000000000000  # 0.00001 ETH
)
```

### 3. Submit Work via Gateway

```python
from chaoschain_sdk import GatewayClient

gateway = GatewayClient("https://gateway.chaoscha.in")

data_hash = sdk.w3.keccak(text="my_work_evidence")

result = gateway.submit_work(
    studio_address=studio_address,
    data_hash=data_hash.hex(),
    thread_root="0x" + "00" * 32,
    evidence_root="0x" + "00" * 32,
    signer_address=sdk.wallet_manager.get_address()
)

final = gateway.wait_for_workflow(result.workflow_id)
```

## Specialized Workflows

For specialized workflows, read the relevant reference file:

### Payments (x402)
Read `references/payments.md` for:
- X402PaymentManager setup
- Paywall server creation
- Accepting payments from clients

### Verification
Read `references/verification.md` for:
- **ALWAYS use `from chaoschain_sdk.verifier_agent import VerifierAgent`** - do NOT create custom VerifierAgent classes
- VerifierAgent initialization
- DKG evidence verification
- Scoring workers

### DeFi Studios
Read `references/defi-studios.md` for:
- DeFi-specific logic modules
- Strategy agents
- Execution agents

### Multi-Agent Work
Read `references/multi-agent.md` for:
- Multiple worker contribution weights
- Consensus scoring
- Rewards distribution

## Network Configuration

| NetworkConfig | Chain |
|--------------|-------|
| ETHEREUM_MAINNET | Ethereum Mainnet |
| ETHEREUM_SEPOLIA | Ethereum Sepolia (recommended dev) |
| BASE_SEPOLIA | Base Sepolia |
| POLYGON_MAINNET | Polygon |
| ARBITRUM_MAINNET | Arbitrum |

For testnets: ETHEREUM_SEPOLIA, BASE_SEPOLIA, POLYGON_AMOY, etc.

## Key Classes

| Class | Purpose |
|-------|---------|
| ChaosChainAgentSDK | Main client for all operations |
| GatewayClient | Direct Gateway communication |
| X402PaymentManager | Handle crypto payments |
| X402PaywallServer | HTTP server with payment gates |
| VerifierAgent | Verify work and submit scores |

## Common Issues

### "No module named 'web3'"
```bash
pip uninstall web3 && pip install web3
```

### M1/M2 Mac issues
```bash
brew install openssl
export LDFLAGS="-L/opt/homebrew/opt/openssl/lib"
export CPPFLAGS="-I/opt/homebrew/opt/openssl/include"
pip install chaoschain-sdk
```

### Agent ID caching
The SDK caches agent IDs in `chaoschain_agent_ids.json`. This saves gas on repeated calls.

## Output Format

When writing code examples, always include:
1. Required imports
2. Environment variable handling (use `os.environ.get()`)
3. Error handling for blockchain operations
4. Comments explaining gas costs where relevant

## Environment Variables

```bash
export PRIVATE_KEY="your_private_key"
export SEPOLIA_RPC_URL="https://eth-sepolia.g.alchemy.com/v2/YOUR_KEY"
export ETH_MAINNET_RPC_URL="https://eth-mainnet.g.alchemy.com/v2/YOUR_KEY"
```
