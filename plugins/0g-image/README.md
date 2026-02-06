# 0G Image Plugin

Text-to-image generation using 0G Compute Network's **z-image** model.

## Features

- 🎨 Generate images from text descriptions
- 🔑 Two authentication methods: API Key (simple) and SDK (wallet-based)
- 📐 512x512 high-quality images
- 💰 Low cost: ~0.003 0G per image

## Quick Start

### API Key Method (Simplest)

```bash
export ZG_API_KEY="app-sk-..."

curl -X POST "https://39.97.249.15:8888/v1/proxy/images/generations" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $ZG_API_KEY" \
  -d '{"model": "z-image", "prompt": "A cute robot", "n": 1, "size": "512x512", "response_format": "b64_json"}' \
  | jq -r '.data[0].b64_json' | base64 -d > output.png
```

### SDK Method (Full Control)

```typescript
import { createZGComputeNetworkBroker } from '@0gfoundation/0g-serving-broker';
// See skills/SKILL.md for complete example
```

## Provider Info

| Field | Value |
|-------|-------|
| Provider | `0xE29a72c7629815Eb480aE5b1F2dfA06f06cdF974` |
| Model | `z-image` |
| Size | 512x512 |
| Price | ~0.003 0G/image |

## Documentation

See [skills/SKILL.md](skills/SKILL.md) for complete documentation.
