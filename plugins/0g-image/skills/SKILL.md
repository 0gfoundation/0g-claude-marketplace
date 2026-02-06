---
name: 0g-image
description: Expert in 0G Compute z-image model for text-to-image generation. Use when helping with image generation, z-image API, decentralized AI art, or 0G Compute image services. Supports both API Key method (simple) and SDK method (wallet-based).
---

# 0G z-image Text-to-Image Generation

Generate images from text descriptions using the **z-image** model on 0G Compute Network.

## Overview

The z-image model provides decentralized text-to-image generation through 0G Compute Network. Two methods are available:

| Method | Best For | Requirements |
|--------|----------|--------------|
| **API Key** | Quick integration, testing | API key from 0G |
| **SDK** | Full control, production apps | Wallet + mainnet 0G tokens |

## z-image Provider Info

| Field | Value |
|-------|-------|
| Provider Address | `0xE29a72c7629815Eb480aE5b1F2dfA06f06cdF974` |
| Endpoint | `https://39.97.249.15:8888/v1/proxy` |
| Model | `z-image` |
| Image Size | 512x512 |
| Price | ~0.003 0G per image |
| Network | 0G Mainnet |

## Method 1: API Key (Recommended for Quick Start)

The simplest way to generate images - no wallet or SDK setup required.

### Setup

```bash
# Set your API key
export ZG_API_KEY="app-sk-..."
```

### Generate Image (Shell)

```bash
curl -X POST "https://39.97.249.15:8888/v1/proxy/images/generations" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $ZG_API_KEY" \
  -d '{
    "model": "z-image",
    "prompt": "A cute robot waving hello",
    "n": 1,
    "size": "512x512",
    "response_format": "b64_json"
  }' | jq -r '.data[0].b64_json' | base64 -d > output.png
```

### Generate Image (Node.js)

```javascript
import fs from 'fs/promises';

const API_KEY = process.env.ZG_API_KEY;
const ENDPOINT = 'https://39.97.249.15:8888/v1/proxy/images/generations';

async function generateImage(prompt) {
    const response = await fetch(ENDPOINT, {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${API_KEY}`
        },
        body: JSON.stringify({
            model: 'z-image',
            prompt: prompt,
            n: 1,
            size: '512x512',
            response_format: 'b64_json'
        })
    });

    const data = await response.json();
    const imageBuffer = Buffer.from(data.data[0].b64_json, 'base64');
    await fs.writeFile('output.png', imageBuffer);
    
    return 'output.png';
}

// Usage
await generateImage('A beautiful sunset over mountains');
```

## Method 2: SDK (Full Control)

Use the 0G Compute SDK for wallet-based authentication and full control.

### Prerequisites

```bash
# Node.js >= 22.0.0 required
node --version

# Install SDK
pnpm add @0gfoundation/0g-serving-broker ethers dotenv
```

### Setup Account

```bash
# Install CLI
pnpm add @0gfoundation/0g-serving-broker -g

# Configure network (choose mainnet)
0g-compute-cli setup-network

# Login with private key
0g-compute-cli login

# Deposit funds (minimum 3 0G for new accounts)
0g-compute-cli add-account --amount 3

# Acknowledge z-image provider
0g-compute-cli inference acknowledge-provider \
  --provider 0xE29a72c7629815Eb480aE5b1F2dfA06f06cdF974
```

### Generate Image (SDK)

```typescript
import { ethers } from 'ethers';
import { createZGComputeNetworkBroker } from '@0gfoundation/0g-serving-broker';
import axios from 'axios';
import fs from 'fs/promises';

const RPC_URL = 'https://evmrpc.0g.ai';
const PROVIDER_ADDRESS = '0xE29a72c7629815Eb480aE5b1F2dfA06f06cdF974';

async function generateImage(prompt: string) {
    // Setup
    const provider = new ethers.JsonRpcProvider(RPC_URL);
    const wallet = new ethers.Wallet(process.env.PRIVATE_KEY!, provider);
    const broker = await createZGComputeNetworkBroker(wallet);
    
    // Get service metadata
    const { endpoint, model } = await broker.inference.getServiceMetadata(PROVIDER_ADDRESS);
    
    // Acknowledge provider (first time only)
    try {
        await broker.inference.acknowledgeProviderSigner(PROVIDER_ADDRESS);
    } catch (e) {
        // Already acknowledged
    }
    
    // Get auth headers
    const headers = await broker.inference.getRequestHeaders(PROVIDER_ADDRESS);
    
    // Generate image
    const response = await axios.post(`${endpoint}/images/generations`, {
        model: model,
        prompt: prompt,
        n: 1,
        size: '512x512',
        response_format: 'b64_json'
    }, {
        headers: {
            'Content-Type': 'application/json',
            ...headers
        },
        timeout: 120000
    });
    
    // Save image
    const imageBuffer = Buffer.from(response.data.data[0].b64_json, 'base64');
    const outputPath = `image_${Date.now()}.png`;
    await fs.writeFile(outputPath, imageBuffer);
    
    // Process response for fee management
    const chatID = response.headers['zg-res-key'];
    if (chatID) {
        await broker.inference.processResponse(
            PROVIDER_ADDRESS,
            chatID,
            undefined
        );
    }
    
    return outputPath;
}

// Usage
const imagePath = await generateImage('A golden dragon flying through clouds');
console.log(`Image saved to: ${imagePath}`);
```

## API Reference

### Request Format

```json
POST /images/generations
Content-Type: application/json
Authorization: Bearer <API_KEY or SDK_TOKEN>

{
    "model": "z-image",
    "prompt": "Your image description",
    "n": 1,
    "size": "512x512",
    "response_format": "b64_json"
}
```

### Response Format

```json
{
    "data": [
        {
            "b64_json": "iVBORw0KGgo..."
        }
    ]
}
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model` | string | Yes | Must be `"z-image"` |
| `prompt` | string | Yes | Image description |
| `n` | number | No | Number of images (default: 1) |
| `size` | string | No | Image size (default: `"512x512"`) |
| `response_format` | string | No | `"b64_json"` or `"url"` |

## Complete Example Project

### Project Structure

```
0g-image-generator/
├── package.json
├── .env
├── generate-api.js      # API Key method
└── generate-sdk.js      # SDK method
```

### package.json

```json
{
  "name": "0g-image-generator",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "api": "node generate-api.js",
    "sdk": "node generate-sdk.js"
  },
  "dependencies": {
    "@0gfoundation/0g-serving-broker": "^0.6.6",
    "axios": "^1.7.9",
    "dotenv": "^16.4.7",
    "ethers": "^6.13.1"
  }
}
```

### .env

```bash
# For API Key method
ZG_API_KEY=app-sk-...

# For SDK method
PRIVATE_KEY=your_wallet_private_key
RPC_URL=https://evmrpc.0g.ai
```

### generate-api.js (API Key Method)

```javascript
import fs from 'fs/promises';
import dotenv from 'dotenv';
dotenv.config();

const API_KEY = process.env.ZG_API_KEY;
const ENDPOINT = 'https://39.97.249.15:8888/v1/proxy/images/generations';

async function main() {
    const prompt = process.argv[2] || 'A beautiful landscape';
    
    console.log('🎨 0G z-image Generator (API Key)');
    console.log(`📝 Prompt: "${prompt}"`);
    
    const response = await fetch(ENDPOINT, {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${API_KEY}`
        },
        body: JSON.stringify({
            model: 'z-image',
            prompt,
            n: 1,
            size: '512x512',
            response_format: 'b64_json'
        })
    });
    
    const data = await response.json();
    
    if (data.error) {
        console.error('❌ Error:', data.error);
        process.exit(1);
    }
    
    const imageBuffer = Buffer.from(data.data[0].b64_json, 'base64');
    const filename = `image_${Date.now()}.png`;
    await fs.writeFile(filename, imageBuffer);
    
    console.log(`✨ Image saved: ${filename}`);
}

main().catch(console.error);
```

## Getting an API Key

### Option 1: From 0G Team
Contact 0G team for internal/testing API keys.

### Option 2: Generate from Wallet
Use the SDK method to authenticate with your wallet.

## Troubleshooting

### "Invalid API Key"
- Check the API key format: should start with `app-sk-`
- Verify the key hasn't expired

### "Insufficient Balance" (SDK)
```bash
0g-compute-cli add-account --amount 3
```

### "Provider Not Acknowledged" (SDK)
```bash
0g-compute-cli inference acknowledge-provider \
  --provider 0xE29a72c7629815Eb480aE5b1F2dfA06f06cdF974
```

### Timeout Errors
- z-image generation can take 30-60 seconds
- Increase timeout to 120000ms (2 minutes)

## Best Practices

1. **Use API Key for testing** - simpler setup
2. **Use SDK for production** - full control and cost management
3. **Handle errors gracefully** - network issues are common
4. **Set appropriate timeouts** - image generation takes time
5. **Monitor costs** - track usage with `get-account` command

## Resources

- **0G Compute Docs**: https://docs.0g.ai
- **SDK Package**: https://www.npmjs.com/package/@0gfoundation/0g-serving-broker
- **Marketplace**: https://compute-marketplace.0g.ai/inference
- **Discord**: https://discord.gg/0gfoundation
