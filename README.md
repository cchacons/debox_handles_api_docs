# DeBox Handle Marketplace - Embed API Documentation

The Embed API allows you to render DeBox handles with their customization effects in your applications. This includes animated text effects, gradients, glow effects, and community branding.

## Two Integration Approaches

The API provides **two different approaches** depending on your platform:

### 1. Web Applications (CSS + JavaScript)

For web-based applications (websites, WebViews, web apps), we provide:
- **`/styles.css`** - Pre-built CSS with all effect animations
- **`/renderer.js`** - JavaScript library to apply effects to DOM elements
- **`/:tokenAddress/config`** - JSON configuration for the web renderer

This approach is ideal when you can use CSS animations and JavaScript DOM manipulation. The renderer handles all the complexity of applying gradients, animations, and glow effects.

### 2. Native Mobile Applications (JSON Config)

For native iOS (Swift) and Android (Kotlin) applications, we provide:
- **`/:tokenAddress/native`** - Platform-agnostic JSON with rendering instructions

This endpoint returns structured data with keyframe animations that your native code can interpret to render handles using native UI frameworks (UIKit/SwiftUI, Jetpack Compose, Android Views). No CSS or JavaScript required.

| Feature | Web (CSS/JS) | Native (JSON) |
|---------|--------------|---------------|
| Platform | Browsers, WebViews | iOS, Android native |
| Rendering | CSS animations, DOM | Native UI frameworks |
| Effects | Full CSS support | Keyframe-based animations |
| Glow | CSS text-shadow | Shadow layer configurations |
| Gradients | CSS gradients | Color array + direction |

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [Authentication](#authentication)
3. [API Endpoints](#api-endpoints)
4. [Configuration Response](#configuration-response)
5. [Native SDK Integration](#native-sdk-integration)
6. [Rendering Handles](#rendering-handles)
7. [Real-World Examples](#real-world-examples)
8. [Caching Strategy](#caching-strategy)
9. [Testing with cURL](#testing-with-curl)

---

## Quick Start

### Step 1: Test the API with cURL

First, verify the API is working by fetching a community configuration:

```bash
curl -s -H "x-api-key: DEBOX_HANDLE_API_KEY" \
  "https://debox-handle-marketplace.replit.app/api/embed/0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7/config?env=testnet" | jq
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
    const TOKEN = '0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7';
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
    fetch('https://debox-handle-marketplace.replit.app/api/embed/0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7/config?env=testnet')
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
  "https://debox-handle-marketplace.replit.app/api/embed/0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7/config?env=testnet" | jq

# Try other testnet communities
curl -s -H "x-api-key: DEBOX_HANDLE_API_KEY" \
  "https://debox-handle-marketplace.replit.app/api/embed/0x9244212403a2e827cadca1f6fb68b43bc0c7a41f/config?env=testnet" | jq

# Get config for mainnet community
curl -s -H "x-api-key: DEBOX_HANDLE_API_KEY" \
  "https://debox-handle-marketplace.replit.app/api/embed/0xYourTokenAddress/config?env=mainnet" | jq

# Conditional request with ETag
curl -s -H "x-api-key: DEBOX_HANDLE_API_KEY" \
  "https://debox-handle-marketplace.replit.app/api/embed/0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7/config?env=testnet" \
  -H "If-None-Match: \"abc123\"" -w "%{http_code}"
```

---

### GET /api/embed/:tokenAddress/native

Returns a native mobile SDK configuration for rendering handles in iOS (Swift) and Android (Kotlin) apps. This endpoint provides platform-agnostic JSON with keyframe animations instead of CSS.

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `tokenAddress` | string | Yes | The community's token contract address (0x...) |

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `env` | string | No | Environment: `mock`, `testnet`, or `mainnet` (defaults to `mock`) |

**Headers Returned:**
- `Content-Type: application/json`
- `ETag: "<hash>"` (for conditional requests)
- `Cache-Control: public, max-age=60`

**cURL Examples:**
```bash
# Get native config for a testnet community
curl -s -H "x-api-key: DEBOX_HANDLE_API_KEY" \
  "https://debox-handle-marketplace.replit.app/api/embed/0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7/native?env=testnet" | jq

# Get native config for mock environment
curl -s -H "x-api-key: DEBOX_HANDLE_API_KEY" \
  "https://debox-handle-marketplace.replit.app/api/embed/0x82e9e495883b0a13f964479117aad17bb061ecfe/native?env=mock" | jq
```

**Response Example:**
```json
{
  "version": "1.0.0",
  "tokenAddress": "0x82e9e495883b0a13f964479117aad17bb061ecfe",
  "community": {
    "name": "Test Token ECFE",
    "slug": "testecfe",
    "namespaceSuffix": ".testecfe"
  },
  "configUpdatedAt": "2025-11-23T06:01:08.984Z",
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
      { "position": 0, "hueRotation": 0 },
      { "position": 0.5, "hueRotation": 90 },
      { "position": 1, "hueRotation": 0 }
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

**Native Config Field Reference:**

| Field | Type | Description |
|-------|------|-------------|
| `version` | string | API version (e.g., "1.0.0") |
| `tokenAddress` | string | Community's token contract address |
| `community` | object | Community info: `name`, `slug`, `namespaceSuffix` |
| `configUpdatedAt` | string | ISO timestamp of last config change |
| `textStyle` | object | Text color/gradient settings |
| `animation` | object | Animation with keyframes (0.0-1.0 positions) |
| `glow` | object | Glow effect settings |
| `shadow` | object | Multi-layer shadow settings |

**textStyle Types:**
- `solid`: Single color with `color` field
- `gradient`: Multiple colors with `direction` and `colors[]` fields

**Animation Keyframe Properties:**
- `position`: 0.0 to 1.0 (progress through animation)
- `hueRotation`: Degrees to rotate hue
- `opacity`: 0.0 to 1.0
- `scale`: Scale factor (1.0 = normal)
- `translateX/Y`: Translation in pixels

---

## Configuration Response

The config endpoint returns a comprehensive customization configuration:

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

## Native SDK Integration

This section covers the `/api/embed/:tokenAddress/native` endpoint for native iOS and Android applications. The native endpoint provides JSON-based rendering instructions that your app can use to render handles without CSS or JavaScript.

### Quick Start for Native Apps

```bash
# Fetch native config for a community
curl -s -H "x-api-key: YOUR_API_KEY" \
  "https://debox-handle-marketplace.replit.app/api/embed/0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7/native?env=testnet"
```

### Configuration Types

The native endpoint returns different configurations based on the community's customization settings. Currently, we support two primary configuration types:

#### 1. Base Standard Color (Solid Color)

When a community uses a simple solid color for their handles, the response is minimal:

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

**Rendering in Swift (iOS):**
```swift
struct HandleConfig: Codable {
    let version: String
    let tokenAddress: String
    let community: Community
    let configUpdatedAt: String
    let textStyle: TextStyle
}

struct TextStyle: Codable {
    let type: String
    let color: String?
}

// Apply solid color to a UILabel
func applyHandleStyle(to label: UILabel, config: HandleConfig) {
    if config.textStyle.type == "solid", let hexColor = config.textStyle.color {
        label.textColor = UIColor(hex: hexColor)
    }
}
```

**Rendering in Kotlin (Android):**
```kotlin
data class HandleConfig(
    val version: String,
    val tokenAddress: String,
    val community: Community,
    val configUpdatedAt: String,
    val textStyle: TextStyle
)

data class TextStyle(
    val type: String,
    val color: String? = null
)

// Apply solid color to a TextView
fun applyHandleStyle(textView: TextView, config: HandleConfig) {
    if (config.textStyle.type == "solid" && config.textStyle.color != null) {
        textView.setTextColor(Color.parseColor(config.textStyle.color))
    }
}
```

#### 2. Prebuilt Effect

Prebuilt effects are complete visual packages with gradients, animations, and glow settings. When a community uses a prebuilt effect, the response includes all rendering details:

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
      { "position": 0.25, "hueRotation": 30 },
      { "position": 0.5, "hueRotation": 90 },
      { "position": 0.75, "hueRotation": 60 },
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

### Field Specifications

#### textStyle Object

| Field | Type | Description |
|-------|------|-------------|
| `type` | `"solid"` \| `"gradient"` | The type of text coloring |
| `color` | string | Hex color (only for `solid` type) |
| `colors` | string[] | Array of hex colors (only for `gradient` type) |
| `direction` | string | Gradient direction: `"toRight"`, `"toBottom"`, `"toBottomRight"`, `"radial"` |

#### animation Object (Optional)

| Field | Type | Description |
|-------|------|-------------|
| `type` | string | Animation type: `"hueShift"`, `"pulse"`, `"shimmer"`, etc. |
| `duration` | number | Duration in milliseconds |
| `repeat` | `"infinite"` \| number | How many times to repeat |
| `easing` | string | Easing function: `"linear"`, `"easeInOut"`, etc. |
| `keyframes` | array | Array of keyframe objects |

#### Keyframe Object

Each keyframe represents a point in the animation. The `position` is a value from 0.0 to 1.0 representing progress through the animation.

| Field | Type | Description |
|-------|------|-------------|
| `position` | number | 0.0 to 1.0 (progress through animation) |
| `hueRotation` | number | Degrees to rotate hue (0-360) |
| `opacity` | number | Opacity value (0.0 to 1.0) |
| `scale` | number | Scale factor (1.0 = normal size) |
| `translateX` | number | Horizontal translation in pixels |
| `translateY` | number | Vertical translation in pixels |

#### glow Object (Optional)

| Field | Type | Description |
|-------|------|-------------|
| `size` | number | Glow radius in pixels |
| `intensity` | number | Glow intensity (100 = normal) |
| `color` | string | Hex color for the glow |
| `colorSource` | `"base"` \| `"custom"` | Whether color is from text or custom |

#### shadow Object (Optional)

| Field | Type | Description |
|-------|------|-------------|
| `layers` | array | Array of shadow layer objects |

Each shadow layer:
| Field | Type | Description |
|-------|------|-------------|
| `color` | string | Hex color for the shadow |
| `blur` | number | Blur radius in pixels |
| `opacity` | number | Shadow opacity |
| `offsetX` | number | Horizontal offset (optional) |
| `offsetY` | number | Vertical offset (optional) |

### Implementation Examples

#### Swift (iOS) - Complete Prebuilt Effect Rendering

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

class HandleRenderer {
    
    func renderHandle(label: UILabel, handleName: String, config: NativeConfig) {
        // Set the text
        label.text = handleName + config.community.namespaceSuffix
        
        // Apply text style
        applyTextStyle(to: label, textStyle: config.textStyle)
        
        // Apply shadow/glow if present
        if let shadow = config.shadow {
            applyShadow(to: label, shadow: shadow)
        }
        
        // Apply animation if present
        if let animation = config.animation {
            applyAnimation(to: label, animation: animation)
        }
    }
    
    private func applyTextStyle(to label: UILabel, textStyle: TextStyle) {
        if textStyle.type == "solid", let color = textStyle.color {
            label.textColor = UIColor(hex: color)
        } else if textStyle.type == "gradient", let colors = textStyle.colors {
            // Apply gradient text (requires CAGradientLayer mask)
            applyGradientText(to: label, colors: colors, direction: textStyle.direction)
        }
    }
    
    private func applyShadow(to label: UILabel, shadow: Shadow) {
        // Use first layer for UILabel shadow (for multiple layers, use custom drawing)
        if let firstLayer = shadow.layers.first {
            label.layer.shadowColor = UIColor(hex: firstLayer.color)?.cgColor
            label.layer.shadowRadius = CGFloat(firstLayer.blur)
            label.layer.shadowOpacity = Float(firstLayer.opacity)
            label.layer.shadowOffset = .zero
        }
    }
    
    private func applyAnimation(to label: UILabel, animation: Animation) {
        // Implement keyframe animation based on type
        if animation.type == "hueShift" {
            startHueShiftAnimation(on: label, animation: animation)
        }
    }
}
```

#### Kotlin (Android) - Complete Prebuilt Effect Rendering

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

class HandleRenderer {
    
    fun renderHandle(textView: TextView, handleName: String, config: NativeConfig) {
        // Set the text
        textView.text = handleName + config.community.namespaceSuffix
        
        // Apply text style
        applyTextStyle(textView, config.textStyle)
        
        // Apply shadow/glow if present
        config.shadow?.let { applyShadow(textView, it) }
        
        // Apply animation if present
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
        val animator = ValueAnimator.ofFloat(0f, 1f).apply {
            duration = animation.duration.toLong()
            repeatCount = if (animation.repeat == "infinite") ValueAnimator.INFINITE else 0
            interpolator = LinearInterpolator()
            
            addUpdateListener { valueAnimator ->
                val progress = valueAnimator.animatedValue as Float
                val hueRotation = interpolateKeyframes(animation.keyframes, progress)
                // Apply hue rotation to the text color/gradient
                applyHueRotation(textView, hueRotation)
            }
        }
        animator.start()
    }
    
    private fun interpolateKeyframes(keyframes: List<Keyframe>, progress: Float): Float {
        // Find the two keyframes we're between
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

### Testing the Native Endpoint

Use these testnet community tokens for testing:

| Token Address | Description |
|---------------|-------------|
| `0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7` | Primary testnet community |
| `0x9244212403a2e827cadca1f6fb68b43bc0c7a41f` | Testnet community |
| `0xf762407aec88afd53be1f6d94a6c98ff5c6e4a25` | Testnet community |
| `0x4b9FF95124d5bD4dc39334372373c005D6b9C859` | Testnet community |

```bash
# Test with different environments
curl -s -H "x-api-key: YOUR_API_KEY" \
  "https://debox-handle-marketplace.replit.app/api/embed/0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7/native?env=testnet"

# Test conditional caching
ETAG=$(curl -sI -H "x-api-key: YOUR_API_KEY" \
  "https://debox-handle-marketplace.replit.app/api/embed/0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7/native?env=testnet" \
  | grep -i etag | cut -d' ' -f2)

curl -s -w "HTTP Status: %{http_code}\n" -o /dev/null \
  -H "x-api-key: YOUR_API_KEY" \
  -H "If-None-Match: $ETAG" \
  "https://debox-handle-marketplace.replit.app/api/embed/0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7/native?env=testnet"
# Should return 304 Not Modified
```

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
  const TOKEN_ADDRESS = '0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7';
  
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
  const TOKEN_ADDRESS = '0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7';
  
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
  tokenAddress="0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7"
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
  "$BASE_URL/api/embed/0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7/config?env=testnet" | jq

# 5. Test conditional request with ETag
echo -e "\n=== ETag Test ==="
ETAG=$(curl -s -I -H "x-api-key: $API_KEY" \
  "$BASE_URL/api/embed/0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7/config?env=testnet" \
  | grep -i etag | cut -d' ' -f2 | tr -d '\r')
echo "ETag: $ETAG"
curl -s -w "Status: %{http_code}\n" -o /dev/null \
  -H "x-api-key: $API_KEY" \
  -H "If-None-Match: $ETAG" \
  "$BASE_URL/api/embed/0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7/config?env=testnet"

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
| `0x82C8796412EaE4dBEB6Df318f3841706e6b98Ed7` | Primary testnet community |
| `0x9244212403a2e827cadca1f6fb68b43bc0c7a41f` | Testnet community |
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

For questions or issues with the Embed API, please contact the DeBox team.
