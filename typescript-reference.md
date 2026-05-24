# Polymarket TypeScript Reference

Companion to `SKILL.md` (which carries Python — the agent's runtime). These TypeScript snippets are reference-only; load this file only when working in a TS codebase.

## Client Setup

```typescript
import { ClobClient, Side, OrderType } from "@polymarket/clob-client";
import { Wallet } from "ethers"; // v5.8.0

const HOST = "https://clob.polymarket.com";
const CHAIN_ID = 137;
const signer = new Wallet(process.env.PRIVATE_KEY);

// Step 1: L1 — derive API credentials
const tempClient = new ClobClient(HOST, CHAIN_ID, signer);
const apiCreds = await tempClient.createOrDeriveApiKey();

// Step 2: L2 — init trading client
const client = new ClobClient(
  HOST,
  CHAIN_ID,
  signer,
  apiCreds,
  2,                // signatureType: 0=EOA, 1=POLY_PROXY, 2=GNOSIS_SAFE
  "FUNDER_ADDRESS"  // proxy wallet address from polymarket.com/settings
);
```

## Place an Order

```typescript
const response = await client.createAndPostOrder(
  {
    tokenID: "TOKEN_ID",
    price: 0.50,
    size: 10,
    side: Side.BUY,
  },
  {
    tickSize: "0.01",  // from client.getTickSize(tokenID) or market object
    negRisk: false,    // from client.getNegRisk(tokenID) or market object
  },
  OrderType.GTC
);
console.log(response.orderID, response.status);
```

## Read Orderbook

```typescript
// No auth needed
const readClient = new ClobClient("https://clob.polymarket.com", 137);
const book = await readClient.getOrderBook("TOKEN_ID");
console.log("Best bid:", book.bids[0], "Best ask:", book.asks[0]);

const mid = await readClient.getMidpoint("TOKEN_ID");
const spread = await readClient.getSpread("TOKEN_ID");
```

## WebSocket Subscribe

```typescript
const ws = new WebSocket("wss://ws-subscriptions-clob.polymarket.com/ws/market");

ws.onopen = () => {
  ws.send(JSON.stringify({
    type: "market",
    assets_ids: ["TOKEN_ID"],
    custom_feature_enabled: true,
  }));
  // Send PING every 10s to keep alive
  setInterval(() => ws.send("PING"), 10_000);
};

ws.onmessage = (event) => {
  if (event.data === "PONG") return;
  const msg = JSON.parse(event.data);
  // msg.event_type: "book" | "price_change" | "last_trade_price" | "tick_size_change" | "best_bid_ask" | "new_market" | "market_resolved"
};
```

Deeper detail on each topic is in the sibling files (`authentication.md`, `order-patterns.md`, `market-data.md`, `websocket.md`, `ctf-operations.md`, `bridge.md`, `gasless.md`).
