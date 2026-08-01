---
name: Sign EIP-712 typed data with a project wallet
description: Produce an EIP-712 typed-data signature from a Syndicate-managed project wallet and retrieve it by id.
api: openapi/syndicate-transaction-cloud-openapi-original.yml
operations: [sign-typed-data, get-signature]
---

# Sign EIP-712 typed data with a project wallet

Use a Syndicate HSM-managed project wallet to produce an EIP-712 typed-data
signature (e.g. for permits, meta-transactions, or off-chain attestations)
without ever handling the private key.

## Prerequisites
- A project API key (`Authorization: Bearer <api-key>`).
- The `signerAddress` is a wallet that belongs to the project (list them with
  `get-wallets-by-project`).
- Base URL: `https://api.syndicate.io`.

## Steps

1. **Sign** — call `sign-typed-data`
   (`POST /wallet/project/{projectId}/signTypedData`) with:
   - `signerAddress` — the project wallet that signs.
   - `domain` — the EIP-712 `EIP712Domain` (at least `chainId`).
   - `types` — type definitions including `EIP712Domain` and your `primaryType`.
   - `primaryType` and `message` — the structured data to sign.
   - Optionally a `signatureId` (UUID) to make the request **idempotent**.

2. **Retrieve** — fetch the stored signature later with `get-signature`
   (`GET /wallet/project/{projectId}/signature/{signatureId}`).

## Notes
- For personal (EIP-191) message signing instead, use `personal-sign`
  (`POST /wallet/project/{projectId}/personalSign`).
- For conditional signing gated on on-chain reads, use
  `sign-typed-data-with-lookup` (up to 5 contract lookups with eq/gt/lt).
- Idempotency and error semantics: see `conventions/syndicate-conventions.yml`
  and `errors/syndicate-problem-types.yml`.
