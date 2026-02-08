# ClawFi 🦀

**All-Access Crypto Intelligence Agent**

ClawFi is a production-grade crypto agent that runs 24/7 on local hardware, monitoring CEX and DEX activity, generating intelligent signals, and providing risk-controlled automation.

![ClawFi Architecture](docs/architecture.png)

## Features

### OpenClaw-Style All-Access Agent (v0.2.0)
- ⚡ **Agent Commands**: Natural command interface (`watch token`, `enable strategy`, `killswitch`)
- 🔗 **Unified Connections**: Single view of all connectors with health, status, enable/disable
- 👁️ **Watchlists**: Token and wallet monitoring via commands or dashboard
- 🤖 **Assist Mode** (stub): Prepare transactions for review (coming soon)

### Core Features
- 🔌 **Plugin Connectors**: Binance CEX, EVM DEX (Uniswap V2/V3), with extensible architecture
- 🚀 **Multi-Chain Launchpads**: Clanker (Base), Pump.fun (Solana), Four.meme (BSC)
- 🧠 **Strategy Engine**: Configurable detection strategies (MoltWatch for wallet rotations)
- 🛡️ **Risk Engine**: Global constraints, kill switch, audit logging
- 📊 **Web Dashboard**: Real-time monitoring, launchpad tracking, signal timeline, metrics
- 🔌 **Chrome Extension**: Multi-chain overlays with risk badges, token detection, signal display
- 💻 **CLI Tool**: Full terminal control with real-time signal streaming
- 📱 **Notifications**: Telegram and Discord webhooks with rich embeds
- 🎯 **Intelligence Signals**: Early distribution analysis, liquidity risk detection, price alerts
- 📈 **Prometheus Metrics**: Full observability endpoint at `/metrics`
- 🔐 **Security First**: Encrypted secrets, no withdrawal support, full audit trail

## Quick Start

### Prerequisites

- Node.js 20+
- pnpm 8+
- PostgreSQL 15+
- Redis 7+

### Installation

```bash
# Clone the repository
git clone https://github.com/clawfiai/clawfi.git
cd clawfi

# Install dependencies
pnpm install

# Copy and configure environment
cp env.example .env
# Edit .env with your settings

# Generate master key
openssl rand -hex 32
# Add to .env as CLAWFI_MASTER_KEY

# Setup database
pnpm db:generate
pnpm db:push
pnpm db:seed

# Start development servers
pnpm dev
```

The dashboard will be available at `http://localhost:3000` and the API at `http://localhost:3001`.

Default credentials: `admin@clawfi.local` / `clawfi123`

### Docker

```bash
cd infra/docker

# Start all services
docker compose up -d

# View logs
docker compose logs -f
```

## Architecture

```
/clawfi
├── /apps
│   ├── /dashboard     # Next.js web UI
│   ├── /extension     # Chrome MV3 extension
│   └── /node          # Fastify API + scheduler
│
├── /packages
│   ├── /core          # Domain models, schemas
│   ├── /connectors    # Exchange integrations
│   ├── /vault         # Secret encryption
│   ├── /sdk           # Client SDK
│   └── /config        # Shared configs
│
├── /infra
│   ├── /docker        # Docker configs
│   └── /systemd       # Service templates
│
└── /docs              # Documentation
```

## Safety & Security

⚠️ **Important Security Notes:**

1. **API Keys**: Only use keys with **withdrawals DISABLED**
2. **Dry Run Mode**: Enabled by default - orders are simulated
3. **Kill Switch**: Emergency stop for all trading
4. **Master Key**: Back up securely - losing it means losing encrypted data

See [SECURITY.md](docs/SECURITY.md) for full security documentation.

## Configuration

### Risk Policy Defaults

| Setting | Default | Description |
|---------|---------|-------------|
| Max Order | $100 | Maximum single order size |
| Max Position | $1000 | Maximum position value |
| Max Daily Loss | $500 | Stop trading after this loss |
| Max Slippage | 100 bps (1%) | Reject trades with higher slippage |
| Cooldown | 60s | Minimum time between trades |
| Dry Run | true | Simulate orders without execution |

### MoltWatch Strategy

Detects when tracked wallets significantly reduce positions ("molting"):

```json
{
  "moltThresholdPercent": 50,
  "rotationWindowMinutes": 60,
  "minPositionUsd": 1000,
  "cooldownMinutes": 30
}
```

Hardened detection requires:
- Position value >= minPositionUsd
- Sell >= moltThresholdPercent of baseline
- Follow-up buy or bridge within rotationWindowMinutes

### Launchpad Connectors

ClawFi supports multiple launchpad connectors across different chains:

| Launchpad | Chain | Status | Features |
|-----------|-------|--------|----------|
| **Clanker** | Base | ✅ Active | Token launches, creator tracking, signals |
| **Pump.fun** | Solana | ✅ Active | Bonding curves, graduation to Raydium |
| **Four.meme** | BSC | ✅ Active | Bonding curves, PancakeSwap graduation |

#### Clanker (Base)

```bash
CLANKER_ENABLED=true
CLANKER_POLL_INTERVAL_MS=10000
```

#### Pump.fun (Solana)

```bash
PUMPFUN_ENABLED=true
PUMPFUN_POLL_INTERVAL_MS=15000
PUMPFUN_GRADUATION_ONLY=false  # Only track graduated tokens
```

#### Four.meme (BSC)

```bash
FOURMEME_ENABLED=true
FOURMEME_POLL_INTERVAL_MS=15000
FOURMEME_MIN_MARKET_CAP=0  # Minimum market cap filter
```

See [CLANKER.md](docs/CLANKER.md) for detailed Clanker configuration.

## API Reference

Full API documentation: [API.md](docs/API.md)

### Key Endpoints

- `POST /auth/login` - Authenticate and get JWT
- `GET /connections` - Unified list of all connectors (OpenClaw-style)
- `GET /connectors` - List user connectors
- `POST /connectors/binance` - Add Binance API key
- `GET /signals` - Get signals with pagination
- `POST /risk/killswitch` - Enable/disable kill switch

### Agent Commands (v0.2.0)

- `GET /agent/status` - Full agent status overview
- `POST /agent/command` - Execute commands:
  - `watch token <address> [chain]`
  - `watch wallet <address> [chain]`
  - `enable strategy <name>`
  - `disable strategy <name>`
  - `killswitch on/off`
  - `status`

### WebSocket

Connect to `/ws?token=<jwt>` for real-time updates:
- Signal notifications
- System status updates

## CLI Tool

The ClawFi CLI provides terminal-based control of your agent:

```bash
# Install globally (or use pnpm -C apps/cli dev)
npm install -g @clawfi/cli

# Login to your ClawFi node
clawfi login --host http://localhost:3001

# Check agent status
clawfi status

# List recent signals
clawfi signals --limit 10 --severity high

# Watch a token
clawfi watch token 0x1234...5678 --chain base

# Execute agent commands
clawfi cmd "watch wallet 0x1234...5678 base"
clawfi cmd "killswitch on"

# Stream real-time signals
clawfi stream

# List strategies
clawfi strategies
```

## Extension

The Chrome extension provides a comprehensive crypto intelligence overlay with **Apple Liquid Glass UI** design:

### Capabilities
- **Injected UI Overlay** - Smart overlay injection on supported sites
- **In-Page Extension Interface** - Native-feeling UI components
- **Content Script UI Injection** - Real-time page augmentation
- **DOM Overlay Toolbar/Panel** - Floating panels with market data
- **Augmented Page UI** - Enhanced token analysis views
- **Client-side Page Augmentation** - Dexscreener market data integration

### Features
- **Clanker Overlay**: First-class support for clanker.world token pages
- **Assist Mode**: Quick actions (Swap on Uniswap, Copy Address, View on DEX, etc.)
- **Market Data**: Live price, volume, liquidity, market cap from Dexscreener
- **Tabbed Interface**: Signals | Market | Assist tabs
- **Multi-chain Support**: Base, Ethereum, Solana, BSC
- **Wallet Detection**: MetaMask, Phantom, Solflare
- **Signal Overlay**: Recent alerts with severity badges
- **Trending Tokens**: Top boosted tokens in popup

### Clanker Support

When visiting `https://clanker.world/clanker/0x...`:
- Automatic token address extraction from URL
- Last 5 signals displayed in overlay
- Metadata extraction (version, creator, admin, verified status)
- Links to Basescan for token and addresses

See [CLANKER.md](docs/CLANKER.md) for full Clanker integration docs.

### Installation

```bash
# Build extension
pnpm -C apps/extension build

# Load in Chrome:
# 1. Go to chrome://extensions/
# 2. Enable "Developer mode"
# 3. Click "Load unpacked"
# 4. Select apps/extension/dist
```

Configure in extension options:
- Node URL (default: http://localhost:3001)
- Auth token (from dashboard login)
- Toggle overlays on/off

## Development

```bash
# Run all in development mode
pnpm dev

# Build all packages
pnpm build

# Run tests
pnpm test

# Type check
pnpm typecheck

# Lint
pnpm lint
```

## Documentation

- [Architecture](docs/ARCHITECTURE.md)
- [Security](docs/SECURITY.md)
- [Connectors](docs/CONNECTORS.md)
- [Strategies](docs/STRATEGIES.md)
- [Clanker Integration](docs/CLANKER.md)
- [Extension Guide](docs/EXTENSION.md)
- [Raspberry Pi Install](docs/INSTALL_PI.md)
- [API Reference](docs/API.md)

## Roadmap

### v0.1.0
- ✅ Binance connector (read + trade)
- ✅ EVM DEX quoting
- ✅ MoltWatch strategy (hardened)
- ✅ Risk engine
- ✅ Web dashboard
- ✅ Chrome extension
- ✅ Clanker launchpad connector
- ✅ Clanker extension overlay
- ✅ Telegram notifications
- ✅ Launchpad dashboard

### v0.1.1 - Trust & Intelligence
- ✅ Launch coverage verification (statistical)
- ✅ Early distribution signal (holder concentration)
- ✅ Liquidity risk signal (LP removal detection)
- ✅ Extension overlay risk badges
- ✅ Dashboard coverage widget
- ✅ Token intelligence API endpoint
- ✅ Prometheus metrics endpoint

### v0.2.0 (Current) - OpenClaw-Style All-Access
- ✅ Unified Connections page (all connector types)
- ✅ Agent Command interface (`/agent/command`)
- ✅ Agent Dashboard with command terminal
- ✅ Watch token/wallet commands
- ✅ Connector enable/disable/health check
- ✅ Assist Mode button stub (extension)
- ✅ Watched tokens/wallets DB tables

### v0.2.1 (Current) - Multi-Chain Launchpads
- ✅ CLI tool (`@clawfi/cli`) with login, status, signals, watch, stream
- ✅ Discord webhook notifications with rich embeds
- ✅ Pump.fun connector (Solana launchpad)
- ✅ Four.meme connector (BSC launchpad)
- ✅ WebSocket reconnection with heartbeat
- ✅ GitHub Actions CI/CD workflows
- ✅ Price alert system for watched tokens
- ✅ API rate limiting middleware
- ✅ Comprehensive health check endpoints
- ✅ Metrics dashboard in web UI
- ✅ Chrome extension popup with stats
- ✅ Solana content script for Solscan/Pump.fun
- ✅ BSC content script for Four.meme
- ✅ Multi-chain SDK utilities
- ✅ Watchlist API routes
- ✅ Launchpad tokens API routes

### v0.3.1 (Current) - Apple Liquid Glass UI
- ✅ **Apple Liquid Glass UI** - iOS 26 inspired design system
- ✅ **Dexscreener Integration** - Full API: tokens, pairs, boosts, search, trending
- ✅ **Assist Mode** - Quick actions: swap links, copy addresses, DEX/scanner links
- ✅ **Tabbed Overlay Interface** - Signals | Market | Assist tabs
- ✅ **Market Data in Overlays** - Price, volume, liquidity, market cap from Dexscreener
- ✅ **Enhanced Popup** - Trending tokens, quick links, improved stats
- ✅ **Solana Overlay Redesign** - Wallet detection, market data, Liquid Glass styling
- ✅ **Four.meme Overlay Redesign** - BSC chain support with Liquid Glass
- ✅ **Options Page Capabilities** - Full feature showcase

### v0.4 (Planned)
- [ ] DEX transaction building (full assist mode)
- [ ] Wallet signature flow
- [ ] More CEX connectors (Kraken, Coinbase)
- [ ] Strategy backtesting
- [ ] Price impact analysis

### v0.4 (Future)
- [ ] Multi-user support
- [ ] Strategy marketplace
- [ ] Mobile app
- [ ] Cloud deployment options

## Contributing

Contributions are welcome! Please read the contributing guidelines and ensure all tests pass.

## License

MIT License - see [LICENSE](LICENSE) for details.

## Disclaimer

ClawFi is a tool for crypto market monitoring and analysis. It does not guarantee profits and users are responsible for their own trading decisions. Always understand the risks involved in cryptocurrency trading.

---

Built with 🦀 by the ClawFi Team



## License

Apache 2.0 - see [LICENSE](LICENSE)
