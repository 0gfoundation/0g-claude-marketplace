# 0G Plugin Marketplace for Claude Code

Official marketplace for **0G ecosystem** plugins - Making Claude an expert in decentralized infrastructure.

## Architecture

This repository is a **marketplace** that hosts multiple independent plugins for different 0G products:

```text
0g-claude-marketplace/
├── .claude-plugin/
│   └── marketplace.json       # Marketplace configuration
│
├── plugins/                   # Individual plugins
│   ├── 0g-compute/           # AI inference & fine-tuning
│   ├── 0g-storage/           # Decentralized storage (Coming Soon)
│   └── ...                   # More plugins
│
└── README.md
```

## Available Plugins

### 🚀 0G Compute (Active)

Decentralized AI inference and fine-tuning platform.

**Capabilities:**

- **AI Inference**: Chatbots, image generation, speech-to-text
- **Model Fine-tuning**: Train models on distributed GPUs (testnet)
- **Account Management**: Deposits, transfers, refunds
- **SDK Integration**: Correct API usage and best practices

**Status**: ✅ Active
**Location**: `plugins/0g-compute/`

### 💾 0G Storage (Coming Soon)

High-performance decentralized storage network.

**Status**: 🔜 Coming Soon
**Location**: `plugins/0g-storage/`

### 📊 0G DA (Coming Soon)

Data availability layer for blockchain applications.

**Status**: 🔜 Coming Soon

## Installation

### Step 1: Add the Marketplace

First, add the 0G marketplace to Claude Code:

```bash
/plugin marketplace add https://github.com/0gfoundation/0g-claude-marketplace
```

Or use the interactive UI:

1. Run `/plugin`
2. Navigate to the **Marketplaces** tab
3. Click **Add Marketplace**
4. Enter the repository URL

### Step 2: Install Plugins

Once the marketplace is added, install plugins from it:

```bash
/plugin install 0g-compute@0g-marketplace
```

### Verification

Ask Claude:

```text
"What 0G plugins are available?"
```

You should see all installed plugins listed.

## Managing Updates

### Update the Marketplace

To refresh the marketplace and get the latest plugin list:

```bash
/plugin marketplace update 0g-marketplace
```

Or via UI: `/plugin` → **Marketplaces** tab → Click refresh icon

### Update Installed Plugins

Update specific plugins:

```bash
/plugin update 0g-compute
```

Update all plugins:

```bash
/plugin update --all
```

### Auto-Update (Recommended)

Enable automatic updates for the marketplace:

1. Run `/plugin`
2. Go to **Marketplaces** tab
3. Select **0g-marketplace**
4. Enable **Auto-update**

When enabled, Claude Code automatically refreshes the marketplace and updates plugins on startup.

**Note**: Third-party marketplaces have auto-update disabled by default for security.

## Usage

Each plugin's skills activate automatically when you mention relevant topics.

### 0G Compute Examples

```text
"Help me integrate 0G Compute SDK"
"Create a streaming chatbot with DeepSeek V3.1"
"Show me how to fine-tune a model on 0G testnet"
"What's the correct way to call processResponse()?"
```

## Plugin Structure

Each plugin follows this structure:

```text
plugins/[plugin-name]/
├── .claude-plugin/
│   └── plugin.json           # Plugin configuration
├── skills/                   # Skill documentation
│   └── [skill-name]/
│       ├── SKILL.md         # Main skill reference
│       └── *.md             # Additional docs
└── src/                     # MCP Server code (if applicable)
```


## Contributing

To add a new plugin:

1. Create a new directory under `plugins/[plugin-name]/`
2. Add `.claude-plugin/plugin.json` configuration
3. Add skills documentation in `skills/` directory
4. Update `marketplace.json` to include the plugin
5. Submit a pull request

## Support

- **Issues**: <https://github.com/0gfoundation/0g-claude-marketplace/issues>

---

**Made with Claude Code** 🚀
