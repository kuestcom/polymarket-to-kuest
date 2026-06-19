# Polymarket -> Kuest compatibility

<p align="center">
  <a href="https://kuest.com">
    <img src="https://github.com/user-attachments/assets/1f608405-c381-4098-88dc-0c37cebb6038" alt="Polymarket to Kuest migration" />
  </a>
</p>

If you already have bots or tooling built for Polymarket, migrating to Kuest
requires only small changes to endpoints, headers, and environment variables.

- Python SDK: https://pypi.org/project/kuest-py-clob-client/
- Rust SDK: https://crates.io/crates/kuest-client-sdk

## What's here
- [`MIGRATION.md`](./MIGRATION.md) — step-by-step migration guide for humans.
- [`mapping.json`](./mapping.json) — machine-readable mapping for automation or LLM-assisted refactors.
- [`AGENTS.md`](./AGENTS.md) — compact instructions for coding agents and migration tooling.

## Quick notes
- Replace `POLYMARKET_` with `KUEST_` in env vars and auth headers.
- Replace `*.polymarket.com` with `*.kuest.com` (same subdomain).
- Kuest order signing uses the EIP-712 domain `CTF Exchange`, version `2`.
- V2 orders remove `taker`, `expiration`, `nonce`, and `feeRateBps` from the signed payload and add `timestamp`, `metadata`, and `builder`.
- Send `owner` as the CLOB API key, not the wallet address from `KUEST_ADDRESS`.
- In Kuest CLOB order responses, deserialize `owner` as a ULID/string user identifier.
- Use `builderCode`/`builder_code` for attribution; Kuest encodes a builder wallet as `bytes32(uint256(uint160(wallet)))`.
- For direct relayer calls, use `WALLET` / `WALLET-CREATE` instead of `SAFE` / `PROXY`.

## Network (beta)
- Kuest beta runs on Polygon Amoy (chainId 80002) and uses testnet USDC.
- Polymarket V2 uses Polygon (chainId 137) and pUSD collateral; Kuest V2 uses USDC Circle directly.

## Kuest V2 contracts and collateral
- CTF Exchange: `0xaCd95F4F42322c7bE215C170362EEc57Ef4E78c2`
- Neg Risk CTF Exchange: `0x961d3230B3BBdb2592D20fa34dBD12Fa19240603`
- USDC Circle (Amoy): `0x41E94Eb019C0762f9Bfcf9Fb1E58725BfB0e7582`
- USDC Circle (Polygon mainnet): `0x3c499c542cef5e3811e1192ce70d8cc03d5c3359`

For a step-by-step migration guide, see [MIGRATION.md](./MIGRATION.md).
