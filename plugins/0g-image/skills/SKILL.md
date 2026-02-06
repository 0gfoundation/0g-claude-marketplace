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
| Model | `z-image` |
| Image Size | 512x512 |
| Price | ~0.003 0G per image |
| Network | 0G Mainnet |
| Response Time | 30-60 seconds typical |

> **Note**: The z-image endpoint is dynamically resolved from the provider's on-chain metadata using `broker.inference.getServiceMetadata()`. The endpoint may change; always retrieve it programmatically in production. For quick testing, you can use the current endpoint shown in the examples.

## Method 1: API Key (Recommended for Quick Start)

The simplest way to generate images - no wallet or SDK setup required.

### Setup

```bash
# Set your API key and endpoint
export ZG_API_KEY="app-sk-..."
export ZG_IMAGE_ENDPOINT="https://39.97.249.15:8888/v1/proxy/images/generations"
```

### Generate Image (Shell)

```bash
curl -X POST "$ZG_IMAGE_ENDPOINT" \
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
const ENDPOINT = process.env.ZG_IMAGE_ENDPOINT || 
    'https://39.97.249.15:8888/v1/proxy/images/generations';

async function generateImage(prompt) {
    if (!API_KEY) {
        throw new Error('ZG_API_KEY environment variable not set');
    }

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

    // Check for errors
    if (!response.ok || data.error) {
        throw new Error(data.error?.message || `API error: ${response.status}`);
    }

    // Validate response structure
    if (!data.data?.[0]?.b64_json) {
        throw new Error('Invalid response: missing image data');
    }

    const imageBuffer = Buffer.from(data.data[0].b64_json, 'base64');
    await fs.writeFile('output.png', imageBuffer);
    
    return 'output.png';
}

// Usage
try {
    const path = await generateImage('A beautiful sunset over mountains');
    console.log(`Image saved to: ${path}`);
} catch (error) {
    console.error('Failed to generate image:', error.message);
}
```

## Method 2: SDK (Full Control)

Use the 0G Compute SDK for wallet-based authentication and full control.

### Prerequisites

```bash
# Node.js >= 22.0.0 required
node --version

# Install SDK and dependencies
pnpm add @0gfoundation/0g-serving-broker@^0.6.6 ethers@^6.13.1 axios@^1.7.9 dotenv@^16.4.7
```

### Setup Account

```bash
# Install CLI
pnpm add @0gfoundation/0g-serving-broker -g

# Configure network (choose mainnet)
0g-compute-cli setup-network

# Set up account with private key
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

async function generateImage(prompt: string): Promise<string> {
    // Validate environment
    if (!process.env.PRIVATE_KEY) {
        throw new Error('PRIVATE_KEY environment variable not set');
    }

    // Setup wallet and broker
    const provider = new ethers.JsonRpcProvider(RPC_URL);
    const wallet = new ethers.Wallet(process.env.PRIVATE_KEY, provider);
    const broker = await createZGComputeNetworkBroker(wallet);
    
    // Get service metadata (endpoint is resolved dynamically)
    const { endpoint, model } = await broker.inference.getServiceMetadata(PROVIDER_ADDRESS);
    
    // Acknowledge provider (first time only)
    try {
        await broker.inference.acknowledgeProviderSigner(PROVIDER_ADDRESS);
        console.log('Provider acknowledged successfully');
    } catch (e: any) {
        if (e.message?.includes('already acknowledged') || e.message?.includes('Acknowledged')) {
            console.log('Provider already acknowledged');
        } else {
            throw new Error(`Failed to acknowledge provider: ${e.message}`);
        }
    }
    
    // Get auth headers
    const headers = await broker.inference.getRequestHeaders(PROVIDER_ADDRESS);
    
    // Generate image
    try {
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
            timeout: 120000  // 2 minutes - image generation can take 30-60 seconds
        });

        // Validate response
        if (!response.data?.data?.[0]?.b64_json) {
            throw new Error('Invalid response: missing image data');
        }
        
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
    } catch (error) {
        if (axios.isAxiosError(error)) {
            const message = error.response?.data?.error || error.message;
            throw new Error(`API request failed: ${message}`);
        }
        throw error;
    }
}

// Usage
try {
    const imagePath = await generateImage('A golden dragon flying through clouds');
    console.log(`Image saved to: ${imagePath}`);
} catch (error) {
    console.error('Failed to generate image:', error.message);
}
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
ZG_IMAGE_ENDPOINT=https://39.97.249.15:8888/v1/proxy/images/generations

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
const ENDPOINT = process.env.ZG_IMAGE_ENDPOINT || 
    'https://39.97.249.15:8888/v1/proxy/images/generations';

async function main() {
    const prompt = process.argv[2] || 'A beautiful landscape';
    
    if (!API_KEY) {
        console.error('Error: ZG_API_KEY not set in .env');
        process.exit(1);
    }
    
    console.log('🎨 0G z-image Generator (API Key)');
    console.log(`📝 Prompt: "${prompt}"`);
    
    try {
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
        
        if (!response.ok || data.error) {
            console.error('❌ API Error:', data.error?.message || response.status);
            process.exit(1);
        }
        
        if (!data.data?.[0]?.b64_json) {
            console.error('❌ Invalid response: missing image data');
            process.exit(1);
        }
        
        const imageBuffer = Buffer.from(data.data[0].b64_json, 'base64');
        const filename = `image_${Date.now()}.png`;
        await fs.writeFile(filename, imageBuffer);
        
        console.log(`✨ Image saved: ${filename}`);
    } catch (error) {
        console.error('❌ Error:', error.message);
        process.exit(1);
    }
}

main();
```

## Activation

This skill activates automatically when you mention:
- "0g image" / "0g-image"
- "z-image"
- "text-to-image" with 0G
- "image generation" with 0G Compute

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
- z-image generation typically takes 30-60 seconds
- Ensure timeout is set to at least 120000ms (2 minutes)

### Endpoint Changes
- The endpoint may change as providers update their infrastructure
- Always use `broker.inference.getServiceMetadata()` in production to get the current endpoint
- Check 0G documentation or marketplace for updated endpoints

## Best Practices

1. **Use environment variables** - Never hardcode API keys or private keys
2. **Use API Key for testing** - Simpler setup, faster iteration
3. **Use SDK for production** - Full control, dynamic endpoint resolution, cost management
4. **Handle errors gracefully** - Network issues are common in decentralized systems
5. **Set appropriate timeouts** - Image generation takes 30-60 seconds
6. **Monitor costs** - Track usage with `0g-compute-cli get-account`
7. **Resolve endpoints dynamically** - Use SDK's `getServiceMetadata()` in production

## Resources

- **0G Compute Docs**: https://docs.0g.ai
- **SDK Package**: https://www.npmjs.com/package/@0gfoundation/0g-serving-broker
- **Marketplace**: https://compute-marketplace.0g.ai/inference
- **Discord**: https://discord.gg/0gfoundation
