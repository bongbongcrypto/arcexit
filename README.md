# Arc Bridge

Non-custodial USDC bridge for Arc mainnet (chain 5042), built on Circle's
CCTP v2. Everything is signed in your own wallet; this site holds no funds,
runs no server, and stores no user data.

## Why there are two helper contracts

Arc caps a CCTP message at **1 USDC**. A 400 USDC withdrawal is therefore 400
separate burns on Arc and 400 separate collections on Base. Signing those one
at a time is unusable, and wallet-side batching turned out to be capped at an
undocumented size that wallets refuse past. So each side gets one contract that
loops internally:

| Contract | Chain | Address | What it does |
|---|---|---|---|
| `ArcBatchExit` | Arc | `0x83839c096c71f3af1732a3685adc1c33db8f661d` | Burns in a loop, takes the service fee |
| `ArcClaimBatch` | Base | `0x174a6cf1382a2e496dbc642461711b59e5d4e0b7` | Calls `receiveMessage` in a loop |

Neither custodies funds.

- `ArcBatchExit` pulls exactly principal + fee and has **no** withdraw, rescue
  or sweep function. The fee is bounded by an immutable `MAX_FEE_BPS = 500`.
- `ArcClaimBatch` has **no owner and no admin function** and never touches a
  token. Funds go to the `mintRecipient` sealed inside each Circle-attested
  message, which the contract cannot read from or alter.

## Safety properties

- Approvals are always for the exact amount being bridged. The site never
  requests an unlimited allowance and never calls `setApprovalForAll`.
- No `eth_sign`, no blanket `permit`, no `transferFrom` on user holdings.
- Content-Security-Policy enumerates every host the page may reach;
  `X-Frame-Options: DENY`.
- Single static file. No backend, no analytics, no third-party scripts.

Disclosure: [`/.well-known/security.txt`](https://arcexit.com/.well-known/security.txt)
