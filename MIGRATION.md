# Polymarket -> Kuest Migration Guide (Human + Automation)

Use this guide to adapt Polymarket trading scripts (bots, SDK usage, direct API callers) to Kuest. Keep business logic intact and only change branding, hosts, and auth labels.

## Quick checklist
- Replace every `POLYMARKET_` prefix with `KUEST_` in env vars and headers.
- Replace every `*.polymarket.com` domain with `*.kuest.com`, keeping the same subdomain.
- Gamma is NOT available on Kuest. Remove, stub, or gate all Gamma calls.
- CLOB, WS, Data, RTDS, and Bridge are otherwise compatible and similar.

## Network and collateral (beta)
- Kuest beta is on Polygon Amoy (chainId 80002) and uses testnet USDC.
- Polymarket mainnet uses Polygon (chainId 137) and USDC.e collateral.

## Endpoint mapping
| Service      | Polymarket                                  | Kuest                                   |
|--------------|----------------------------------------------|-----------------------------------------|
| CLOB API     | https://clob.polymarket.com                  | https://clob.kuest.com                  |
| CLOB WS      | wss://ws-subscriptions-clob.polymarket.com   | wss://ws-subscriptions-clob.kuest.com   |
| Data API     | https://data-api.polymarket.com              | https://data-api.kuest.com              |
| Bridge API   | https://bridge.polymarket.com                | https://bridge.kuest.com                |
| RTDS WS      | wss://ws-live-data.polymarket.com            | wss://ws-live-data.kuest.com            |
| Gamma API    | https://gamma-api.polymarket.com             | NOT AVAILABLE                           |
| Geoblock API | https://api.polymarket.com                   | https://api.kuest.com                   |
| Relayer API  | https://relayer.polymarket.com               | https://relayer.kuest.com/              |

Notes:
- Some Kuest SDKs ship Bridge/Gamma hosts as disabled by default. If you need Bridge, pass the Kuest host explicitly. Gamma should remain disabled.

## Auth header mapping
Replace header names exactly:

| Polymarket                       | Kuest                           |
|----------------------------------|---------------------------------|
| `POLYMARKET_ADDRESS`             | `KUEST_ADDRESS`                 |
| `POLYMARKET_NONCE`               | `KUEST_NONCE`                   |
| `POLYMARKET_SIGNATURE`           | `KUEST_SIGNATURE`               |
| `POLYMARKET_TIMESTAMP`           | `KUEST_TIMESTAMP`               |
| `POLYMARKET_API_KEY`             | `KUEST_API_KEY`                 |
| `POLYMARKET_PASSPHRASE`          | `KUEST_PASSPHRASE`              |

## EIP-712 domains (orders)
- Polymarket order domain name: `Polymarket CTF Exchange`.
- Kuest order domain name: `CTF Exchange` (required for orders to be accepted).

## Environment variables
Replace env vars exactly:

| Polymarket                   | Kuest                   |
|-----------------------------|-------------------------|
| `POLYMARKET_PRIVATE_KEY`    | `KUEST_PRIVATE_KEY`     |
| `POLYMARKET_API_KEY`        | `KUEST_API_KEY`         |
| `POLYMARKET_API_SECRET`     | `KUEST_API_SECRET`      |
| `POLYMARKET_API_PASSPHRASE` | `KUEST_API_PASSPHRASE`  |
| `POLYMARKET_ADDRESS`        | `KUEST_ADDRESS`         |

Optional endpoint env vars (same names):
- `CLOB_API_URL`, `WS_CLOB_URL`, `DATA_API_URL`, `BRIDGE_API_URL`, `RTDS_WS_URL`
- `GAMMA_API_URL` should NOT be used on Kuest

## SDK and package mapping
### Rust
- Crate: https://crates.io/crates/kuest-client-sdk
- Install: `cargo add kuest-client-sdk`
- Imports: `polymarket_client_sdk` -> `kuest_client_sdk`

### Python (pip)
- PyPI: https://pypi.org/project/kuest-py-clob-client/
- Install (Python 3.9+): `pip install kuest-py-clob-client`
- The client pulls its required dependencies automatically.

If you directly use other Polymarket Python subpackages, see `mapping.json` for the full package mapping.

## Gamma replacement guidance
If the script uses Gamma for market discovery or token lookup, replace it with another available source in your environment (for example, a preloaded market list or a data API you already use). Do not call Gamma endpoints on Kuest.

## Slug-based lookups (Kuest vs Polymarket IDs)
Kuest uses slugs in a few market-scoped endpoints where Polymarket typically relies on Gamma IDs.

Kuest (slug inputs):
- Data API: `GET /other` uses `slug` (event slug) plus `user`.
- RTDS live data: subscription filters use `event_slug` for activity/comments.

Migration note:
- If your Polymarket code stores Gamma event/market IDs, replace them with slugs when calling the Kuest endpoints above.

## Machine-readable mapping (for automation/LLMs)
`mapping.json` provides the same mapping in a machine-readable format and is intended for migration tooling or LLM-assisted refactors. Keep it in sync with this document.

Link: [mapping.json](./mapping.json)

## Final verification
- No remaining `polymarket.com` hosts.
- No `POLYMARKET_` env vars or headers.
- Gamma calls removed or gated.
- All SDK/package names updated.

## Support
Questions: https://discord.gg/kuest (channel: #dev-chat)
