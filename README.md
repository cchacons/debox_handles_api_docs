# DeBox Handles - Embed API Documentation

The Embed API allows you to render DeBox handles with their customization effects on your own website or application. This includes animated text effects, gradients, glow effects, and community branding.

## Table of Contents

1. [Quick Start](#quick-start)
2. [Authentication](#authentication)
3. [API Endpoints](#api-endpoints)
4. [Configuration Response](#configuration-response)
5. [Rendering Handles](#rendering-handles)
6. [Real-World Examples](#real-world-examples)
7. [Caching Strategy](#caching-strategy)
8. [Testing with cURL](#testing-with-curl)

---

## Quick Start

### Step 1: Test the API with cURL

First, verify the API is working by fetching a community configuration:

```bash
curl -s -H "x-api-key: DEBOX_HANDLE_API_KEY" \
  "https://debox-handle-marketplace.replit.app/api/embed/0x9244212403a2e827cadca1f6fb68b43bc0c7a41f/config?env=testnet" | jq
```

You should see a JSON response with the community's customization settings including `namespaceSuffix`, `baseColor`, `effect`, and `glow` configuration.

### Step 2: Try the HTML Demo

Save the following code as `test.html` and open it in your web browser to see a styled DeBox handle rendered:

```html
<!DOCTYPE html>
<html>
<head>
  <title>DeBox Handle Demo</title>
</head>
<body style="background:#fff; padding:40px; font-family:sans-serif;">
  <h1 style="color:black;">DeBox Handle Demo</h1>
  <div style="font-size:32px; margin:40px 0;">
    <span id="handle">loading...</span>
  </div>

  <script>
    const API = 'https://debox-handle-marketplace.replit.app/api/embed';
    const KEY = 'DEBOX_HANDLE_API_KEY';
    // Available testnet community tokens:
    // const TOKEN = '0x9244212403a2e827cadca1f6fb68b43bc0c7a41f';
    // const TOKEN = '0xf762407aec88afd53be1f6d94a6c98ff5c6e4a25';
    // const TOKEN = '0x4b9FF95124d5bD4dc39334372373c005D6b9C859';
    const TOKEN = '0x9244212403a2e827cadca1f6fb68b43bc0c7a41f';
    const headers = { 'x-api-key': KEY };

    Promise.all([
      fetch(`${API}/styles.css`, { headers }).then(r => r.text()),
      fetch(`${API}/renderer.js`, { headers }).then(r => r.text()),
      fetch(`${API}/${TOKEN}/config?env=testnet`, { headers }).then(r => r.json())
    ]).then(([css, js, config]) => {
      // Inject CSS
      const style = document.createElement('style');
      style.textContent = css;
      document.head.appendChild(style);
      
      eval(js);
      
      // Render handle
      const el = document.getElementById('handle');
      el.textContent = 'carlos' + (config.namespaceSuffix || '');
      DeBoxHandleRenderer.applyToElement(el, config);
    }).catch(e => {
      document.getElementById('handle').textContent = 'Error: ' + e.message;
    });
  </script>
</body>
</html>
```

This demo fetches the CSS, JavaScript renderer, and community configuration, then applies the styling to display a handle with the community's active effects.

### Step 3: Basic Integration

For a simpler integration approach, you can load assets directly via script/link tags:

```html
<!DOCTYPE html>
<html>
<head>
  <!-- Step 1: Load the effect CSS -->
  <link rel="stylesheet" href="https://debox-handle-marketplace.replit.app/api/embed/styles.css">
</head>
<body>
  <!-- Step 2: Container for the handle -->
  <span id="my-handle"></span>

  <!-- Step 3: Load renderer and apply styles -->
  <script src="https://debox-handle-marketplace.replit.app/api/embed/renderer.js"></script>
  <script>
    // Fetch config for a testnet community token
    fetch('https://debox-handle-marketplace.replit.app/api/embed/0x9244212403a2e827cadca1f6fb68b43bc0c7a41f/config?env=testnet')
      .then(res => res.json())
      .then(config => {
        // Render the handle with effects
        const element = document.getElementById('my-handle');
        DeBoxHandleRenderer.render(element, 'alice' + config.namespaceSuffix, config);
      });
  </script>
</body>
</html>
```

---

## Authentication

The Embed API uses API keys for authentication. API keys allow you to track usage and access protected endpoints.

### Obtaining an API Key

To get an API key for your integration:

1. **Contact the DeBox team** to request an API key
2. Provide details about your use case and expected usage volume
3. The team will generate a key and provide it to you securely

### Using Your API Key

Include the API key in your requests using one of these methods:

**Option 1: x-api-key Header (Recommended)**
```bash
curl -H "x-api-key: dbx_abc123..." \
  "https://debox-handle-marketplace.replit.app/api/embed/0x.../config"
```

**Option 2: Authorization Bearer Token**
```bash
curl -H "Authorization: Bearer dbx_abc123..." \
  "https://debox-handle-marketplace.replit.app/api/embed/0x.../config"
```

### JavaScript Example

```javascript
// Using fetch with API key
const API_KEY = 'dbx_abc123...';

fetch('https://debox-handle-marketplace.replit.app/api/embed/0x.../config', {
  headers: {
    'x-api-key': API_KEY
  }
})
  .then(res => res.json())
  .then(config => {
    console.log('Community config:', config);
  });
```

### API Key Format

- API keys start with the prefix `dbx_`
- Example: `dbx_a1b2c3d4e5f6...`
- Keys are 32+ characters long
- Store your key securely - it cannot be retrieved after creation

### Rate Limiting & Usage

- API key usage is tracked automatically
- Contact the DeBox team if you need higher rate limits

### Error Responses

Authentication errors return JSON with both `error` and `message` fields:

```json
{
  "error": "API key required",
  "message": "Provide API key via x-api-key header or Authorization: Bearer <key>"
}
```

| Status | Error | Message |
|--------|-------|---------|
| 401 | `API key required` | `Provide API key via x-api-key header or Authorization: Bearer <key>` |
| 401 | `Invalid API key` | `The provided API key is invalid or revoked` |
| 500 | `Authentication error` | (Server error during validation) |

---

## API Endpoints

All endpoints are available at `https://debox-handle-marketplace.replit.app/api/embed/`

### GET /api/embed/version

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

**cURL Example:**
```bash
curl -s https://debox-handle-marketplace.replit.app/api/embed/version | jq
```

---

### GET /api/embed/styles.css

Returns the CSS file containing all effect animations and styles.

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `v` | string | No | Specific version (defaults to latest) |

**Headers Returned:**
- `Content-Type: text/css`
- `X-Embed-Version: <version>`
- `Cache-Control: public, max-age=300` (or `max-age=31536000, immutable` for versioned requests)

**cURL Examples:**
```bash
# Get latest CSS
curl -s https://debox-handle-marketplace.replit.app/api/embed/styles.css -o styles.css

# Get specific version
curl -s "https://debox-handle-marketplace.replit.app/api/embed/styles.css?v=0.0.1" -o styles.css

# Check headers
curl -I https://debox-handle-marketplace.replit.app/api/embed/styles.css
```

---

### GET /api/embed/renderer.js

Returns the JavaScript renderer for applying handle styles.

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `v` | string | No | Specific version (defaults to latest) |

**Headers Returned:**
- `Content-Type: application/javascript`
- `X-Embed-Version: <version>`

**cURL Examples:**
```bash
# Get latest renderer
curl -s https://debox-handle-marketplace.replit.app/api/embed/renderer.js -o renderer.js

# Get specific version
curl -s "https://debox-handle-marketplace.replit.app/api/embed/renderer.js?v=0.0.1" -o renderer.js
```

---

### GET /api/embed/:tokenAddress/config

Returns the active customization configuration for a community identified by its token address.

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `tokenAddress` | string | Yes | The community's token contract address (0x...) |

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `env` | string | No | Environment: `mock`, `testnet`, or `mainnet` (defaults to server environment) |

**cURL Examples:**
```bash
# Get config for a testnet community
curl -s -H "x-api-key: DEBOX_HANDLE_API_KEY" \
  "https://debox-handle-marketplace.replit.app/api/embed/0x9244212403a2e827cadca1f6fb68b43bc0c7a41f/config?env=testnet" | jq

# Try other testnet communities
curl -s -H "x-api-key: DEBOX_HANDLE_API_KEY" \
  "https://debox-handle-marketplace.replit.app/api/embed/0x9244212403a2e827cadca1f6fb68b43bc0c7a41f/config?env=testnet" | jq

# Get config for mainnet community
curl -s -H "x-api-key: DEBOX_HANDLE_API_KEY" \
  "https://debox-handle-marketplace.replit.app/api/embed/0xYourTokenAddress/config?env=mainnet" | jq

# Conditional request with ETag
curl -s -H "x-api-key: DEBOX_HANDLE_API_KEY" \
  "https://debox-handle-marketplace.replit.app/api/embed/0x9244212403a2e827cadca1f6fb68b43bc0c7a41f/config?env=testnet" \
  -H "If-None-Match: \"abc123\"" -w "%{http_code}"
```

---

## Configuration Response

The config endpoint returns a comprehensive customization configuration:

```json
{
  "tokenAddress": "0x9244212403a2e827cadca1f6fb68b43bc0c7a41f",
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

### Field Reference

| Field | Type | Description |
|-------|------|-------------|
| `tokenAddress` | string | The community's token contract address |
| `communitySlug` | string | URL-safe community identifier |
| `communityName` | string | Display name of the community |
| `namespaceSuffix` | string | The handle suffix (e.g., `.debox`) |
| `baseColor` | object | Color configuration (see below) |
| `effect` | string\|null | Individual effect name (e.g., `shimmer`, `pulse`) |
| `prebuiltEffect` | string\|null | Prebuilt effect name (e.g., `rainbow`, `fire`) |
| `glow` | object | Glow effect settings |
| `logoUrl` | string\|null | Community logo URL |
| `configUpdatedAt` | string | ISO timestamp of last config change |
| `embedVersion` | string | Current embed assets version |

### baseColor Types

| Type | Value | Description |
|------|-------|-------------|
| `black` | `"#000000"` | Default black text |
| `basic` | `"#hexcolor"` | Predefined palette color |
| `custom` | `"#hexcolor"` | Custom hex color |
| `gradient` | `{color1, color2, gradientType}` | Two-color gradient |
| `prebuilt` | `"effectName"` | Prebuilt effect (overrides other settings) |

### Gradient Types

When `baseColor.type` is `gradient`, the value contains:
```json
{
  "color1": "#FF6B6B",
  "color2": "#4ECDC4",
  "gradientType": "linear-horizontal"
}
```

Available `gradientType` values:
- `linear-horizontal` - Left to right
- `linear-vertical` - Top to bottom
- `linear-diagonal` - 135 degree angle
- `radial` - Circular gradient

### Available Effects

**Individual Effects** (applied with `effect` field):
- `shimmer` - Shimmering light sweep
- `pulse` - Pulsing brightness
- `electric` - Electric crackling
- `flicker` - Flickering light
- `prism-shift` - Color prism shifting
- `radiant-burst` - Radiating burst
- `chromatic-shift` - Color chromatic shifting
- `spectrum-cycle` - Full spectrum cycling
- `color-wave` - Wave color animation

**Prebuilt Effects** (applied with `prebuiltEffect` field):
- `rainbow` - Rainbow color cycling
- `fire` - Burning fire effect
- `ice` - Frozen ice effect
- `neon` - Neon glow
- `holographic` - Holographic shimmer
- `cosmic` - Cosmic space effect
- `matrix` - Matrix digital rain
- `diamond` - Diamond sparkle
- `glowing` - Soft glow pulsing

---

## Rendering Handles

### Option 1: Using the DeBox Renderer (Recommended)

The provided renderer handles all the complexity of applying styles:

```html
<script src="https://debox-handle-marketplace.replit.app/api/embed/renderer.js"></script>
<script>
  const element = document.getElementById('handle-container');
  
  // Render with config from API
  DeBoxHandleRenderer.render(element, 'username.community', config);
</script>
```

The renderer automatically:
- Applies correct CSS classes for effects
- Sets inline styles for colors and gradients
- Configures CSS custom properties for glow
- Handles all baseColor types

### Option 2: Manual Implementation

For full control, you can implement rendering manually. Here's how our internal components work:

#### Step 1: Determine CSS Classes

```javascript
function getHandleClasses(config) {
  const classes = ['font-bold'];
  
  // Prebuilt effects use special class names
  if (config.baseColor?.type === 'prebuilt') {
    const effectName = config.baseColor.value;
    // Convert camelCase to kebab-case
    const kebabName = effectName.replace(/([a-z0-9])([A-Z])/g, '$1-$2').toLowerCase();
    classes.push(`${kebabName}-text`);
    return classes;
  }
  
  // Individual effects
  if (config.effect) {
    const kebabEffect = config.effect.replace(/([a-z0-9])([A-Z])/g, '$1-$2').toLowerCase();
    classes.push(`effect-${kebabEffect}`);
    
    // Add glow class if enabled
    if (config.glow?.enabled) {
      classes.push('with-glow');
      
      // Add gradient-version class for compatible effects
      const gradientCompatible = ['shimmer', 'electric', 'flicker', 'pulse', 
        'prism-shift', 'radiant-burst', 'chromatic-shift', 'spectrum-cycle', 'color-wave'];
      if (config.baseColor?.type === 'gradient' && gradientCompatible.includes(kebabEffect)) {
        classes.push('gradient-version');
      }
    }
  } else if (config.glow?.enabled) {
    // Glow without effect
    if (config.baseColor?.type === 'gradient') {
      classes.push('gradient-glow');
    } else {
      classes.push('effect-base-glow');
    }
  }
  
  return classes;
}
```

#### Step 2: Apply Inline Styles

```javascript
function getHandleStyles(config) {
  const styles = {};
  
  // Color variant CSS variables (needed for effects)
  if (config.baseColor) {
    const baseHex = getBaseColorHex(config.baseColor);
    styles['--effect-primary'] = baseHex;
    styles['--effect-light'] = lighten(baseHex, 20);
    styles['--effect-dark'] = darken(baseHex, 20);
    styles['--effect-accent'] = saturate(baseHex, 20);
  }
  
  // Text color
  if (config.baseColor?.type === 'custom' || config.baseColor?.type === 'basic') {
    styles.color = config.baseColor.value;
  } else if (config.baseColor?.type === 'gradient') {
    const { color1, color2, gradientType } = config.baseColor.value;
    let gradient;
    switch (gradientType) {
      case 'linear-horizontal':
        gradient = `linear-gradient(to right, ${color1}, ${color2})`;
        break;
      case 'linear-vertical':
        gradient = `linear-gradient(to bottom, ${color1}, ${color2})`;
        break;
      case 'radial':
        gradient = `radial-gradient(circle, ${color1}, ${color2})`;
        break;
      default:
        gradient = `linear-gradient(135deg, ${color1}, ${color2})`;
    }
    styles.backgroundImage = gradient;
    styles.WebkitBackgroundClip = 'text';
    styles.WebkitTextFillColor = 'transparent';
    styles.backgroundClip = 'text';
  }
  
  // Glow styles
  if (config.glow?.enabled) {
    let glowColor = config.glow.color;
    if (config.glow.colorSource === 'base') {
      glowColor = getBaseColorHex(config.baseColor);
    }
    styles['--glow-color'] = glowColor;
    styles['--glow-size'] = `${config.glow.size}px`;
    styles['--glow-intensity'] = config.glow.intensity / 100;
  }
  
  return styles;
}

function getBaseColorHex(baseColor) {
  if (!baseColor) return '#000000';
  if (baseColor.type === 'black') return '#000000';
  if (baseColor.type === 'basic' || baseColor.type === 'custom') {
    return baseColor.value;
  }
  if (baseColor.type === 'gradient') {
    return baseColor.value.color1;
  }
  return '#000000';
}
```

#### Step 3: Render the Element

```javascript
function renderHandle(element, handleName, config) {
  const classes = getHandleClasses(config);
  const styles = getHandleStyles(config);
  
  element.className = classes.join(' ');
  Object.assign(element.style, styles);
  element.textContent = handleName;
}
```

---

## Real-World Examples

### Example 1: Community Card (Like /communities page)

This example shows how to display a community's namespace suffix with its active styling, similar to our Communities page:

```html
<div class="community-card">
  <img src="" alt="Logo" id="community-logo" class="w-10 h-10 rounded-full">
  <div>
    <span class="text-sm text-gray-500" id="community-name"></span>
    <span id="namespace-suffix" class="text-2xl font-bold"></span>
  </div>
</div>

<script src="https://debox-handle-marketplace.replit.app/api/embed/renderer.js"></script>
<script>
  const API_KEY = 'DEBOX_HANDLE_API_KEY';
  const TOKEN_ADDRESS = '0x9244212403a2e827cadca1f6fb68b43bc0c7a41f';
  
  fetch(`https://debox-handle-marketplace.replit.app/api/embed/${TOKEN_ADDRESS}/config?env=testnet`, {
    headers: { 'x-api-key': API_KEY }
  })
    .then(res => res.json())
    .then(config => {
      // Set community info
      document.getElementById('community-name').textContent = config.communityName;
      if (config.logoUrl) {
        document.getElementById('community-logo').src = config.logoUrl;
      }
      
      // Render the namespace suffix with effects
      const suffixEl = document.getElementById('namespace-suffix');
      DeBoxHandleRenderer.render(suffixEl, config.namespaceSuffix, config);
    });
</script>
```

### Example 2: Handle Preview (Like Mint Flow)

Display a full handle name with user's chosen name and community suffix:

```html
<div class="handle-preview" style="padding: 24px; background: #111; border-radius: 8px;">
  <span id="full-handle" class="text-4xl"></span>
</div>

<script src="https://debox-handle-marketplace.replit.app/api/embed/renderer.js"></script>
<script>
  const API_KEY = 'DEBOX_HANDLE_API_KEY';
  const userName = 'alice';  // User's chosen handle name
  const TOKEN_ADDRESS = '0x9244212403a2e827cadca1f6fb68b43bc0c7a41f';
  
  fetch(`https://debox-handle-marketplace.replit.app/api/embed/${TOKEN_ADDRESS}/config?env=testnet`, {
    headers: { 'x-api-key': API_KEY }
  })
    .then(res => res.json())
    .then(config => {
      const fullHandle = userName + config.namespaceSuffix;
      const element = document.getElementById('full-handle');
      DeBoxHandleRenderer.render(element, fullHandle, config);
    });
</script>
```

### Example 3: React Component

```jsx
import { useEffect, useRef, useState } from 'react';

const API_KEY = 'DEBOX_HANDLE_API_KEY';

function DeBoxHandle({ tokenAddress, handleName, env = 'testnet' }) {
  const spanRef = useRef(null);
  const [config, setConfig] = useState(null);
  
  // Load CSS once
  useEffect(() => {
    const link = document.createElement('link');
    link.rel = 'stylesheet';
    link.href = 'https://debox-handle-marketplace.replit.app/api/embed/styles.css';
    document.head.appendChild(link);
    return () => document.head.removeChild(link);
  }, []);
  
  // Fetch config
  useEffect(() => {
    fetch(`https://debox-handle-marketplace.replit.app/api/embed/${tokenAddress}/config?env=${env}`, {
      headers: { 'x-api-key': API_KEY }
    })
      .then(res => res.json())
      .then(setConfig)
      .catch(console.error);
  }, [tokenAddress, env]);
  
  // Apply styles when config loads
  useEffect(() => {
    if (!config || !spanRef.current) return;
    
    // Use the renderer if loaded, otherwise apply manually
    if (window.DeBoxHandleRenderer) {
      window.DeBoxHandleRenderer.render(spanRef.current, handleName, config);
    }
  }, [config, handleName]);
  
  return <span ref={spanRef} className="font-bold text-2xl">{handleName}</span>;
}

// Usage with testnet community tokens
<DeBoxHandle 
  tokenAddress="0x9244212403a2e827cadca1f6fb68b43bc0c7a41f"
  handleName="alice.community"
  env="testnet"
/>
```

### Example 4: Vue Component

```vue
<template>
  <span ref="handleEl" class="font-bold text-2xl">{{ handleName }}</span>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue';

const API_KEY = 'DEBOX_HANDLE_API_KEY';

const props = defineProps({
  tokenAddress: String,
  handleName: String,
  env: { type: String, default: 'testnet' }
});

const handleEl = ref(null);
const config = ref(null);

onMounted(async () => {
  // Load CSS
  const link = document.createElement('link');
  link.rel = 'stylesheet';
  link.href = 'https://debox-handle-marketplace.replit.app/api/embed/styles.css';
  document.head.appendChild(link);
  
  // Load renderer
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

## Caching Strategy

The API uses intelligent caching to balance freshness with performance:

| Endpoint | Cache Duration | Notes |
|----------|----------------|-------|
| `/version` | 1 hour | Version info rarely changes |
| `/styles.css` | 5 min (latest) / Forever (versioned) | Use `?v=X.X.X` for immutable caching |
| `/renderer.js` | 5 min (latest) / Forever (versioned) | Use `?v=X.X.X` for immutable caching |
| `/:token/config` | 1 minute | Supports ETag for conditional requests |

### Recommended Approach

1. **For production**: Pin to a specific version using the `v` query parameter
2. **For development**: Use the latest (no `v` parameter)
3. **For config**: Use the `configUpdatedAt` field to detect changes

```javascript
// Check if cached config is stale
const lastKnown = localStorage.getItem('configUpdatedAt');
fetch(`/api/embed/${token}/config?env=mainnet`)
  .then(res => res.json())
  .then(config => {
    if (config.configUpdatedAt !== lastKnown) {
      localStorage.setItem('configUpdatedAt', config.configUpdatedAt);
      // Re-render handles with new config
      updateAllHandles(config);
    }
  });
```

---

## Testing with cURL

### Test All Endpoints

```bash
# Set base URL and API key
BASE_URL="https://debox-handle-marketplace.replit.app"
API_KEY="DEBOX_HANDLE_API_KEY"

# 1. Check version info
echo "=== Version Info ==="
curl -s -H "x-api-key: $API_KEY" "$BASE_URL/api/embed/version" | jq

# 2. Download CSS and check content
echo -e "\n=== CSS (first 500 chars) ==="
curl -s -H "x-api-key: $API_KEY" "$BASE_URL/api/embed/styles.css" | head -c 500

# 3. Download renderer and check content
echo -e "\n\n=== Renderer (first 500 chars) ==="
curl -s -H "x-api-key: $API_KEY" "$BASE_URL/api/embed/renderer.js" | head -c 500

# 4. Get config for testnet community
echo -e "\n\n=== Testnet Community Config ==="
curl -s -H "x-api-key: $API_KEY" \
  "$BASE_URL/api/embed/0x9244212403a2e827cadca1f6fb68b43bc0c7a41f/config?env=testnet" | jq

# 5. Test conditional request with ETag
echo -e "\n=== ETag Test ==="
ETAG=$(curl -s -I -H "x-api-key: $API_KEY" \
  "$BASE_URL/api/embed/0x9244212403a2e827cadca1f6fb68b43bc0c7a41f/config?env=testnet" \
  | grep -i etag | cut -d' ' -f2 | tr -d '\r')
echo "ETag: $ETAG"
curl -s -w "Status: %{http_code}\n" -o /dev/null \
  -H "x-api-key: $API_KEY" \
  -H "If-None-Match: $ETAG" \
  "$BASE_URL/api/embed/0x9244212403a2e827cadca1f6fb68b43bc0c7a41f/config?env=testnet"

# 6. Test invalid token address
echo -e "\n=== Invalid Token Test ==="
curl -s -H "x-api-key: $API_KEY" "$BASE_URL/api/embed/0xinvalid/config?env=testnet" | jq

# 7. Test non-existent community
echo -e "\n=== Non-existent Community Test ==="
curl -s -H "x-api-key: $API_KEY" \
  "$BASE_URL/api/embed/0x0000000000000000000000000000000000000000/config?env=testnet" | jq
```

### Sample Testnet Token Addresses

These token addresses are registered on testnet and can be used for testing:

| Token Address | Description |
|---------------|-------------|
| `0x9244212403a2e827cadca1f6fb68b43bc0c7a41f` | Primary testnet community |
| `0xf762407aec88afd53be1f6d94a6c98ff5c6e4a25` | Testnet community |
| `0x4b9FF95124d5bD4dc39334372373c005D6b9C859` | Testnet community |

For mainnet communities, check `/api/communities?env=mainnet` to find active communities.

---

## Error Responses

All error responses follow this format:

```json
{
  "error": "Error message description",
  "tokenAddress": "0x...",  // When applicable
  "environment": "mock"     // When applicable
}
```

| Status Code | Meaning |
|-------------|---------|
| 400 | Invalid token address format |
| 404 | Community/version not found |
| 500 | Server error |

---

## Support

For questions or issues with the Debox Handles Embed API, please contact Carlos Chacon (cchakons@gmail.com)
