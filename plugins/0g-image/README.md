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
# Set your API key
export ZG_API_KEY="app-sk-..."

# Generate image (see SKILL.md for complete examples)
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
| Response Time | 30-60 seconds |

> **Note**: Endpoint is dynamically resolved via SDK. See SKILL.md for details.

## Activation

This skill activates when you mention:
- "0g image" / "0g-image" / "z-image"
- "text-to-image" with 0G
- "image generation" with 0G Compute

## Documentation

See [skills/SKILL.md](skills/SKILL.md) for complete documentation including:
- Full code examples with error handling
- API reference
- Troubleshooting guide
- Best practices
