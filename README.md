# DeBox Handle Marketplace - Embed API Documentation

The Embed API allows you to render DeBox handles with their customization effects in your applications. This includes animated text effects, gradients, glow effects, and community branding.

---

## Table of Contents

1. [Overview](#overview)
2. [Quick Start](#quick-start)
3. [Authentication](#authentication)
4. [API Reference](#api-reference)
   - [Discovery Endpoints](#discovery-endpoints)
   - [Asset Endpoints](#asset-endpoints)
   - [Configuration Endpoints](#configuration-endpoints)
   - [Handle Ownership Endpoints](#handle-ownership-endpoints)
5. [Blockchain Event Monitoring](#blockchain-event-monitoring)
6. [Web Integration Guide](#web-integration-guide)
7. [Native Mobile SDK Guide](#native-mobile-sdk-guide)
8. [Configuration Reference](#configuration-reference)
9. [Caching & Change Detection](#caching--change-detection)
10. [Testing](#testing)
11. [Error Handling](#error-handling)

---

## Overview

The API provides **two integration approaches** depending on your platform:

### Web Applications (CSS + JavaScript)

For web-based applications (websites, WebViews, web apps):
- **`/styles.css`** - Pre-built CSS with all effect animations
- **`/renderer.js`** - JavaScript library to apply effects to DOM elements
- **`/:tokenAddress/config`** - JSON configuration for the web renderer

The renderer handles all the complexity of applying gradients, animations, and glow effects using CSS.

### Native Mobile Applications (JSON Config)

For native iOS (Swift) and Android (Kotlin) applications:
- **`/:tokenAddress/native`** - Platform-agnostic JSON with rendering instructions

This endpoint returns structured data with keyframe animations that your native code can interpret. No CSS or JavaScript required.

### Comparison

| Feature | Web (CSS/JS) | Native (JSON) |
|---------|--------------|---------------|
| Platform | Browsers, WebViews | iOS, Android native |
| Rendering | CSS animations, DOM | Native UI frameworks |
| Effects | Full CSS support | Keyframe-based animations |
| Glow | CSS text-shadow | Shadow layer configurations |
| Gradients | CSS gradients | Color array + direction |

---

## Quick Start

### For Web Applications

**Step 1:** Test the API to make sure it works:

```bash
curl -s -H "x-api-key: YOUR_API_KEY" \
  "https://debox-handle-marketplace.replit.app/api/embed/0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7/config?env=testnet"
```

**Step 2:** Create a simple HTML page:

```html
<!DOCTYPE html>
<html>
<head>
  <title>DeBox Handle Demo</title>
  <link rel="stylesheet" href="https://debox-handle-marketplace.replit.app/api/embed/styles.css">
</head>
<body style="background:#111; padding:40px; font-family:sans-serif;">
  <h1 style="color:white;">DeBox Handle Demo</h1>
  <div style="font-size:32px; margin:40px 0;">
    <span id="handle">loading...</span>
  </div>

  <script src="https://debox-handle-marketplace.replit.app/api/embed/renderer.js"></script>
  <script>
    const API_KEY = 'YOUR_API_KEY';
    const TOKEN = '0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7';
    
    fetch(`https://debox-handle-marketplace.replit.app/api/embed/${TOKEN}/config?env=testnet`, {
      headers: { 'x-api-key': API_KEY }
    })
      .then(res => res.json())
      .then(config => {
        const el = document.getElementById('handle');
        DeBoxHandleRenderer.render(el, 'alice' + config.namespaceSuffix, config);
      });
  </script>
</body>
</html>
```

### For Native Mobile Apps

**Step 1:** Fetch the native config:

```bash
curl -s -H "x-api-key: YOUR_API_KEY" \
  "https://debox-handle-marketplace.replit.app/api/embed/0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7/native?env=testnet"
```

**Step 2:** Parse the JSON response and render using your platform's native UI framework. See the [Native Mobile SDK Guide](#native-mobile-sdk-guide) for complete Swift and Kotlin examples.

---

## Authentication

All API requests require an API key.

### Obtaining an API Key

1. **Contact the DeBox team** to request an API key
2. Provide details about your use case and expected usage volume
3. The team will generate a key and provide it securely

### Using Your API Key

Include the API key in your requests using one of these methods:

**Option 1: x-api-key Header (Recommended)**
```bash
curl -H "x-api-key: dbx_abc123..." \
  "https://debox-handle-marketplace.replit.app/api/embed/..."
```

**Option 2: Authorization Bearer Token**
```bash
curl -H "Authorization: Bearer dbx_abc123..." \
  "https://debox-handle-marketplace.replit.app/api/embed/..."
```

### API Key Format

- Keys start with the prefix `dbx_`
- Example: `dbx_a1b2c3d4e5f6...`
- Keys are 32+ characters long
- Store securely - keys cannot be retrieved after creation

### Authentication Errors

| Status | Error | Description |
|--------|-------|-------------|
| 401 | `API key required` | No API key provided |
| 401 | `Invalid API key` | Key is invalid or revoked |

---

## API Reference

**Base URL:** `https://debox-handle-marketplace.replit.app/api/embed`

All endpoints require authentication via API key.

---

### Discovery Endpoints

These endpoints help you discover communities and detect configuration changes.

#### GET /communities

Returns a list of all registered communities with minimal data.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `env` | string | `mock` | Environment: `mock`, `testnet`, or `mainnet` |

**Response:**
```json
{
  "environment": "testnet",
  "count": 3,
  "communities": [
    {
      "name": "DeBox",
      "tokenAddress": "0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7",
      "slug": "debox"
    }
  ]
}
```

**Example:**
```bash
curl -s -H "x-api-key: YOUR_API_KEY" \
  "https://debox-handle-marketplace.replit.app/api/embed/communities?env=testnet"
```

---

#### GET /changes

Returns communities with configuration changes since a specified timestamp. Designed for efficient polling to detect when cached configs need refreshing.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `env` | string | `mock` | Environment: `mock`, `testnet`, or `mainnet` |
| `since` | string | *(none)* | ISO timestamp - return only communities changed after this time |

**Response:**
```json
{
  "environment": "testnet",
  "serverTime": "2025-12-17T12:05:42.082Z",
  "since": "2025-12-17T11:00:00.000Z",
  "count": 2,
  "changes": [
    {
      "name": "DeBox",
      "tokenAddress": "0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7",
      "slug": "debox",
      "configUpdatedAt": "2025-12-17T11:45:00.000Z"
    }
  ]
}
```

**Polling Pattern:**

1. Make initial call without `since` to get all communities and their timestamps
2. Save `serverTime` from the response
3. Poll every 5 minutes using saved `serverTime` as the `since` parameter
4. For each changed community, refresh your cached config
5. Update saved timestamp to new `serverTime`

**Example:**
```bash
# Initial call
curl -s -H "x-api-key: YOUR_API_KEY" \
  "https://debox-handle-marketplace.replit.app/api/embed/changes?env=testnet"

# Subsequent poll (use serverTime from previous response)
curl -s -H "x-api-key: YOUR_API_KEY" \
  "https://debox-handle-marketplace.replit.app/api/embed/changes?env=testnet&since=2025-12-17T12:00:00.000Z"
```

---

### Asset Endpoints

These endpoints serve static assets for web rendering.

#### GET /version

Returns version information about the embed assets.

**Response:**
```json
{
  "latest": "0.0.1",
  "versions": [
    { "version": "0.0.1", "createdAt": "2025-12-06T05:41:35.821Z" }
  ],
  "generatedAt": "2025-12-06T05:41:35.821Z"
}
```

---

#### GET /styles.css

Returns the CSS file containing all effect animations and styles.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `v` | string | *(latest)* | Specific version for immutable caching |

**Headers:**
- `Content-Type: text/css`
- `X-Embed-Version: <version>`
- `Cache-Control: public, max-age=300` (or `max-age=31536000, immutable` for versioned)

**Example:**
```bash
# Get latest
curl -s -H "x-api-key: YOUR_API_KEY" \
  "https://debox-handle-marketplace.replit.app/api/embed/styles.css"

# Get specific version (immutable caching)
curl -s -H "x-api-key: YOUR_API_KEY" \
  "https://debox-handle-marketplace.replit.app/api/embed/styles.css?v=0.0.1"
```

---

#### GET /renderer.js

Returns the JavaScript renderer for applying handle styles.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `v` | string | *(latest)* | Specific version for immutable caching |

**Headers:**
- `Content-Type: application/javascript`
- `X-Embed-Version: <version>`

---

### Configuration Endpoints

These endpoints return customization configurations for specific communities.

#### GET /:tokenAddress/config

Returns the web-optimized configuration for a community. Use with the CSS and JavaScript renderer.

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `tokenAddress` | string | Community's token contract address (0x...) |

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `env` | string | `mock` | Environment: `mock`, `testnet`, or `mainnet` |

**Headers:**
- `ETag: "<hash>"` (for conditional requests)
- `Cache-Control: public, max-age=60`

**Response:**
```json
{
  "tokenAddress": "0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7",
  "communitySlug": "community",
  "communityName": "Community",
  "namespaceSuffix": ".debox",
  "baseColor": {
    "type": "custom",
    "value": "#22C55E"
  },
  "effect": "shimmer",
  "prebuiltEffect": null,
  "glow": {
    "enabled": true,
    "colorSource": "base",
    "color": null,
    "size": 15,
    "intensity": 100
  },
  "logoUrl": "https://example.com/logo.png",
  "configUpdatedAt": "2025-12-06T10:30:00.000Z",
  "embedVersion": "0.0.1"
}
```

**Conditional Requests (ETag):**
```bash
# Get the ETag
curl -sI -H "x-api-key: YOUR_API_KEY" \
  "https://debox-handle-marketplace.replit.app/api/embed/0x82C8.../config?env=testnet"

# Use If-None-Match for conditional request (returns 304 if unchanged)
curl -s -H "x-api-key: YOUR_API_KEY" \
  -H "If-None-Match: \"abc123\"" \
  "https://debox-handle-marketplace.replit.app/api/embed/0x82C8.../config?env=testnet"
```

---

#### GET /:tokenAddress/native

Returns the native mobile SDK configuration. Use for iOS (Swift) and Android (Kotlin) apps.

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `tokenAddress` | string | Community's token contract address (0x...) |

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `env` | string | `mock` | Environment: `mock`, `testnet`, or `mainnet` |

**Headers:**
- `Content-Type: application/json`
- `ETag: "<hash>"` (for conditional requests)
- `Cache-Control: public, max-age=60`

**Response (Solid Color):**
```json
{
  "version": "1.0.0",
  "tokenAddress": "0x1234567890abcdef...",
  "community": {
    "name": "My Community",
    "slug": "mycommunity",
    "namespaceSuffix": ".mycom"
  },
  "configUpdatedAt": "2025-12-17T10:00:00.000Z",
  "textStyle": {
    "type": "solid",
    "color": "#22C55E"
  }
}
```

**Response (Prebuilt Effect with Animation):**
```json
{
  "version": "1.0.0",
  "tokenAddress": "0x1234567890abcdef...",
  "community": {
    "name": "My Community",
    "slug": "mycommunity",
    "namespaceSuffix": ".mycom"
  },
  "configUpdatedAt": "2025-12-17T10:00:00.000Z",
  "textStyle": {
    "type": "gradient",
    "direction": "toRight",
    "colors": ["#22C55E", "#3B82F6"]
  },
  "animation": {
    "type": "hueShift",
    "duration": 4000,
    "repeat": "infinite",
    "easing": "linear",
    "keyframes": [
      { "position": 0.0, "hueRotation": 0 },
      { "position": 0.5, "hueRotation": 90 },
      { "position": 1.0, "hueRotation": 0 }
    ]
  },
  "glow": {
    "size": 10,
    "intensity": 140,
    "color": "#84ecff",
    "colorSource": "custom"
  },
  "shadow": {
    "layers": [
      { "color": "#84ecff", "blur": 5, "opacity": 1.26 },
      { "color": "#84ecff", "blur": 10, "opacity": 0.98 },
      { "color": "#84ecff", "blur": 15, "opacity": 0.7 }
    ]
  }
}
```

---

### Handle Ownership Endpoints

These endpoints allow you to query which handles a wallet address owns.

#### GET /api/handles/by-wallet/:address

Returns all handles owned by a specific wallet address. Supports multiple query modes for different use cases.

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `address` | string | Wallet address (0x...) - case-insensitive |

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `mode` | string | `database` | Query mode: `database`, `chain`, or `auto` |
| `network` | string | `testnet` | Network for chain queries: `testnet` or `mainnet` |

**Query Modes:**

| Mode | Speed | Data Source | Use Case |
|------|-------|-------------|----------|
| `database` | ~10ms | Database only | Fast UI updates, non-critical displays |
| `chain` | ~700ms | Blockchain only | Authoritative ownership verification |
| `auto` | ~10ms | Database + background chain verification | **Recommended** - Best of both worlds |

**Recommended Modes:**

- **`mode=auto`** (Recommended for most use cases) - Returns database results immediately for fast UI, while triggering background blockchain verification. The verification status is cached for 10 minutes. Subsequent calls show whether the data is fresh or stale.

- **`mode=chain`** (Recommended for critical operations) - Queries the blockchain directly for authoritative ownership data. Use this when you need guaranteed accuracy, such as before processing transactions or displaying ownership proofs.

**Response (mode=database):**
```json
{
  "source": "database",
  "mode": "database",
  "network": "testnet",
  "walletAddress": "0x3121d4d08EcDDd4dfd4cc4F1fD29B0A6E0b9649B",
  "handles": [
    {
      "id": "f7c876af-ede4-47a1-8729-442f60b4e327",
      "tokenId": "1",
      "name": "carlos",
      "fullName": "carlos.doge",
      "communityId": "011bb319-ae83-4bef-a236-d42949393816",
      "communitySlug": "doge",
      "mintedAt": "2025-11-26T10:10:45.070Z"
    }
  ],
  "totalCount": 1
}
```

**Response (mode=chain):**
```json
{
  "source": "chain",
  "mode": "chain",
  "network": "testnet",
  "walletAddress": "0x3121d4d08EcDDd4dfd4cc4F1fD29B0A6E0b9649B",
  "handles": [
    {
      "tokenId": "1",
      "fullName": "carlos.doge",
      "communityLabel": "doge"
    }
  ],
  "totalCount": 1
}
```

**Response (mode=auto):**
```json
{
  "source": "database",
  "mode": "auto",
  "network": "testnet",
  "walletAddress": "0x3121d4d08EcDDd4dfd4cc4F1fD29B0A6E0b9649B",
  "handles": [...],
  "totalCount": 15,
  "verification": {
    "status": "fresh",
    "lastVerifiedAt": "2025-12-24T03:35:26.964Z",
    "nextCheckAt": "2025-12-24T03:45:26.964Z",
    "verificationPending": false,
    "discrepancyCount": 6,
    "cachedChainCount": 21
  },
  "hint": "Verification cache is fresh"
}
```

**Verification Status Values (mode=auto):**

| Status | Description |
|--------|-------------|
| `unknown` | No previous verification - background check triggered |
| `pending` | Verification currently in progress |
| `fresh` | Verified within the last 10 minutes |
| `stale` | Cache expired - background re-verification triggered |
| `error` | Last verification failed - will retry |

**Discrepancy Detection:**

When using `mode=auto`, the response includes:
- `discrepancyCount`: Difference between blockchain count and database count
- `cachedChainCount`: Number of handles found on the blockchain

A non-zero `discrepancyCount` indicates the database may be out of sync with the blockchain. This can happen when:
- Handles were transferred directly on-chain
- Recent mints haven't been indexed yet
- Event indexer is behind

**Examples:**

```bash
# Fast database query (default)
curl -s -H "x-api-key: YOUR_API_KEY" \
  "https://debox-handle-marketplace.replit.app/api/handles/by-wallet/0x3121d4d08EcDDd4dfd4cc4F1fD29B0A6E0b9649B"

# Recommended: Auto mode with background verification (testnet)
curl -s -H "x-api-key: YOUR_API_KEY" \
  "https://debox-handle-marketplace.replit.app/api/handles/by-wallet/0x3121d4d08EcDDd4dfd4cc4F1fD29B0A6E0b9649B?mode=auto&network=testnet"

# Recommended: Auto mode with background verification (mainnet)
curl -s -H "x-api-key: YOUR_API_KEY" \
  "https://debox-handle-marketplace.replit.app/api/handles/by-wallet/0x3121d4d08EcDDd4dfd4cc4F1fD29B0A6E0b9649B?mode=auto&network=mainnet"

# Authoritative blockchain query (requires network parameter)
curl -s -H "x-api-key: YOUR_API_KEY" \
  "https://debox-handle-marketplace.replit.app/api/handles/by-wallet/0x3121d4d08EcDDd4dfd4cc4F1fD29B0A6E0b9649B?mode=chain&network=testnet"
```

**Important Notes:**

1. **Network parameter is required for `mode=chain` and `mode=auto`** - You must specify `network=testnet` or `network=mainnet` when using blockchain queries.

2. **10-minute cache TTL** - In auto mode, blockchain verification is cached for 10 minutes. The `nextCheckAt` field shows when the cache will expire.

3. **Background verification is non-blocking** - Auto mode returns database results immediately while verification happens asynchronously.

4. **Poll for verification status** - After triggering a background verification (when `verificationPending: true`), call the endpoint again after a few seconds to see the updated status.

---

## Blockchain Event Monitoring

Because DeBox handles are ERC-721 NFTs, every ownership change emits the standard `Transfer` event that all marketplaces and indexers understand. You can monitor these events directly on the blockchain to get real-time notifications for mints, transfers, and burns.

### Understanding Transfer Events

The ERC-721 `Transfer` event has this signature:

```solidity
event Transfer(address indexed from, address indexed to, uint256 indexed tokenId);
```

You can detect all lifecycle events from this single event:

| Event Type | `from` Address | `to` Address | Example |
|------------|---------------|--------------|---------|
| **Mint** | `0x0000...0000` | User wallet | New handle created |
| **Transfer** | User A | User B | Handle sold or gifted |
| **Burn** | User wallet | `0x0000...0000` | Handle destroyed |

**Event Topic (keccak256 hash):**
```
0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef
```

### Contract Addresses

| Network | HandleRegistry Proxy Address |
|---------|------------------------------|
| BSC Testnet | See `docs/DEPLOYED_CONTRACTS.md` |
| BSC Mainnet | See `docs/DEPLOYED_CONTRACTS.md` |

---

### Option A: WebSocket Subscription (Near Real-Time)

Use `eth_subscribe` with a WebSocket provider for streaming events. This gives you near-instant notifications.

**JavaScript Example (ethers.js v6):**

```javascript
import { ethers } from 'ethers';

// Use a WebSocket provider for real-time events
const wsProvider = new ethers.WebSocketProvider('wss://bsc-rpc.publicnode.com');

// HandleRegistry contract address (use your deployed proxy address)
const HANDLE_REGISTRY = '0xYourHandleRegistryProxyAddress';

// Minimal ABI for Transfer event
const abi = [
  'event Transfer(address indexed from, address indexed to, uint256 indexed tokenId)'
];

const contract = new ethers.Contract(HANDLE_REGISTRY, abi, wsProvider);

// Listen for all Transfer events
contract.on('Transfer', (from, to, tokenId, event) => {
  const ZERO_ADDRESS = '0x0000000000000000000000000000000000000000';
  
  if (from === ZERO_ADDRESS) {
    console.log(`MINT: Handle #${tokenId} minted to ${to}`);
    // Notify your system about new handle
  } else if (to === ZERO_ADDRESS) {
    console.log(`BURN: Handle #${tokenId} burned by ${from}`);
    // Handle was destroyed
  } else {
    console.log(`TRANSFER: Handle #${tokenId} moved from ${from} to ${to}`);
    // Ownership changed
  }
  
  // event.log contains full event data including transactionHash, blockNumber
  console.log(`   Block: ${event.log.blockNumber}, Tx: ${event.log.transactionHash}`);
});

console.log('Listening for Transfer events...');
```

**Filtering by Specific Wallet:**

```javascript
// Only listen for events involving a specific wallet
const WALLET_TO_WATCH = '0x3121d4d08EcDDd4dfd4cc4F1fD29B0A6E0b9649B';

// Filter for events where wallet is sender (outgoing transfers/burns)
const filterFrom = contract.filters.Transfer(WALLET_TO_WATCH, null, null);

// Filter for events where wallet is receiver (incoming transfers/mints)
const filterTo = contract.filters.Transfer(null, WALLET_TO_WATCH, null);

contract.on(filterFrom, (from, to, tokenId) => {
  console.log(`Outgoing: Handle #${tokenId} sent to ${to}`);
});

contract.on(filterTo, (from, to, tokenId) => {
  console.log(`Incoming: Handle #${tokenId} received from ${from}`);
});
```

**Python Example:**

```python
from web3 import Web3
import json

# WebSocket connection
w3 = Web3(Web3.WebsocketProvider('wss://bsc-rpc.publicnode.com'))

HANDLE_REGISTRY = '0xYourHandleRegistryProxyAddress'
TRANSFER_TOPIC = '0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef'
ZERO_ADDRESS = '0x0000000000000000000000000000000000000000'

def handle_event(event):
    from_addr = '0x' + event['topics'][1].hex()[26:]
    to_addr = '0x' + event['topics'][2].hex()[26:]
    token_id = int(event['topics'][3].hex(), 16)
    
    if from_addr == ZERO_ADDRESS:
        print(f"MINT: Handle #{token_id} minted to {to_addr}")
    elif to_addr == ZERO_ADDRESS:
        print(f"BURN: Handle #{token_id} burned by {from_addr}")
    else:
        print(f"TRANSFER: Handle #{token_id}: {from_addr} -> {to_addr}")

# Subscribe to Transfer events
log_filter = w3.eth.filter({
    'address': HANDLE_REGISTRY,
    'topics': [TRANSFER_TOPIC]
})

print("Listening for Transfer events...")
while True:
    for event in log_filter.get_new_entries():
        handle_event(event)
```

---

### Option B: Polling with eth_getLogs (Most Reliable)

WebSocket connections can drop, causing missed events. For production reliability, use `eth_getLogs` to poll for events and maintain a cursor (last processed block).

**Best Practice:** Combine WebSocket for speed with periodic `eth_getLogs` backfill for guaranteed delivery.

**JavaScript Example:**

```javascript
import { ethers } from 'ethers';

const provider = new ethers.JsonRpcProvider('https://bsc-dataseed.binance.org');
const HANDLE_REGISTRY = '0xYourHandleRegistryProxyAddress';

// Store this persistently (database, file, etc.)
let lastProcessedBlock = 45000000; // Start from a known block

async function pollForEvents() {
  const currentBlock = await provider.getBlockNumber();
  
  // Process in chunks of 1000 blocks (BSC limit)
  const CHUNK_SIZE = 1000;
  
  while (lastProcessedBlock < currentBlock) {
    const fromBlock = lastProcessedBlock + 1;
    const toBlock = Math.min(fromBlock + CHUNK_SIZE - 1, currentBlock);
    
    console.log(`Scanning blocks ${fromBlock} to ${toBlock}...`);
    
    const logs = await provider.getLogs({
      address: HANDLE_REGISTRY,
      topics: ['0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef'],
      fromBlock,
      toBlock
    });
    
    for (const log of logs) {
      const from = '0x' + log.topics[1].slice(26);
      const to = '0x' + log.topics[2].slice(26);
      const tokenId = BigInt(log.topics[3]);
      
      const ZERO = '0x0000000000000000000000000000000000000000';
      
      if (from === ZERO) {
        console.log(`MINT: Handle #${tokenId} to ${to} (block ${log.blockNumber})`);
      } else if (to === ZERO) {
        console.log(`BURN: Handle #${tokenId} by ${from} (block ${log.blockNumber})`);
      } else {
        console.log(`TRANSFER: #${tokenId}: ${from} -> ${to} (block ${log.blockNumber})`);
      }
    }
    
    lastProcessedBlock = toBlock;
    // Persist lastProcessedBlock to your database here
  }
}

// Poll every 10 seconds
setInterval(pollForEvents, 10000);
pollForEvents(); // Initial call
```

---

### Recommended: Hybrid Approach

For production systems, use both methods together:

1. **WebSocket** for near-instant notifications (when connection is active)
2. **eth_getLogs** polling as a backup to catch any missed events
3. **Store last processed block** in your database for recovery

```javascript
import { ethers } from 'ethers';

class HandleEventMonitor {
  constructor(proxyAddress, httpRpc, wsRpc) {
    this.proxyAddress = proxyAddress;
    this.httpProvider = new ethers.JsonRpcProvider(httpRpc);
    this.wsProvider = new ethers.WebSocketProvider(wsRpc);
    this.lastProcessedBlock = 0;
    this.contract = new ethers.Contract(
      proxyAddress,
      ['event Transfer(address indexed from, address indexed to, uint256 indexed tokenId)'],
      this.wsProvider
    );
  }

  async start(startBlock) {
    this.lastProcessedBlock = startBlock;
    
    // 1. Backfill any missed events
    await this.backfill();
    
    // 2. Start WebSocket listener for real-time
    this.contract.on('Transfer', this.handleEvent.bind(this));
    
    // 3. Periodic backfill every 60 seconds as safety net
    setInterval(() => this.backfill(), 60000);
    
    console.log('Handle event monitor started');
  }

  handleEvent(from, to, tokenId, event) {
    const ZERO = '0x0000000000000000000000000000000000000000';
    const type = from === ZERO ? 'MINT' : to === ZERO ? 'BURN' : 'TRANSFER';
    
    console.log(`[${type}] Handle #${tokenId}: ${from} -> ${to}`);
    
    // Send webhook, update database, etc.
    this.notifySubscribers({ type, from, to, tokenId: tokenId.toString() });
  }

  async backfill() {
    const currentBlock = await this.httpProvider.getBlockNumber();
    // ... same polling logic as Option B
  }

  notifySubscribers(event) {
    // Your notification logic here
  }
}

// Usage
const monitor = new HandleEventMonitor(
  '0xYourHandleRegistryProxyAddress',
  'https://bsc-dataseed.binance.org',
  'wss://bsc-rpc.publicnode.com'
);
monitor.start(45000000);
```

---

### Summary

| Method | Speed | Reliability | Use Case |
|--------|-------|-------------|----------|
| WebSocket only | Instant | Medium (can miss events) | Development, testing |
| eth_getLogs only | Delayed | High | Simple production setups |
| Hybrid (recommended) | Instant + reliable | Very high | Production systems |

**Key Takeaway:** Because handles are ERC-721 NFTs, every mint and transfer triggers the standard `Transfer` event. Monitor the HandleRegistry proxy contract's `Transfer` logs to track ownership changes. For near real-time updates use WebSocket `eth_subscribe`, and for full reliability store a last-seen block and backfill with `eth_getLogs` in chunks.

---

## Web Integration Guide

This section covers how to render handles in web applications using the CSS and JavaScript renderer.

### Using the DeBox Renderer (Recommended)

The provided renderer handles all the complexity:

```html
<link rel="stylesheet" href="https://debox-handle-marketplace.replit.app/api/embed/styles.css">
<script src="https://debox-handle-marketplace.replit.app/api/embed/renderer.js"></script>

<span id="handle"></span>

<script>
  // Fetch config and render
  fetch(`https://debox-handle-marketplace.replit.app/api/embed/${TOKEN}/config?env=testnet`, {
    headers: { 'x-api-key': API_KEY }
  })
    .then(res => res.json())
    .then(config => {
      DeBoxHandleRenderer.render(
        document.getElementById('handle'),
        'alice' + config.namespaceSuffix,
        config
      );
    });
</script>
```

The renderer automatically:
- Applies correct CSS classes for effects
- Sets inline styles for colors and gradients
- Configures CSS custom properties for glow
- Handles all baseColor types

### React Component Example

```jsx
import { useEffect, useRef, useState } from 'react';

function DeBoxHandle({ tokenAddress, handleName, env = 'testnet' }) {
  const spanRef = useRef(null);
  const [config, setConfig] = useState(null);
  
  useEffect(() => {
    // Load CSS
    const link = document.createElement('link');
    link.rel = 'stylesheet';
    link.href = 'https://debox-handle-marketplace.replit.app/api/embed/styles.css';
    document.head.appendChild(link);
    return () => document.head.removeChild(link);
  }, []);
  
  useEffect(() => {
    fetch(`https://debox-handle-marketplace.replit.app/api/embed/${tokenAddress}/config?env=${env}`, {
      headers: { 'x-api-key': API_KEY }
    })
      .then(res => res.json())
      .then(setConfig);
  }, [tokenAddress, env]);
  
  useEffect(() => {
    if (config && spanRef.current && window.DeBoxHandleRenderer) {
      window.DeBoxHandleRenderer.render(spanRef.current, handleName, config);
    }
  }, [config, handleName]);
  
  return <span ref={spanRef} className="font-bold text-2xl">{handleName}</span>;
}

// Usage
<DeBoxHandle 
  tokenAddress="0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7"
  handleName="alice.community"
  env="testnet"
/>
```

### Vue Component Example

```vue
<template>
  <span ref="handleEl" class="font-bold text-2xl">{{ handleName }}</span>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue';

const props = defineProps({
  tokenAddress: String,
  handleName: String,
  env: { type: String, default: 'testnet' }
});

const handleEl = ref(null);
const config = ref(null);

onMounted(async () => {
  // Load CSS and renderer
  const link = document.createElement('link');
  link.rel = 'stylesheet';
  link.href = 'https://debox-handle-marketplace.replit.app/api/embed/styles.css';
  document.head.appendChild(link);
  
  const script = document.createElement('script');
  script.src = 'https://debox-handle-marketplace.replit.app/api/embed/renderer.js';
  document.head.appendChild(script);
  
  // Fetch config
  const res = await fetch(
    `https://debox-handle-marketplace.replit.app/api/embed/${props.tokenAddress}/config?env=${props.env}`,
    { headers: { 'x-api-key': API_KEY } }
  );
  config.value = await res.json();
});

watch([config, () => props.handleName], () => {
  if (config.value && handleEl.value && window.DeBoxHandleRenderer) {
    window.DeBoxHandleRenderer.render(handleEl.value, props.handleName, config.value);
  }
});
</script>
```

---

## Native Mobile SDK Guide

This section covers how to render handles in native iOS and Android applications.

### Configuration Types

The native endpoint returns different configurations based on the community's customization settings:

1. **Base Standard Color (Solid)** - Simple solid color, minimal response
2. **Prebuilt Effect** - Complete visual package with gradients, animations, glow, and shadows

### Swift (iOS) Implementation

**Data Models:**

```swift
import UIKit

struct NativeConfig: Codable {
    let version: String
    let tokenAddress: String
    let community: Community
    let configUpdatedAt: String
    let textStyle: TextStyle
    let animation: Animation?
    let glow: Glow?
    let shadow: Shadow?
}

struct Community: Codable {
    let name: String
    let slug: String
    let namespaceSuffix: String
}

struct TextStyle: Codable {
    let type: String
    let color: String?
    let colors: [String]?
    let direction: String?
}

struct Animation: Codable {
    let type: String
    let duration: Int
    let `repeat`: String
    let easing: String
    let keyframes: [Keyframe]
}

struct Keyframe: Codable {
    let position: Double
    let hueRotation: Double?
    let opacity: Double?
    let scale: Double?
}

struct Glow: Codable {
    let size: Int
    let intensity: Int
    let color: String
    let colorSource: String
}

struct Shadow: Codable {
    let layers: [ShadowLayer]
}

struct ShadowLayer: Codable {
    let color: String
    let blur: Int
    let opacity: Double
}
```

**Renderer:**

```swift
class HandleRenderer {
    
    func renderHandle(label: UILabel, handleName: String, config: NativeConfig) {
        label.text = handleName + config.community.namespaceSuffix
        
        applyTextStyle(to: label, textStyle: config.textStyle)
        
        if let shadow = config.shadow {
            applyShadow(to: label, shadow: shadow)
        }
        
        if let animation = config.animation {
            applyAnimation(to: label, animation: animation)
        }
    }
    
    private func applyTextStyle(to label: UILabel, textStyle: TextStyle) {
        if textStyle.type == "solid", let color = textStyle.color {
            label.textColor = UIColor(hex: color)
        } else if textStyle.type == "gradient", let colors = textStyle.colors {
            applyGradientText(to: label, colors: colors, direction: textStyle.direction)
        }
    }
    
    private func applyShadow(to label: UILabel, shadow: Shadow) {
        if let firstLayer = shadow.layers.first {
            label.layer.shadowColor = UIColor(hex: firstLayer.color)?.cgColor
            label.layer.shadowRadius = CGFloat(firstLayer.blur)
            label.layer.shadowOpacity = Float(firstLayer.opacity)
            label.layer.shadowOffset = .zero
        }
    }
    
    private func applyAnimation(to label: UILabel, animation: Animation) {
        if animation.type == "hueShift" {
            startHueShiftAnimation(on: label, animation: animation)
        }
    }
}
```

### Kotlin (Android) Implementation

**Data Models:**

```kotlin
import android.graphics.*
import android.widget.TextView
import android.animation.ValueAnimator
import android.view.animation.LinearInterpolator

data class NativeConfig(
    val version: String,
    val tokenAddress: String,
    val community: Community,
    val configUpdatedAt: String,
    val textStyle: TextStyle,
    val animation: Animation? = null,
    val glow: Glow? = null,
    val shadow: Shadow? = null
)

data class Community(
    val name: String,
    val slug: String,
    val namespaceSuffix: String
)

data class TextStyle(
    val type: String,
    val color: String? = null,
    val colors: List<String>? = null,
    val direction: String? = null
)

data class Animation(
    val type: String,
    val duration: Int,
    val repeat: String,
    val easing: String,
    val keyframes: List<Keyframe>
)

data class Keyframe(
    val position: Double,
    val hueRotation: Double? = null,
    val opacity: Double? = null,
    val scale: Double? = null
)

data class Glow(
    val size: Int,
    val intensity: Int,
    val color: String,
    val colorSource: String
)

data class Shadow(
    val layers: List<ShadowLayer>
)

data class ShadowLayer(
    val color: String,
    val blur: Int,
    val opacity: Double
)
```

**Renderer:**

```kotlin
class HandleRenderer {
    
    fun renderHandle(textView: TextView, handleName: String, config: NativeConfig) {
        textView.text = handleName + config.community.namespaceSuffix
        
        applyTextStyle(textView, config.textStyle)
        config.shadow?.let { applyShadow(textView, it) }
        config.animation?.let { applyAnimation(textView, it) }
    }
    
    private fun applyTextStyle(textView: TextView, textStyle: TextStyle) {
        when (textStyle.type) {
            "solid" -> {
                textStyle.color?.let { 
                    textView.setTextColor(Color.parseColor(it))
                }
            }
            "gradient" -> {
                textStyle.colors?.let { colors ->
                    val shader = createGradientShader(textView, colors, textStyle.direction)
                    textView.paint.shader = shader
                }
            }
        }
    }
    
    private fun createGradientShader(
        textView: TextView, 
        colors: List<String>, 
        direction: String?
    ): Shader {
        val colorInts = colors.map { Color.parseColor(it) }.toIntArray()
        val width = textView.paint.measureText(textView.text.toString())
        val height = textView.textSize
        
        return when (direction) {
            "toRight" -> LinearGradient(0f, 0f, width, 0f, colorInts, null, Shader.TileMode.CLAMP)
            "toBottom" -> LinearGradient(0f, 0f, 0f, height, colorInts, null, Shader.TileMode.CLAMP)
            "radial" -> RadialGradient(width/2, height/2, width/2, colorInts, null, Shader.TileMode.CLAMP)
            else -> LinearGradient(0f, 0f, width, 0f, colorInts, null, Shader.TileMode.CLAMP)
        }
    }
    
    private fun applyShadow(textView: TextView, shadow: Shadow) {
        shadow.layers.firstOrNull()?.let { layer ->
            val color = Color.parseColor(layer.color)
            val alpha = (layer.opacity * 255).toInt().coerceIn(0, 255)
            val shadowColor = Color.argb(alpha, Color.red(color), Color.green(color), Color.blue(color))
            textView.setShadowLayer(layer.blur.toFloat(), 0f, 0f, shadowColor)
        }
    }
    
    private fun applyAnimation(textView: TextView, animation: Animation) {
        if (animation.type == "hueShift") {
            startHueShiftAnimation(textView, animation)
        }
    }
    
    private fun startHueShiftAnimation(textView: TextView, animation: Animation) {
        ValueAnimator.ofFloat(0f, 1f).apply {
            duration = animation.duration.toLong()
            repeatCount = if (animation.repeat == "infinite") ValueAnimator.INFINITE else 0
            interpolator = LinearInterpolator()
            
            addUpdateListener { animator ->
                val progress = animator.animatedValue as Float
                val hueRotation = interpolateKeyframes(animation.keyframes, progress)
                applyHueRotation(textView, hueRotation)
            }
        }.start()
    }
    
    private fun interpolateKeyframes(keyframes: List<Keyframe>, progress: Float): Float {
        var prevKeyframe = keyframes.first()
        var nextKeyframe = keyframes.last()
        
        for (i in 0 until keyframes.size - 1) {
            if (progress >= keyframes[i].position && progress <= keyframes[i + 1].position) {
                prevKeyframe = keyframes[i]
                nextKeyframe = keyframes[i + 1]
                break
            }
        }
        
        val localProgress = if (nextKeyframe.position == prevKeyframe.position) 0f
            else ((progress - prevKeyframe.position) / (nextKeyframe.position - prevKeyframe.position)).toFloat()
        
        val prevHue = prevKeyframe.hueRotation ?: 0.0
        val nextHue = nextKeyframe.hueRotation ?: 0.0
        
        return (prevHue + (nextHue - prevHue) * localProgress).toFloat()
    }
}
```

---

## Configuration Reference

### Web Config Fields (`/config`)

| Field | Type | Description |
|-------|------|-------------|
| `tokenAddress` | string | Community's token contract address |
| `communitySlug` | string | URL-safe community identifier |
| `communityName` | string | Display name of the community |
| `namespaceSuffix` | string | Handle suffix (e.g., `.debox`) |
| `baseColor` | object | Color configuration |
| `effect` | string\|null | Individual effect name |
| `prebuiltEffect` | string\|null | Prebuilt effect name |
| `glow` | object | Glow effect settings |
| `logoUrl` | string\|null | Community logo URL |
| `configUpdatedAt` | string | ISO timestamp of last change |
| `embedVersion` | string | Current embed assets version |

### baseColor Types

| Type | Value | Description |
|------|-------|-------------|
| `black` | `"#000000"` | Default black text |
| `basic` | `"#hexcolor"` | Predefined palette color |
| `custom` | `"#hexcolor"` | Custom hex color |
| `gradient` | `{color1, color2, gradientType}` | Two-color gradient |
| `prebuilt` | `"effectName"` | Prebuilt effect |

### Gradient Types

| Value | Description |
|-------|-------------|
| `linear-horizontal` | Left to right |
| `linear-vertical` | Top to bottom |
| `linear-diagonal` | 135 degree angle |
| `radial` | Circular gradient |

### Available Effects

**Individual Effects** (`effect` field):
- `shimmer`, `pulse`, `electric`, `flicker`
- `prism-shift`, `radiant-burst`, `chromatic-shift`
- `spectrum-cycle`, `color-wave`

**Prebuilt Effects** (`prebuiltEffect` field):
- `rainbow`, `fire`, `ice`, `neon`
- `holographic`, `cosmic`, `matrix`
- `diamond`, `glowing`

### Native Config Fields (`/native`)

#### textStyle Object

| Field | Type | Description |
|-------|------|-------------|
| `type` | `"solid"` \| `"gradient"` | Text coloring type |
| `color` | string | Hex color (solid only) |
| `colors` | string[] | Color array (gradient only) |
| `direction` | string | `"toRight"`, `"toBottom"`, `"toBottomRight"`, `"radial"` |

#### animation Object

| Field | Type | Description |
|-------|------|-------------|
| `type` | string | Animation type: `"hueShift"`, `"pulse"`, etc. |
| `duration` | number | Duration in milliseconds |
| `repeat` | `"infinite"` \| number | Repeat count |
| `easing` | string | `"linear"`, `"easeInOut"`, etc. |
| `keyframes` | array | Array of keyframe objects |

#### Keyframe Object

| Field | Type | Description |
|-------|------|-------------|
| `position` | number | 0.0 to 1.0 (animation progress) |
| `hueRotation` | number | Degrees to rotate hue (0-360) |
| `opacity` | number | 0.0 to 1.0 |
| `scale` | number | Scale factor (1.0 = normal) |
| `translateX` | number | Horizontal translation (pixels) |
| `translateY` | number | Vertical translation (pixels) |

#### glow Object

| Field | Type | Description |
|-------|------|-------------|
| `size` | number | Glow radius in pixels |
| `intensity` | number | Glow intensity (100 = normal) |
| `color` | string | Hex color for glow |
| `colorSource` | `"base"` \| `"custom"` | Color source |

#### shadow Object

| Field | Type | Description |
|-------|------|-------------|
| `layers` | array | Array of shadow layers |
| `layers[].color` | string | Hex color |
| `layers[].blur` | number | Blur radius in pixels |
| `layers[].opacity` | number | Shadow opacity |

---

## Caching & Change Detection

### Cache Durations

| Endpoint | Cache Duration | Notes |
|----------|----------------|-------|
| `/version` | 1 hour | Rarely changes |
| `/styles.css` | 5 min / Forever | Use `?v=X.X.X` for immutable caching |
| `/renderer.js` | 5 min / Forever | Use `?v=X.X.X` for immutable caching |
| `/communities` | 5 minutes | Community list |
| `/changes` | 1 minute | Change detection |
| `/:token/config` | 1 minute | Supports ETag |
| `/:token/native` | 1 minute | Supports ETag |

### Recommended Strategy

**For Web Apps:**
1. Pin assets to a specific version using `?v=X.X.X` for immutable caching
2. Use the `configUpdatedAt` field or ETag to detect config changes

**For Native Apps (Backend Polling):**
1. Poll `/changes` every 5 minutes
2. Use `serverTime` from previous response as `since` for next poll
3. Refresh cached configs only for communities in the `changes` array

---

## Testing

### Testnet Token Addresses

| Token Address | Description |
|---------------|-------------|
| `0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7` | Primary testnet community |
| `0x9244212403a2e827cadca1f6fb68b43bc0c7a41f` | Testnet community |
| `0xf762407aec88afd53be1f6d94a6c98ff5c6e4a25` | Testnet community |
| `0x4b9FF95124d5bD4dc39334372373c005D6b9C859` | Testnet community |

### Test Script

```bash
BASE_URL="https://debox-handle-marketplace.replit.app"
API_KEY="YOUR_API_KEY"

echo "=== List Communities ==="
curl -s -H "x-api-key: $API_KEY" "$BASE_URL/api/embed/communities?env=testnet"

echo -e "\n\n=== Check Changes ==="
curl -s -H "x-api-key: $API_KEY" "$BASE_URL/api/embed/changes?env=testnet"

echo -e "\n\n=== Get Web Config ==="
curl -s -H "x-api-key: $API_KEY" \
  "$BASE_URL/api/embed/0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7/config?env=testnet"

echo -e "\n\n=== Get Native Config ==="
curl -s -H "x-api-key: $API_KEY" \
  "$BASE_URL/api/embed/0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7/native?env=testnet"
```

---

## Error Handling

### Error Response Format

```json
{
  "error": "Error message description",
  "tokenAddress": "0x...",
  "environment": "testnet"
}
```

### HTTP Status Codes

| Status | Meaning |
|--------|---------|
| 400 | Invalid request (bad token address, invalid environment, invalid timestamp) |
| 401 | Authentication required or invalid API key |
| 404 | Community or resource not found |
| 500 | Server error |

---

## Support

For questions or issues with the Embed API, please contact the DeBox team.
