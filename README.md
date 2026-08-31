# Polymarket -> Kuest compatibility

<p align="center">
  <a href="https://kuest.com">
    <img src="https://github.com/user-attachments/assets/1f608405-c381-4098-88dc-0c37cebb6038" alt="Polymarket to Kuest migration" />
  </a>
</p>

Migrating Polymarket bots to Kuest requires endpoint, authentication, network,
and V2 signed-order changes.

- TypeScript SDK: `@kuestcom/clob-client`
- Rust SDK: https://crates.io/crates/kuest-client-sdk
- Python SDK: https://pypi.org/project/kuest-py-clob-client/

## What's here
- [`MIGRATION.md`](./MIGRATION.md) — step-by-step migration guide for humans.
- [`mapping.json`](./mapping.json) — machine-readable mapping for automation or LLM-assisted refactors.
- [`AGENTS.md`](./AGENTS.md) — compact instructions for coding agents and migration tooling.

## Quick notes
- Replace `POLYMARKET_` with `KUEST_` in env vars and auth headers.
- Replace `*.polymarket.com` with `*.kuest.com` (same subdomain).
- Kuest order signing uses the EIP-712 domain `CTF Exchange`, version `2`.
- V2 orders remove `taker`, `expiration`, `nonce`, and `feeRateBps` from the signed payload and add `timestamp`, `metadata`, and `builder`.
- Use Deposit Wallet as `maker` and `signer`, with signature type `3`.
- Send `owner` as the CLOB API key, not `KUEST_ADDRESS`.
- In Kuest CLOB order responses, deserialize `owner` as a ULID/string user identifier.
- Use `builderCode`/`builder_code` for attribution; Kuest encodes a builder wallet as `bytes32(uint256(uint160(wallet)))`.
- Deploy the Deposit Wallet before posting orders. Relayer calls require Builder credentials and use `WALLET` / `WALLET-CREATE`.

## Network (beta)
- Kuest beta runs on Polygon Amoy (chainId 80002) and uses testnet USDC.
- Polymarket V2 uses Polygon (chainId 137) and pUSD collateral; Kuest V2 uses USDC Circle directly.

## Kuest V2 contracts and collateral
- CTF Exchange: `0xaa1b8dE834E16eC69C044F5300041673C968c9eF`
- Neg Risk CTF Exchange: `0xe7FA09cA716FDf498d74AFF618d32AFeacc310aB`
- USDC Circle (Amoy): `0x41E94Eb019C0762f9Bfcf9Fb1E58725BfB0e7582`
- USDC Circle (Polygon mainnet): `0x3c499c542cef5e3811e1192ce70d8cc03d5c3359`

For the full migration guide, see [MIGRATION.md](./MIGRATION.md).
