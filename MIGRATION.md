# Polymarket -> Kuest Migration Guide (Human + Automation)

Use this guide to adapt Polymarket trading scripts (bots, SDK usage, direct API callers) to Kuest. Keep business logic intact and only change branding, hosts, and auth labels.

## Quick checklist
- Replace every `POLYMARKET_` prefix with `KUEST_` in env vars and headers.
- Replace every `*.polymarket.com` domain with `*.kuest.com`, keeping the same subdomain.
- Use Exchange EIP-712 domain `name = "CTF Exchange"` and `version = "2"` for orders. Auth/L1/L2 headers keep their existing auth-domain version.
- Remove `taker`, `expiration`, `nonce`, and `feeRateBps` from the signed order struct.
- Add signed order fields `timestamp` in milliseconds, `metadata` bytes32, and `builder` bytes32.
- Keep `expiration` only as an offchain HTTP field for `GTD` orders.
- Send `owner` as the CLOB API key (`KUEST_API_KEY`), not the wallet address.
- In Kuest CLOB responses, deserialize `owner` as a ULID/string user identifier.
- Use `builderCode`/`builder_code` for attribution; Kuest builder codes are builder wallets encoded as bytes32.
- Trading is Deposit Wallet + signature type 3 only.
- Gamma is NOT available on Kuest. Remove, stub, or gate all Gamma calls.
- CLOB, WS, Data, RTDS, and Bridge are otherwise compatible and similar.

## Network and collateral (beta)
- Kuest beta is on Polygon Amoy (chainId 80002) and uses testnet USDC.
- Polymarket V2 uses Polygon (chainId 137) and pUSD collateral; Kuest V2 uses USDC Circle directly.

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
- For V2 discovery and builder flows, wire `GET /version`, `GET /clob-markets/{conditionId}`, `GET /fees/builder-fees/{builderCode}`, and `GET /builder/trades?builder_code=...`.

## Kuest V2 contracts and collateral
| Name | Address |
|------|---------|
| CTF Exchange | `0x4bB1871fdaE80331ce5fF87547b8ff886D1695a5` |
| Neg Risk CTF Exchange | `0xdb1E374a05130d7DE3F16677066553F225D2eE53` |
| NegRisk Adapter | `0xd9416E904e1ab925ad72F03F6D6ce0Aa80fd2dC5` |
| NegRisk Operator | `0xEb330AB2700aCAf8aF73EF12fA4225aE31E137c3` |
| NegRisk UMA CTF Adapter | `0x4b8D05583085716A684E50c2861333d88D85Bbd6` |
| DepositWallet Factory | `0x3DaBe8f032833CE42CC26d9149660E6f596759C5` |
| DepositWallet Impl | `0xFB2f5D822Ecb062dE63a7B830C5e83C994698851` |
| USDC Circle (Amoy) | `0x41E94Eb019C0762f9Bfcf9Fb1E58725BfB0e7582` |
| USDC Circle (Polygon mainnet) | `0x3c499c542cef5e3811e1192ce70d8cc03d5c3359` |

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
- Kuest order domain name: `CTF Exchange` and version `2` (required for orders to be accepted).

## Signed order payload V2
Remove these fields from the signed payload:
- `taker`
- `expiration`
- `nonce`
- `feeRateBps`

Add these fields to the signed payload:
- `timestamp` as a millisecond timestamp
- `metadata` as bytes32
- `builder` as bytes32

Do not send fee basis points in the order signature. The CLOB calculates Kuest + builder fees and sends absolute USDC fee amounts to the V2 exchanges during settlement.

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

### Builder relayer clients
- TypeScript: `@polymarket/builder-relayer-client` -> `@kuestcom/builder-relayer-client`
- Python: `py-builder-relayer-client` -> `kuest-py-builder-relayer-client`
- Use only Deposit Wallet methods: derive, deploy, execute batch, and transaction polling.

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
