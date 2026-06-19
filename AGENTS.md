# Migration Agent Notes

Purpose:
- Help agents migrate Polymarket bots, SDK integrations, and direct API clients to Kuest with minimal behavior changes.

Primary rule:
- Keep business logic intact. Only change hosts, headers, env vars, package names, and network-specific details required for Kuest.

Required migration rules:
- Replace every `POLYMARKET_` env var or header prefix with `KUEST_`.
- Replace every `*.polymarket.com` host with the matching `*.kuest.com` host.
- Kuest order signing uses the EIP-712 domain name `CTF Exchange`.
- Kuest order signing uses Exchange domain version `2`; auth headers keep the auth-domain version used by the SDK.
- V2 signed orders remove `taker`, `expiration`, `nonce`, and `feeRateBps`.
- V2 signed orders add `timestamp`, `metadata`, and `builder`.
- Send order `owner` as the CLOB API key, not `KUEST_ADDRESS`.
- In Kuest CLOB responses, deserialize `owner` as a ULID/string user identifier.
- Use `builderCode`/`builder_code` for attribution; Kuest encodes builder wallets as bytes32.
- Direct relayer calls use `WALLET` / `WALLET-CREATE` instead of `SAFE` / `PROXY`.
- Do not keep `Polymarket CTF Exchange` when building Kuest order signatures.

SDK/package mapping:
- Rust crate: `polymarket-client-sdk` -> `kuest-client-sdk`
- Rust import: `polymarket_client_sdk` -> `kuest_client_sdk`
- Python package: `py-clob-client` -> `kuest-py-clob-client`
- Python package: `py-order-utils` -> `kuest-py-order-utils`
- Python package: `py-builder-signing-sdk` -> `kuest-py-builder-signing-sdk`
- Python package: `py-eip712-structs` -> `kuest-py-eip712-structs`
- Python import rename required: `poly_eip712_structs` -> `kuest_eip712_structs`
- Python imports unchanged: `py_clob_client`, `py_order_utils`, `py_builder_signing_sdk`

Network rules:
- Kuest beta uses Polygon Amoy, chainId `80002`
- Polymarket mainnet uses Polygon, chainId `137`
- Kuest uses USDC Circle directly
- Polymarket V2 uses pUSD collateral on mainnet
- Kuest CTF Exchange: `0xaCd95F4F42322c7bE215C170362EEc57Ef4E78c2`
- Kuest Neg Risk CTF Exchange: `0x961d3230B3BBdb2592D20fa34dBD12Fa19240603`

Endpoint notes:
- Keep the same Kuest subdomain when replacing a Polymarket host.
- Wire V2 helper endpoints when SDK code calls them: `/version`, `/clob-markets/{conditionId}`, `/fees/builder-fees/{builderCode}`, and `/builder/trades?builder_code=...`.
- Kuest uses slugs in some places where Polymarket code often uses Gamma IDs.
- In particular, market-scoped Kuest lookups may require event slugs for Data API and RTDS flows.

Recommended agent workflow:
1. Replace env vars and auth headers.
2. Replace domains and websocket hosts.
3. Update SDK package names and imports.
4. Fix EIP-712 order domain to `CTF Exchange`.
5. Check whether stored Gamma IDs need to become slugs.
6. Run final grep checks for leftover `POLYMARKET_`, `polymarket.com`, and `Polymarket CTF Exchange`.

Repo files:
- `README.md` gives the short overview.
- `MIGRATION.md` is the human migration guide.
- `mapping.json` is the machine-readable mapping for codemods and automation.
