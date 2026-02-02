# Polymarket -> Kuest compatibility

<p align="center">
  <a href="https://kuest.com">
    <img src="https://i.imgur.com/D4sTGrV.png" alt="Kuest" />
  </a>
</p>

If you already have bots or tooling built for Polymarket, migrating to Kuest
requires only small changes to endpoints, headers, and environment variables.

- Python SDK: https://pypi.org/project/kuest-py-clob-client/
- Rust SDK: https://crates.io/crates/kuest-client-sdk

## What's here
- [`MIGRATION.md`](./MIGRATION.md) — step-by-step migration guide for humans.
- [`mapping.json`](./mapping.json) — machine-readable mapping for automation or LLM-assisted refactors.

## Quick notes
- Replace `POLYMARKET_` with `KUEST_` in env vars and auth headers.
- Replace `*.polymarket.com` with `*.kuest.com` (same subdomain).
- Gamma API is not available on Kuest.

## Network (beta)
- Kuest beta runs on Polygon Amoy (chainId 80002) and uses testnet USDC.
- Polymarket mainnet uses Polygon (chainId 137) and USDC.e collateral.

For a step-by-step migration guide, see [MIGRATION.md](./MIGRATION.md).
