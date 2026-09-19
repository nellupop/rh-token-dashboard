# Token Dashboard — Robinhood Chain

A single-page, live-refreshing dashboard for an ERC-20 token on [Robinhood Chain](https://docs.robinhood.com/chain/) (chain ID `4663`): holder count, total supply, top wallets, and 24h transfer activity.

Live at: `https://<your-github-username>.github.io/rh-token-dashboard/` once Pages is enabled (see below).

## What it is

One file, `index.html` — no build step, no dependencies, no framework. Open it locally or drop it on any static host.

**Data sources**

| Data | Source | Why |
|---|---|---|
| Token name, symbol, decimals, total supply | Direct `eth_call` to the Robinhood Chain RPC (`rpc.mainnet.chain.robinhood.com`) | Cheap, single-block reads — pulled straight from the contract |
| Live block ticker | `eth_blockNumber` polled every 4s against the same RPC | Confirms the page is actually talking to the chain, not a cache |
| Top holders, 24h transfers | [Blockscout](https://robinhoodchain.blockscout.com) REST API (the chain's official explorer/indexer) | Enumerating holders or scanning `Transfer` events over 24h requires an indexer — Robinhood Chain produces blocks roughly every 0.1–0.25s, so a 24h range is hundreds of thousands of blocks, which isn't practical to scan live from a browser via raw `eth_getLogs` |

This is a deliberate hybrid, not a shortcut: raw RPC for anything that's a direct contract read, and the chain's own indexer for anything that requires scanning history. Both ultimately read the same chain state.

## Configuring it for a different token

Everything is in one `CONFIG` object near the top of the `<script>` block in `index.html`:

```js
var CONFIG = {
  tokenAddress: "0x583583a74a13e15d40ea2dab8c3a9895e073e4ea",
  chainId: 4663,
  rpcUrl: "https://rpc.mainnet.chain.robinhood.com",
  explorerBase: "https://robinhoodchain.blockscout.com",
  blockscoutApi: "https://robinhoodchain.blockscout.com/api/v2",
  blockPollMs: 4000,      // live block ticker poll interval
  dataPollMs: 30000,      // holders + transfers poll interval
  topWalletsCount: 10,
  transferWindowHours: 24,
  maxTransferPages: 8     // safety cap on pagination through the transfers API
};
```

Swap `tokenAddress` and you're pointed at a different token on the same chain. Swap `rpcUrl` / `blockscoutApi` to move to a different chain entirely (any EVM chain with a Blockscout instance will work with this same code).

## Brand colors

Colors, type, and spacing are CSS custom properties at the top of the `<style>` block (`--bg`, `--green`, `--green-dim`, `--text`, etc.) — I picked a near-black base with a single green accent since you said green/black; swap the hex values there if you've got exact brand hex codes and it'll repaint everywhere consistently. Fonts are Space Grotesk (UI) + IBM Plex Mono (numbers/addresses), loaded from Google Fonts.

## Deploying to GitHub Pages

1. Push this repo (already done if you're reading this from GitHub).
2. In the repo: **Settings → Pages → Source → Deploy from a branch → `main` / root**.
3. Wait a minute or two — it'll publish at `https://<username>.github.io/rh-token-dashboard/`.

No build step, so there's nothing else to configure.

## Rate limits

The Blockscout public API is free but rate-limited. If you start seeing errors in the holders/transfers panels under heavy traffic, get a free key at [dev.blockscout.com](https://dev.blockscout.com) and add it as a query param or header to the `bsGet()` calls in `index.html`.

## A note on the token used above

`0x583583a74a13e15d40ea2dab8c3a9895e073e4ea` was passed in as the target token when this was built — double check it's the right contract (mainnet vs. any testnet deployment) before pointing this at production, since it wasn't independently verified against a known token listing.
