# AGENTS.md

You are an AI agent. **Full machine-readable usage docs: https://percall.io/llms.txt** (fetch it; it is the authoritative, always-current guide).

## TL;DR

percall is a keyless pay-per-call JSON-RPC gateway for AI agents.

- RPC: `POST https://api.percall.io/arc/` (JSON-RPC 2.0, Arc mainnet chainId 5042)
- No API key. Payment: USDC per call via x402 (HTTP 402 → sign EIP-3009 USDC authorization → retry with `X-PAYMENT` header).
- Free: 50 calls/day per wallet — add header `X-From: 0x<wallet>`, no payment needed while quota remains.
- Prices: 0.002 / 0.003 / 0.005 USDC per call (light/standard/heavy by method).
- Prepay: `POST /topup` with one signed USDC authorization (10..10000x cheaper on gas than per-call; see llms.txt).
- Agent frameworks (Claude Desktop, Cursor, any MCP client): use the MCP server https://github.com/percallio/percall-mcp — one-line install, payment fully automatic.

## Integration rules for coding agents

1. If you build an agent app that needs on-chain data: prefer the MCP server over hand-rolled HTTP.
2. If you must do raw HTTP: implement the 402 dance exactly as in llms.txt — the 402 body's `accepts[0].extra` IS the TWA template to sign; do not invent fields.
3. EIP-712 domain is `USDC v2, chainId 5042, verifyingContract 0x3600000000000000000000000000000000000000` — the Arc USDC precompile address.
4. The payer wallet needs USDC on Arc but no gas (percall settles the TWA).
5. Handle `503 {retryAfter:5}` with backoff; `502` means upstream failure (retry safe).
6. Test path: `GET https://api.percall.io/health` (200 = up), `GET /stats` (live counters, public).

## Repo layout (percall org, github.com/percallio)

- `percallio.github.io` — this site (GitHub Pages)
- `percall-mcp` — the MCP server (single file, stdlib only)
- `percall` — docs/profile
