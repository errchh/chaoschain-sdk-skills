# DeFi Studios

This reference covers building DeFi trading studios with ChaosChain agents.

## Overview

DeFi Studios enable multi-agent trading workflows:
- Strategy Agents: Analyze markets and propose trades
- Execution Agents: Execute trades on protocols
- Verifiers: Validate execution and manage risk

## Create DeFi Studio

```python
from chaoschain_sdk import ChaosChainAgentSDK, NetworkConfig, AgentRole

# Initialize client
client = ChaosChainAgentSDK(
    agent_name="DeFiStudioAdmin",
    agent_role=AgentRole.CLIENT,
    network=NetworkConfig.ETHEREUM_SEPOLIA
)

# Create DeFi Studio
studio, _ = client.create_studio(
    logic_module_address="0x05A70e3994d996513C2a88dAb5C3B9f5EBB7D11C",  # DeFi logic
    init_params=encode_defi_params({
        "max_position_size": 10000,  # USD
        "allowed_assets": ["ETH", "BTC", "USDC"],
        "risk_tolerance": "medium"
    })
)

# Fund escrow
client.fund_studio_escrow(
    studio_address=studio,
    amount_wei=100000000000000  # 0.0001 ETH
)
```

## Strategy Agent

```python
class StrategyAgent:
    """Proposes trading strategies."""
    
    def __init__(self, sdk):
        self.sdk = sdk
        self.role = AgentRole.WORKER
    
    def analyze_market(self) -> dict:
        """Generate trading signal."""
        return {
            "signal": "long",
            "asset": "ETH",
            "entry": 3200,
            "target": 3400,
            "stop_loss": 3100,
            "confidence": 0.75,
            "reasoning": "RSI oversold + support level"
        }
    
    def submit_strategy(self, studio, strategy):
        """Submit strategy to Studio."""
        dkg = self.build_strategy_dkg(strategy)
        
        self.sdk.submit_work(
            studio_address=studio,
            data_hash=self.sdk.w3.keccak(text=str(strategy)),
            thread_root=dkg.compute_thread_root(),
            evidence_root=bytes(32)
        )
```

## Execution Agent

```python
class ExecutionAgent:
    """Executes trading strategies."""
    
    def __init__(self, sdk):
        self.sdk = sdk
    
    def execute_strategy(self, strategy, studio):
        # Execute the trade on-chain
        # Build DKG showing execution
        
        result = {
            "executed": True,
            "tx_hash": "0x...",
            "price": strategy["entry"],
            "amount": strategy.get("amount", 1.0)
        }
        
        dkg = self.build_execution_dkg(result)
        return result, dkg
```

## Multi-Agent Work Submission

```python
# Contribution weights from DKG path analysis
contribution_weights = {
    strategy_agent_address: 0.60,  # Strategy is 60%
    execution_agent_address: 0.40  # Execution is 40%
}

# Submit multi-agent work
tx_hash = sdk.submit_work_multi_agent(
    studio_address=studio,
    data_hash=data_hash,
    thread_root=thread_root,
    evidence_root=evidence_root,
    participants=[strategy_addr, exec_addr],
    contribution_weights=contribution_weights,
    evidence_cid="ipfs://Qm..."
)
```

## Full Example

```python
from chaoschain_sdk import ChaosChainAgentSDK, NetworkConfig, AgentRole

# Initialize agents
strategy_sdk = ChaosChainAgentSDK(
    agent_name="StrategyAgent",
    agent_role=AgentRole.WORKER,
    network=NetworkConfig.ETHEREUM_SEPOLIA
)

exec_sdk = ChaosChainAgentSDK(
    agent_name="ExecutionAgent", 
    agent_role=AgentRole.WORKER,
    network=NetworkConfig.ETHEREUM_SEPOLIA
)

# Strategy agent proposes
strategy_agent = StrategyAgent(strategy_sdk)
strategy = strategy_agent.analyze_market()

# Execution agent executes
exec_agent = ExecutionAgent(exec_sdk)
result, exec_dkg = exec_agent.execute_strategy(strategy, studio)

# Submit as multi-agent work
exec_sdk.submit_work_multi_agent(
    studio_address=studio,
    data_hash=data_hash,
    participants=[strategy_addr, exec_addr],
    contribution_weights={
        strategy_addr: 0.6,
        exec_addr: 0.4
    }
)
```

## Risk Management

DeFi studios typically include:
- Maximum position size limits
- Allowed asset lists
- Stop-loss requirements
- Slippage tolerances

These are enforced by the logic module on-chain.

## Supported Protocols

The SDK can work with any DeFi protocol:
- Uniswap (swaps)
- Aave (lending)
- Compound (borrowing)
- Custom protocols via direct contract calls
