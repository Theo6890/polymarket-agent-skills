---
name: web3-polymarket
description: Polymarket integration for prediction market trading on Polygon. Covers authentication (L1 EIP-712, L2 HMAC-SHA256, builder headers), order placement (GTC/GTD/FOK/FAK, batch, post-only, heartbeat), market data (Gamma API, Data API, orderbook, subgraph), WebSocket streaming (market/user/sports channels), CTF operations (split, merge, redeem, negative risk), bridge (deposits, withdrawals, multi-chain), and gasless relayer transactions. Use when building AI agents, autonomous market makers, prediction market UIs, or any application integrating with Polymarket on Polygon.
compatibility: Requires network access to Polymarket APIs (clob.polymarket.com, gamma-api.polymarket.com) and Polygon RPC
---

# Polymarket Skill

## When to use this skill

Use this skill when the user asks about or needs to build:
- Polymarket API authentication (L1/L2, API keys, HMAC signing)
- Placing or managing orders (limit, market, GTC, GTD, FOK, FAK, batch, cancel)
- Reading orderbook data (prices, spreads, midpoints, depth)
- Market data fetching (events, markets, by slug, by tag, pagination)
- WebSocket subscriptions (market channel, user channel, sports)
- CTF operations (split, merge, redeem positions)
- Negative risk markets (multi-outcome, conversion, augmented neg risk)
- Bridge operations (deposits, withdrawals, multi-chain)
- Gasless transactions (relayer client, order attribution)
- Builder program integration (order attribution, API keys, tiers)
- Polymarket SDK usage (TypeScript @polymarket/clob-client, Python py-clob-client)

## API Configuration

| API | Base URL | Auth | Purpose |
|-----|----------|------|---------|
| CLOB | `https://clob.polymarket.com` | L2 for trade endpoints | Orderbook, prices, order submission |
| Gamma / Data | `https://gamma-api.polymarket.com` | None | Events, markets, search |
| Data API | `https://data-api.polymarket.com` | None | Trades, positions, user data |
| WebSocket (Market) | `wss://ws-subscriptions-clob.polymarket.com/ws/market` | None | Real-time orderbook |
| WebSocket (User) | `wss://ws-subscriptions-clob.polymarket.com/ws/user` | API creds in message | Trade/order updates |
| WebSocket (Sports) | `wss://sports-api.polymarket.com/ws` | None | Live scores |
| Relayer | `https://relayer-v2.polymarket.com/` | Builder headers | Gasless transactions |
| Bridge | `https://bridge.polymarket.com` | None | Deposits/withdrawals |

## Contract Addresses (Polygon)

| Contract | Address |
|----------|---------|
| USDC (USDC.e) | `0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174` |
| CTF (Conditional Tokens) | `0x4D97DCd97eC945f40cF65F87097ACe5EA0476045` |
| CTF Exchange | `0x4bFb41d5B3570DeFd03C39a9A4D8dE6Bd8B8982E` |
| Neg Risk CTF Exchange | `0xC5d563A36AE78145C45a50134d48A1215220f80a` |
| Neg Risk Adapter | `0xd91E80cF2E7be2e162c6513ceD06f1dD0dA35296` |

## Client Setup (Python — agent runtime)

```python
from py_clob_client.client import ClobClient
import os

host = "https://clob.polymarket.com"
chain_id = 137
pk = os.getenv("PRIVATE_KEY")

# Step 1: L1 — derive API credentials
temp_client = ClobClient(host, key=pk, chain_id=chain_id)
api_creds = temp_client.create_or_derive_api_creds()

# Step 2: L2 — init trading client
client = ClobClient(
    host,
    key=pk,
    chain_id=chain_id,
    creds=api_creds,
    signature_type=2,  # 0=EOA, 1=POLY_PROXY, 2=GNOSIS_SAFE
    funder="FUNDER_ADDRESS",
)
```

TypeScript variant: `typescript-reference.md`.

## Quick Reference: Order Types

| Type | Behavior | Use Case |
|------|----------|----------|
| **GTC** | Rests on book until filled or cancelled | Default limit orders |
| **GTD** | Active until expiration (UTC seconds). Min = `now + 60 + N` | Auto-expire before events |
| **FOK** | Fill entirely immediately or cancel | All-or-nothing market orders |
| **FAK** | Fill what's available, cancel rest | Partial-fill market orders |

- FOK/FAK BUY: `amount` = dollar amount to spend
- FOK/FAK SELL: `amount` = number of shares to sell
- Post-only: GTC/GTD only — rejected if would cross spread

## Quick Reference: Signature Types

| Type | Value | Description |
|------|-------|-------------|
| EOA | `0` | Standard Ethereum wallet (MetaMask). Funder is the EOA address and will need POL for gas. |
| POLY_PROXY | `1` | Custom proxy wallet for Magic Link email/Google users who exported PK from Polymarket.com. |
| GNOSIS_SAFE | `2` | Gnosis Safe multisig proxy wallet (most common). Use for any new or returning user. |

## Core Pattern: Place an Order (Python)

```python
from py_clob_client.clob_types import OrderArgs, OrderType
from py_clob_client.order_builder.constants import BUY

response = client.create_and_post_order(
    OrderArgs(token_id="TOKEN_ID", price=0.50, size=10, side=BUY),
    options={"tick_size": "0.01", "neg_risk": False},
    order_type=OrderType.GTC,
)
print(response["orderID"], response["status"])
```

## Core Pattern: Read Orderbook (Python)

```python
read_client = ClobClient("https://clob.polymarket.com", chain_id=137)
book = read_client.get_order_book("TOKEN_ID")
mid = read_client.get_midpoint("TOKEN_ID")
spread = read_client.get_spread("TOKEN_ID")
```

## WebSocket

Full subscribe/heartbeat pattern lives in `websocket.md` (loaded on demand). TypeScript variant in `typescript-reference.md`.

## Reference files (load on demand)

Only read these when the task requires deeper detail on a specific topic:

- **Authentication** (L1/L2, builder headers, credential lifecycle): [authentication.md](authentication.md)
- **Order patterns** (GTC/GTD/FOK/FAK, tick sizes, cancel, heartbeat, errors): [order-patterns.md](order-patterns.md)
- **Market data** (Gamma API, Data API, CLOB orderbook, subgraph): [market-data.md](market-data.md)
- **WebSocket** (market/user/sports channels, subscribe, heartbeat): [websocket.md](websocket.md)
- **CTF operations** (split, merge, redeem, neg risk, token IDs): [ctf-operations.md](ctf-operations.md)
- **Bridge** (deposits, withdrawals, supported chains/tokens, status): [bridge.md](bridge.md)
- **Gasless transactions** (relayer client, wallet deployment, builder setup): [gasless.md](gasless.md)
- **TypeScript variants** of the core patterns above: [typescript-reference.md](typescript-reference.md)
