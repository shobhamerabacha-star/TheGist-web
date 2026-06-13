# TheGist Web

Decentralized map interface for the TheGist protocol — post and browse anonymous hyperlocal gists in the browser.

No user accounts. No login. Connect your Freighter wallet and start posting.

---

## How It Works

TheGist Web is a fully client-side Web3 app. There is no backend that holds your identity or mediates your posts.

```
1. Connect Freighter wallet (browser extension)
2. Compose a gist — text or image
3. Content is pinned to IPFS directly from the browser → CID returned
4. Transaction (CID + geohash + author pubkey) signed by Freighter
5. Signed transaction submitted directly to Soroban GistRegistry contract
6. Map reads nearby gists from the GraphQL indexer (TheGist-API)
7. Gist appears on the map for anyone nearby
```

All write operations go directly to Soroban — this app never routes your transaction through a server.

---

## Stack

| Layer | Choice |
|-------|--------|
| Framework | Next.js 14 (App Router) |
| Language | TypeScript |
| Styling | Tailwind CSS |
| Map | Mapbox GL JS |
| Wallet | Freighter Wallet SDK |
| Blockchain | Stellar SDK |
| Data fetching | Apollo Client |
| GraphQL | TheGist-API |

---

## Key Features

- **Interactive map** — gists rendered as pins, clustered at low zoom levels
- **Post at your location** — one click to anchor a gist to your current GPS coordinates
- **24h expiry by default** — gists self-expire on-chain; the map stays fresh
- **Anonymous by default** — no username required, though author Stellar address is visible on-chain to anyone who looks
- **Freighter wallet** — sign transactions in the browser with no seed phrase exposure
- **IPFS pinning** — gist content lives on IPFS, not on any server controlled by TheGist

---

## Local Setup

### 1. Install Freighter

Download the [Freighter browser extension](https://freighter.app) and create or import a Stellar wallet. Switch to **Testnet** during development.

### 2. Clone and install

```bash
git clone https://github.com/TheGist-Org/TheGist-web.git
cd TheGist-web
npm install
```

### 3. Environment variables

```bash
cp .env.local.example .env.local
```

```env
NEXT_PUBLIC_SOROBAN_RPC_URL=https://soroban-testnet.stellar.org
NEXT_PUBLIC_CONTRACT_ID=<your_deployed_gist_registry_contract_id>
NEXT_PUBLIC_MAPBOX_TOKEN=pk.eyJ1Ijoixxxxxxx...
NEXT_PUBLIC_GRAPHQL_URL=http://localhost:4000/graphql
```

Get a free Mapbox token at [mapbox.com](https://www.mapbox.com).

### 4. Start the dev server

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

---

## Project Layout

```
TheGist-web/
├── app/                         # Next.js App Router
│   ├── layout.tsx               # Root layout (wallet provider, Apollo)
│   ├── page.tsx                 # Home — mounts the map
│   └── gist/[id]/page.tsx       # Gist detail page
├── components/
│   ├── map/
│   │   ├── GistMap.tsx          # Mapbox GL wrapper
│   │   └── GistPin.tsx          # Individual map pin
│   ├── gists/
│   │   ├── GistCard.tsx         # Gist summary card
│   │   └── PostGistModal.tsx    # Compose and sign a new gist
│   └── wallet/
│       └── ConnectButton.tsx    # Freighter connect / disconnect
├── lib/
│   ├── soroban.ts               # Transaction building and submission
│   ├── ipfs.ts                  # IPFS pinning (browser)
│   └── graphql/                 # Apollo Client setup and queries
├── .env.local.example
├── next.config.ts
├── tailwind.config.ts
└── README.md
```

---

## Useful Scripts

| Command | Description |
|---------|-------------|
| `npm run dev` | Start dev server with hot reload |
| `npm run build` | Production build |
| `npm run lint` | ESLint |
| `npm run type-check` | TypeScript check without emitting |

---

## Contributing

- All blockchain interactions must go through `lib/soroban.ts` — no raw SDK calls in components.
- IPFS pinning logic lives in `lib/ipfs.ts` — do not call any centralized storage from the client.
- Any changes to the GraphQL query shapes must be coordinated with TheGist-API.

For global contribution rules, see [CONTRIBUTING.md](https://github.com/TheGist-Org/theGist-Meta/blob/main/CONTRIBUTING.md).

---

## License

MIT
