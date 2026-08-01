---
name: Authorize a contract then broadcast a transaction
description: Create a project, authorize a contract's function signatures, then send an on-chain transaction.
api: openapi/syndicate-transaction-cloud-openapi-original.yml
operations: [create-project, authorize-contract-with-function-signatures, send-transaction, get-transaction-request]
---

# Authorize a contract then broadcast a transaction

Full setup path: stand up a project, whitelist the contract functions your app
will call, then broadcast. Contract authorization is what lets Syndicate's
managed wallet call your contract.

## Prerequisites
- An **environment-wide** API key (required to create projects).
- Base URL: `https://api.syndicate.io`.

## Steps

1. **Create a project** — call `create-project` (`POST /admin/project`) with an
   environment-wide key. Returns the `projectId` and an auto-generated
   transaction wallet. (Skip if you already have a project.)

2. **Authorize the contract** — call
   `authorize-contract-with-function-signatures`
   (`POST /admin/contract/authorizeWithFunctionSignatures`) with the
   `chainId`, `contractAddress`, and the human-readable `functionSignatures[]`
   you intend to call. (Alternatively use
   `authorize-contract-with-jsonabi` to authorize from a full JSON ABI.)

3. **Broadcast** — call `send-transaction`
   (`POST /transact/sendTransaction`) with `projectId`, `contractAddress`,
   `chainId`, `functionSignature`, `args`, and an idempotent `requestId` (UUID).

4. **Confirm** — poll `get-transaction-request`
   (`GET /wallet/project/{projectId}/request/{transactionId}`) to `CONFIRMED`.

## Rules
- Only authorized `contractAddress` + `functionSignature` pairs can be
  broadcast; an unauthorized call is rejected (`403`/`422`).
- Optionally restrict the project to allowed IP ranges via
  `create-allowed-ip-range` (`POST /admin/allowedIP`).
- Keep the environment-wide key server-side only.
- See `conventions/syndicate-conventions.yml` and
  `errors/syndicate-problem-types.yml`.
