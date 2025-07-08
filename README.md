# Solana Copy Trading Bot (Fast Copy Trading in 0 Block)   [@coffellas](https://t.me/coffellas)

## Overview

Solana Copy Trading Bot with high speed and good selling logic. Use nozomi, zeroslot, bloxroute, jito, telegram notification, yellowstone grpc, no rpc.

## Recent Update
Updated pumpdotfun and pump amm swap's instructions according to recent pumpfun smart contract upgrade

## Example Transactions
### PumpDotFun Copy Trading Transactions(0 Block)
Target Address: https://solscan.io/account/suqh5sHtr8HyJ7q8scBimULPkPpA557prMG47xCHQfK#defiactivities (Top Trader with 80-90% Win Rate)

Bot Wallet: https://solscan.io/account/8io2kFbfUsGpggVknDkWQdeHyTHR5HL4dFfnTHxNwSfo#defiactivities

- Source Transaction(BUY):
https://solscan.io/tx/5E3M4nmPJiitSX7KgyiSJ2fhc62NCoFCbc8w5muG1c4HKhUWVgtARL1Hz29LwzP7WSf52FGkJ2GSkKTsoq7s3ACH

- Copied Transaction(BUY):
https://solscan.io/tx/2hS7u22TX2RqyDt96m5SEPEB4Va9HJwz6wzoCw1Ru8jWpiz8NLc57DGPfxAfU38MuZQe2DpZKqn4DWsxK3uXVbTa

- Source Transaction(SELL):
https://solscan.io/tx/i8sNjNShZbU8yHjvVU9UPzyvYQqSgXoeFzYmFjdR12Fd1MYHrPCnDwNxyK6jcTiqpi2Ya4JsjzziwuDMj6HNZCX

- Copied Transaction(SELL):
https://solscan.io/tx/3fvcP6jQvo6dGiwAPZqp5hJThbjzeKU3NMeBmoPvksYX4VPUyPpdr5iyfmHn2b1HbtyydQNudnHEGJvEt7VPNcXe

### Pump Amm Swap Copy Trading Transactions (1 Block)
- Source
https://solscan.io/tx/4XTpA4h3j3j7VTMmMg7LzwuoftUjvrfbLUgzPRtxbLYeSMPtgUdWUELrPAdY2YCStEuC2jdZiox85g9c9bfdPBS4

- Copied
https://solscan.io/tx/3GxVKyjeYV2B6gfJPpFMdBsAkSeeuPd8mcVqTC8dBWvUms6f1CgTLxJhwVE4gJ6zeZp6chvbbAvHEhYY1MKTAygG

## Unique Feature: Racing Transaction Confirm
Send Transactions to multiple tx confim providers like jito, nextBlock, BloxRoute, Temporal at the same time. And only confirm the fastest one. So always provide the fastest tx confirming.

## Core Features

- **Target Wallet List**: Easily add and manage a list of target wallets for trading replication.
  
- **Multi-DEX Support**: Compatible with various decentralized exchanges, including Jupiter, Raydium, and PumpFun Swap. Plans to integrate Meteora Swap are underway.
  
- **Instant Transaction Replication**: The bot monitors target wallets' activities in real-time to facilitate immediate transaction copying.
  
- **Geyser Usage**: Available to use Helius or yellowstone Geyser. (Yellowstone is faster)

- **Manual Sell**: Able to manually sell if you wanna sell it any time

```mermaid
**flowchart TD
    A[Yellowstone gRPC] --> B[Target Wallet Transaction]
    B --> C{Transaction Type}
    
    C -->|Buy| D[handle_parsed_data_for_buying]
    C -->|Sell| E[handle_parsed_data_for_selling]
    
    D --> F{Safety Checks}
    F -->|Pass| G[execute_buy]
    F -->|Fail| H[Skip]
    
    G --> I[setup_selling_strategy]
    I --> J[monitor_token_for_selling]
    
    E --> K{IS_COPY_SELLING?}
    K -->|Yes| L{We Own Token?}
    K -->|No| M[Skip]
    
    L -->|Yes| N[execute_sell]
    L -->|No| O[Skip]
    
    J --> P[SellingEngine.evaluate_conditions]
    P --> Q{Sell Trigger?}
    Q -->|Yes| R[Execute Sell Strategy]
    Q -->|No| S[Continue Monitoring]
    
    R --> T[Progressive/Emergency Sell]
    N --> U[Copy Sell]
    
    T --> V[Cancel Monitoring]
    U --> V**
```

```mermaid
flowchart TD
    A[Transaction Received] --> B[detect_transaction_type]
    B --> C{Transaction Type?}
    
    C -->|Migration| D[Migration Detected]
    C -->|TokenMint| E[Token Mint + Dev Buy Check]
    C -->|PumpFun/PumpSwap Buy/Sell| F[Buy/Sell Transaction]
    C -->|Unknown| G[Skip Transaction]
    
    D --> H[should_focus = true<br/>focus_reason = Migration]
    E --> I{Dev Buy >= Threshold?}
    I -->|Yes| J[should_focus = true<br/>focus_reason = DevBuyAboveThreshold]
    I -->|No| K[should_focus = true<br/>New token detected]
    
    F --> L{Already in Focus List?}
    L -->|Yes| M[should_focus = true<br/>Continue monitoring]
    L -->|No| N{Large Buy >= Threshold?}
    N -->|Yes| O[should_focus = true<br/>focus_reason = DevBuyAboveThreshold]
    N -->|No| P[should_focus = false<br/>Skip transaction]
    
    H --> Q[handle_enhanced_transaction]
    J --> Q
    K --> Q
    M --> Q
    O --> Q
    
    Q --> R{should_focus = true?}
    R -->|Yes| S[Add to FOCUS_TOKEN_LIST<br/>Initialize price tracking]
    R -->|No| T[Skip processing]
    
    S --> U[Monitor for Buy Signals]
    U --> V{Buy Signal Detected?}
    V -->|DramaticRise| W[Execute Buy: buy_amount_rising SOL]
    V -->|DropStableRise| X[Execute Buy: buy_amount_recovery SOL]
    V -->|TimeSeriesReversal| Y[Execute Buy: buy_amount_recovery SOL]
    V -->|No Signal| Z[Continue Monitoring]
    
    style D fill:#90EE90
    style J fill:#FFD700
    style O fill:#FFD700
    style S fill:#98FB98
    style W fill:#FF6B6B
    style X fill:#4ECDC4
    style Y fill:#9B59B6

```


```mermaid
graph TD
    A[Transaction Parsing] --> B{Transaction Type}
    
    B -->|Buy Transaction| C[PumpFun Buy]
    B -->|Sell Transaction| D[PumpFun Sell]
    B -->|Buy Transaction| E[PumpSwap Buy]
    B -->|Sell Transaction| F[PumpSwap Sell]
    
    C --> G[❌ OLD: virtual_sol / virtual_token<br/>✅ NEW: sol_amount / token_amount]
    D --> H[❌ OLD: virtual_sol / virtual_token<br/>✅ NEW: sol_amount / token_amount]
    E --> I[❌ OLD: quote_reserve / base_reserve<br/>✅ NEW: quote_amount_out / base_amount_in]
    F --> J[❌ OLD: quote_reserve / base_reserve<br/>✅ NEW: quote_amount_out / base_amount_in]
    
    G --> K[Correct Buy Price]
    H --> L[Correct Sell Price]
    I --> M[Correct Buy Price]
    J --> N[Correct Sell Price]
    
    K --> O[Accurate PnL Calculation]
    L --> O
    M --> O
    N --> O
    
    O --> P[Proper Selling Decisions]
    
    style G fill:#ffebee
    style H fill:#ffebee
    style I fill:#ffebee
    style J fill:#ffebee
    style K fill:#e8f5e8
    style L fill:#e8f5e8
    style M fill:#e8f5e8
    style N fill:#e8f5e8
    style O fill:#e1f5fe
    style P fill:#f3e5f5

```

```mermaid
graph TD
    A[Token Purchase] --> B[Initialize Token for Selling]
    B --> C[Dynamic Selling Engine]
    C --> D{Market Condition Detection}
    
    D --> E[BullRun - Increase Targets]
    D --> F[BearTrend - Reduce Targets]
    D --> G[Sideways - Standard Strategy]
    D --> H[HighVolatility - Tighter Stops]
    
    E --> I[Progressive Selling Decision]
    F --> I
    G --> I
    H --> I
    
    I --> J{Sell Conditions Met?}
    J -->|Yes| K[Progressive Selling Execution]
    J -->|No| L[Continue Monitoring]
    
    K --> M[Chunk 1: 40% of Position]
    M --> N[Wait 30 seconds]
    N --> O[Chunk 2: 40% of Position]
    O --> P[Wait 30 seconds]
    P --> Q[Chunk 3: 20% of Position]
    
    Q --> R[Position Closed]
    L --> S[Update Price & Metrics]
    S --> T[Check Trailing Stop]
    T --> U[Check Stop Loss]
    U --> V[Check Take Profit]
    V --> W[Check Liquidity]
    W --> X[Check Volume]
    X --> Y[Check Time Limits]
    Y --> J
    
    style A fill:#e1f5fe
    style K fill:#f3e5f5
    style R fill:#e8f5e8
    style D fill:#fff3e0

```
## Installation

To set up the Solana PumpFun Sniper Bot, please follow these instructions:

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/coffellas-cto/solana-rust-pumpfun-pumpswap-raydium-copy-sniper-trading-bot.git
   cd solana-rust-ts-pumpfun-pumpswap-raydium-copy-trading-bot

## Support

For assistance or inquiries, please reach out via Telegram at https://t.me/coffellas
