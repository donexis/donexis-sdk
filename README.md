# Donexis SDK

Multi-chain donation infrastructure SDK for accepting cryptocurrency donations on EVM and Solana blockchains.

[![npm version](https://badge.fury.io/js/@donexis%2Fsdk.svg)](https://www.npmjs.com/package/@donexis/sdk)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

## Features

- 🌐 **Multi-Chain Support** - EVM (Ethereum, Base) and Solana
- ⚡ **Real-Time Updates** - WebSocket notifications for instant donation alerts
- 🎨 **Custom Overlays** - Built-in OBS overlay system for streamers
- 🔒 **Type-Safe** - Full TypeScript support with comprehensive types
- 💎 **Token Benefits** - Reduced fees with $DNX token holdings
- 📦 **Production Ready** - Battle-tested infrastructure

## Installation

```bash
npm install @donexis/sdk
```

## Quick Start

```typescript
import { DonexisClient } from '@donexis/sdk';

// Initialize the client
const client = new DonexisClient({
  apiKey: 'your-api-key',
  environment: 'testnet' // or 'mainnet'
});

// Create a donation session
const session = await client.createSession({
  projectId: 'your-project-id',
  recipientAddress: '0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb',
  chainId: 11155111, // Sepolia
  metadata: {
    streamTitle: 'My Stream',
    streamerName: 'YourName'
  }
});

console.log('Donation URL:', client.getDonationUrl(session.sessionId));

// Subscribe to real-time donation updates
client.subscribeToSession(session.sessionId);

client.on('donation', (donation) => {
  console.log('New donation received!', donation);
});

client.on('connected', () => {
  console.log('WebSocket connected');
});
```

## Supported Chains

| Chain | Network | Chain ID | Status |
|-------|---------|----------|--------|
| Ethereum | Sepolia | 11155111 | ✅ Live |
| Base | Mainnet | 8453 | ✅ Live |
| Solana | Devnet | - | ✅ Live |

## API Reference

### DonexisClient

#### Constructor

```typescript
new DonexisClient(config: DonexisConfig)
```

**Config Options:**
- `apiKey` (string, required) - Your API key from the dashboard
- `environment` ('testnet' | 'mainnet', required) - Network environment
- `baseUrl` (string, optional) - Custom API URL
- `wsUrl` (string, optional) - Custom WebSocket URL
- `customRpcUrls` (object, optional) - Custom RPC endpoints

#### Methods

##### createSession(params)
Create a new donation session.

```typescript
const session = await client.createSession({
  projectId: 'project-id',
  recipientAddress: '0x...',
  chainId: 11155111,
  metadata: {
    streamTitle: 'My Stream',
    streamerName: 'YourName'
  }
});
```

##### getSession(sessionId)
Retrieve session details.

```typescript
const session = await client.getSession('session-id');
```

##### getDonations(sessionId)
Get all donations for a session.

```typescript
const donations = await client.getDonations('session-id');
```

##### subscribeToSession(sessionId)
Subscribe to real-time donation updates via WebSocket.

```typescript
client.subscribeToSession('session-id');

client.on('donation', (donation) => {
  console.log('New donation:', donation);
});
```

##### unsubscribe()
Disconnect from WebSocket.

```typescript
client.unsubscribe();
```

##### linkWallet(walletAddress)
Link a wallet to your API key for token benefits.

```typescript
await client.linkWallet('0x...');
```

##### getDonationUrl(sessionId)
Get the donation page URL for a session.

```typescript
const url = client.getDonationUrl('session-id');
```

## Events

The client extends EventEmitter and emits the following events:

- `donation` - New donation received
- `connected` - WebSocket connected
- `disconnected` - WebSocket disconnected
- `error` - Error occurred

```typescript
client.on('donation', (data) => {
  console.log('Donation:', data);
});

client.on('error', (error) => {
  console.error('Error:', error);
});
```

## EVM Provider

For direct blockchain interactions:

```typescript
import { EVMProvider } from '@donexis/sdk';

const provider = new EVMProvider({
  sepolia: 'https://eth-sepolia.g.alchemy.com/v2/YOUR-KEY',
  base: 'https://mainnet.base.org'
});

// Get donation contract address
const contractAddress = provider.getContractAddress(11155111);

// Verify a donation on-chain
const isValid = await provider.verifyDonation(
  11155111,
  'tx-hash',
  'recipient-address',
  'amount'
);
```

## Solana Provider

```typescript
import { SolanaProvider } from '@donexis/sdk';

const provider = new SolanaProvider({
  solana: 'https://api.devnet.solana.com'
});

// Get program ID
const programId = provider.getProgramId();

// Verify a Solana donation
const isValid = await provider.verifyDonation(
  'signature',
  'recipient-pubkey',
  'amount'
);
```

## Token Benefits

Hold $DNX tokens to unlock benefits:

| Token Balance | Platform Fee | Benefits |
|---------------|--------------|----------|
| 0 - 49,999 | 2% | Standard features |
| 50,000+ | 0% | No fees + Premium overlays |

Link your wallet to activate:

```typescript
await client.linkWallet('your-wallet-address');
```

## Examples

### Streamer Integration

```typescript
import { DonexisClient } from '@donexis/sdk';

const client = new DonexisClient({
  apiKey: process.env.DONEXIS_API_KEY!,
  environment: 'mainnet'
});

// Create session for stream
const session = await client.createSession({
  projectId: 'my-stream',
  recipientAddress: '0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb',
  chainId: 8453, // Base
  metadata: {
    streamTitle: 'Epic Gaming Stream',
    streamerName: 'ProGamer123'
  }
});

// Show donation URL in stream description
console.log('Donate at:', client.getDonationUrl(session.sessionId));

// Listen for donations
client.subscribeToSession(session.sessionId);

client.on('donation', (donation) => {
  // Trigger OBS alert
  showDonationAlert(donation);
  
  // Thank donor
  console.log(`Thanks ${donation.metadata?.message || 'Anonymous'}!`);
});
```

### Next.js Integration

```typescript
// app/api/donations/route.ts
import { DonexisClient } from '@donexis/sdk';

const client = new DonexisClient({
  apiKey: process.env.DONEXIS_API_KEY!,
  environment: 'mainnet'
});

export async function POST(req: Request) {
  const { recipientAddress, chainId } = await req.json();
  
  const session = await client.createSession({
    projectId: 'my-project',
    recipientAddress,
    chainId
  });
  
  return Response.json({
    sessionId: session.sessionId,
    donationUrl: client.getDonationUrl(session.sessionId)
  });
}
```

## Error Handling

```typescript
try {
  const session = await client.createSession({
    projectId: 'my-project',
    recipientAddress: '0x...',
    chainId: 11155111
  });
} catch (error) {
  if (error.response?.status === 429) {
    console.error('Rate limit exceeded');
  } else if (error.response?.status === 401) {
    console.error('Invalid API key');
  } else {
    console.error('Error:', error.message);
  }
}
```

## Rate Limits

| Tier | Requests/Minute |
|------|-----------------|
| Free | 100 |
| Premium | 1,000 |
| Enterprise | Custom |

Check your current limits:

```typescript
const limits = await client.getRateLimitInfo();
console.log('Remaining:', limits.remaining);
```

## TypeScript Support

Full TypeScript support with comprehensive type definitions:

```typescript
import type {
  DonexisConfig,
  Session,
  Donation,
  CreateSessionParams,
  DonationEventData
} from '@donexis/sdk';
```

## Contributing

Donexis SDK is open-source! Contributions are welcome.

- GitHub: [github.com/donexis](https://github.com/donexis)
- Issues: [github.com/donexis/sdk/issues](https://github.com/donexis/sdk/issues)

## License

Apache 2.0 - See [LICENSE](LICENSE) file for details.

## Links

- 🌐 Website: Coming Soon
- 🐦 Twitter: [@donexis_sdk](https://x.com/donexis_sdk)
- 📚 Documentation: Coming Soon
- 🎮 Demo: [testnet.zidanmutaqin.dev](https://testnet.zidanmutaqin.dev)

## Support

- Email: support@donexis.io
- Twitter: [@donexis_sdk](https://x.com/donexis_sdk)
- GitHub Issues: [github.com/donexis/sdk/issues](https://github.com/donexis/sdk/issues)

---

Built with ❤️ by the Donexis team
