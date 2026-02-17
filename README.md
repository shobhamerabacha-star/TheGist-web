# GistPin Web

The web frontend for GistPin — a map-based interface for browsing and posting hyperlocal gists.

---

## What This Repo Does

- Renders an interactive map with gist pins pulled from `gistpin-backend`
- Lets users post new gists by clicking a location on the map
- Shows a list view of nearby gists
- (Future) Wallet-based identity via Freighter (Stellar browser extension)

This app is a **pure client** — all blockchain and IPFS logic lives in the backend. The web app only talks to `gistpin-backend` via REST.

---

## Tech Stack

| Layer | Choice |
|---|---|
| Framework | Next.js 14 (App Router) |
| Language | TypeScript |
| Styling | Tailwind CSS |
| Map | Mapbox GL JS (`react-map-gl`) |
| Data fetching | TanStack Query (React Query v5) |
| HTTP client | Axios |
| Forms | React Hook Form + Zod |
| Icons | Lucide React |
| Testing | Jest + React Testing Library |

---

## Project Layout

```
gistpin-web/
├── app/                        # Next.js App Router
│   ├── layout.tsx              # Root layout (providers, fonts)
│   ├── page.tsx                # Home page — mounts MapView
│   ├── gists/
│   │   └── [id]/
│   │       └── page.tsx        # Individual gist detail page
│   └── api/                    # Next.js route handlers (thin proxies if needed)
├── components/
│   ├── map/
│   │   ├── MapView.tsx         # Main Mapbox map wrapper
│   │   ├── GistMarker.tsx      # Individual pin on the map
│   │   └── GistCluster.tsx     # Clustered pins for dense areas
│   ├── gists/
│   │   ├── GistCard.tsx        # Single gist summary card
│   │   ├── GistList.tsx        # Scrollable list of nearby gists
│   │   └── GistForm.tsx        # Create-gist modal / drawer
│   └── ui/                     # Shared primitives (Button, Input, Modal, etc.)
├── lib/
│   ├── api/
│   │   └── client.ts           # Typed Axios instance for gistpin-backend
│   ├── hooks/
│   │   ├── useGists.ts         # TanStack Query hooks for gists
│   │   └── useLocation.ts      # Browser geolocation wrapper
│   └── utils/
│       └── geo.ts              # Small geo helpers (distance, bbox)
├── types/
│   └── gist.ts                 # Shared TypeScript types
├── public/
├── .env.local.example
├── next.config.ts
├── tailwind.config.ts
├── tsconfig.json
├── package.json
└── README.md
```

---

## Prerequisites

- **Node.js** >= 20 — [nodejs.org](https://nodejs.org)
- **npm** 
- A running instance of `gistpin-backend` (see that repo's README)
- A **Mapbox access token** — get one free at [mapbox.com](https://mapbox.com)

---

## Local Development

### 1. Clone and install

```bash
git clone https://github.com/GistPin-Org/gistpin-web.git
cd gistpin-web
npm install
```

### 2. Environment variables

```bash
cp .env.local.example .env.local
```

Open `.env.local` and fill in:

```env
# Backend API base URL
NEXT_PUBLIC_API_BASE_URL=http://localhost:3000

# Mapbox token (public — safe to expose in browser)
NEXT_PUBLIC_MAPBOX_TOKEN=pk.eyJ1Ijoixxxxxxx...

# Default map center (lon, lat) — adjust to your city
NEXT_PUBLIC_DEFAULT_LNG=-0.1870
NEXT_PUBLIC_DEFAULT_LAT=5.6037
NEXT_PUBLIC_DEFAULT_ZOOM=14
```

### 3. Run the dev server

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

---

## Key Patterns

### API client

All calls to `gistpin-backend` go through `lib/api/client.ts`, which exports a typed Axios instance. **Never call `fetch` directly from components.** Keep network logic in hooks.

```ts
// lib/api/client.ts
import axios from 'axios';
import type { Gist, CreateGistDto, GistQueryParams, GistsResponse } from '@/types/gist';

export const api = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_BASE_URL,
});

export const gistsApi = {
  getNearby: (params: GistQueryParams) =>
    api.get<GistsResponse>('/gists', { params }),

  create: (data: CreateGistDto) =>
    api.post<Gist>('/gists', data),
};
```

### Data fetching with TanStack Query

```ts
// lib/hooks/useGists.ts
import { useQuery } from '@tanstack/react-query';
import { gistsApi } from '@/lib/api/client';

export function useGists(lat: number, lon: number, radius = 500) {
  return useQuery({
    queryKey: ['gists', lat, lon, radius],
    queryFn: () => gistsApi.getNearby({ lat, lon, radius }).then(r => r.data),
    enabled: !!lat && !!lon,
  });
}
```

### Map integration

`MapView.tsx` uses `react-map-gl`. On map move / zoom end, it calls `useGists` with the new center coordinates so pins update automatically.

```tsx
// components/map/MapView.tsx (simplified)
'use client';
import Map, { Marker } from 'react-map-gl';
import { useGists } from '@/lib/hooks/useGists';

export default function MapView() {
  const [center, setCenter] = useState({ lat: ..., lon: ... });
  const { data } = useGists(center.lat, center.lon);

  return (
    <Map
      mapboxAccessToken={process.env.NEXT_PUBLIC_MAPBOX_TOKEN}
      onMoveEnd={(e) => setCenter({ lat: e.viewState.latitude, lon: e.viewState.longitude })}
    >
      {data?.data.map(gist => (
        <Marker key={gist.gistId} latitude={gist.lat} longitude={gist.lon}>
          <GistMarker gist={gist} />
        </Marker>
      ))}
    </Map>
  );
}
```

---

## Useful Scripts

| Command | Description |
|---|---|
| `npm dev` | Start dev server with hot reload |
| `npm build` | Production build |
| `npm start` | Serve production build |
| `npm lint` | ESLint |
| `npm type-check` | TypeScript type check without emitting |
| `npm test` | Run Jest tests |

---

## Environment Notes

- `NEXT_PUBLIC_*` variables are exposed to the browser. Never put secrets (API keys with write access, etc.) in these.
- The Mapbox token is public by design — Mapbox restricts tokens by allowed URL in their dashboard.

---

## Contribution Guidelines

- Keep components small and focused on a single responsibility.
- Use TanStack Query for all server state — no `useState` + `useEffect` for data fetching.
- Coordinate any API response shape changes with `gistpin-backend`.
- New components should have at least a smoke test in `__tests__/`.

For global contribution rules: [gistpin-meta/CONTRIBUTING.md](https://github.com/gistpin/gistpin-meta/blob/main/CONTRIBUTING.md)
