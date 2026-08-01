---
name: Send and monitor an on-chain transaction
description: Broadcast an EVM transaction through Syndicate's Transaction Cloud and poll it to confirmation.
api: openapi/syndicate-transaction-cloud-openapi-original.yml
operations: [send-transaction, get-transaction-request]
---

# Send and monitor an on-chain transaction

Broadcast a contract call on any supported EVM chain without managing keys,
nonces, or gas. Syndicate signs from an HSM-managed project wallet and
guarantees the transaction lands on-chain.

## Prerequisites
- A project API key (`Authorization: Bearer <api-key>`).
- The target contract has been authorized for the project (see the
  "Authorize a contract then broadcast a transaction" skill).
- Base URL: `https://api.syndicate.io`.

## Steps

1. **Broadcast** — call `send-transaction` (`POST /transact/sendTransaction`).
   Body: `projectId`, `contractAddress`, `chainId`, `functionSignature`
   (human-readable ABI, e.g. `mint(address account)`), and `args`.
   - Supply a `requestId` (a UUID you generate) to make the call **idempotent** —
     retrying with the same `requestId` will not double-broadcast (a duplicate
     returns HTTP `409`). If you omit it, Syndicate returns the generated one.
   - The response returns the `transactionId` / `requestId`.

2. **Monitor** — poll `get-transaction-request`
   (`GET /wallet/project/{projectId}/request/{transactionId}`) until the status
   reaches `CONFIRMED`. Statuses progress `PENDING → SUBMITTED → PROCESSED →
   CONFIRMED`. The response includes `transactionAttempts[]` with `hash`,
   `nonce`, `reverted`, and `status`.

3. **React** — for push-style updates instead of polling, subscribe to the
   `TransactionStatusChange` webhook (verify the `Syndicate-Signature`
   HMAC-SHA256 header; reject events older than 5 minutes).

## Rules
- Idempotency: always send a stable `requestId` for retriable writes.
- Errors: `401` bad key, `403` IP not allowlisted / no project access,
  `404` unknown projectId/transactionId, `409` duplicate requestId. Envelope is
  `{ "message": "..." }` (see `errors/syndicate-problem-types.yml`).
- Use a testnet `chainId` (e.g. `84532` Base Sepolia) to test safely.
