# ChaosChain Payments (x402)

This reference covers setting up crypto payments using the x402 protocol (Coinbase Protocol v2.0).

## Overview

x402 allows AI agents to accept payments for their services. Clients pay using USDC or other tokens via the x402 protocol.

```
Client → Payment Request → 402 Response → Payment Header → Service
```

## Setup Payment Manager

```python
from chaoschain_sdk import WalletManager, X402PaymentManager, NetworkConfig
import os

# Initialize wallet
wallet = WalletManager(
    network=NetworkConfig.BASE_SEPOLIA,
    private_key=os.environ.get("PRIVATE_KEY")
)

# Create payment manager
payments = X402PaymentManager(
    wallet_manager=wallet,
    network=NetworkConfig.BASE_SEPOLIA
)

# Check facilitator connection
schemes = payments.get_facilitator_supported_schemes()
print(f"Supported schemes: {schemes}")
```

## Create Paywall Server

```python
from chaoschain_sdk import X402PaywallServer

server = X402PaywallServer(
    agent_name="MyAIService",
    payment_manager=payments
)

# Decorate functions to require payment
@server.require_payment(amount=0.50, description="Quick Analysis")
def quick_analysis(data):
    prompt = data.get("prompt", "")
    return {"result": f"Analysis of: {prompt}", "confidence": 0.92}

@server.require_payment(amount=2.00, description="Deep Analysis")
def deep_analysis(data):
    return {"result": "Deep analysis complete", "insights": []}

# Start server
server.run(host="0.0.0.0", port=8402)
```

## Client Payment Flow

```python
# Client making a payment
result = payments.execute_agent_payment(
    from_agent="ClientBot",
    to_agent="MyAIService",
    amount_usdc=1.00,
    service_description="AI Image Generation",
    evidence_cid="ipfs://Qm..."  # Optional
)

if result["success"]:
    print(f"Payment TX: {result['main_transaction_hash']}")
```

## Full Example: Paid AI Agent

```python
from chaoschain_sdk import (
    ChaosChainAgentSDK, 
    X402PaymentManager, 
    X402PaywallServer,
    WalletManager,
    NetworkConfig
)

# 1. Register agent identity
sdk = ChaosChainAgentSDK(
    agent_name="SmartAnalyzer",
    agent_domain="analyzer.ai",
    network=NetworkConfig.ETHEREUM_MAINNET,
    private_key="0x..."
)
agent_id, _ = sdk.register_identity()

# 2. Setup payment manager
wallet = WalletManager(network=NetworkConfig.BASE_SEPOLIA)
payments = X402PaymentManager(wallet, NetworkConfig.BASE_SEPOLIA)

# 3. Create paywall server
server = X402PaywallServer("SmartAnalyzer", payments)

# 4. Define paid services
@server.require_payment(amount=0.50, description="Quick Analysis")
def quick_analysis(data):
    return {"result": "Analysis complete", "confidence": 0.92}

# 5. Run
server.run(port=8402)
```

## Payment Amounts

Amounts are in the smallest unit (like wei):
- USDC: 6 decimals (1.00 USDC = 1000000)
- ETH: 18 decimals

Use the protocol's `exact` scheme for exact amounts.

## Networks

- **Base Sepolia**: Testing (recommended)
- **Base Mainnet**: Production payments
- Other chains supported via NetworkConfig
