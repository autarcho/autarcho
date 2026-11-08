# Autarcho API

OpenAI-compatible inference on a decentralised network of independent operators. You pay per
token generated, in USDC on Solana. There is no signup, no subscription and no card.

- Base URL: `https://api.autarcho.com/v1`
- Live state: <https://api.autarcho.com/observe>
- Site: <https://autarcho.com>
- Support: <https://discord.gg/4qx4rsCSE>

**This repository is documentation only.** The gateway, worker and payout software are
proprietary. Nothing here is a build of them, and there is no source code to clone.

## Quick start

```bash
KEY="ak_$(openssl rand -hex 32)"

curl https://api.autarcho.com/v1/chat/completions \
  -H "Authorization: Bearer $KEY" \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "lfm25-350m",
    "messages": [{"role": "user", "content": "Reply with the single word: ready"}],
    "max_tokens": 16
  }'
```

A key is a string the client makes up: `ak_` followed by 64 lowercase hex characters. The gateway
stores only its SHA-256, so a key cannot be recovered if lost. Present one the network has never
seen and it is issued a single trial grant of 0.05 USDC, which expires after 30 days. That grant is
rate-limited and finite: the network issues at most twenty per hour.

Any OpenAI-compatible client works. Point it at the base URL above with the key as a bearer token.

```bash
OPENAI_BASE_URL=https://api.autarcho.com/v1
OPENAI_API_KEY=ak_...
```

## Models

`GET /v1/models` lists what is registered and verified. Whether a request is served depends on an
operator being online for that model, so a listed model is not a promise of capacity.

Each GGUF file is named by its SHA-256 and checked against that hash on download. A file with
different bytes is a different model.

## Pricing

The price is set per model by the operator who registered it. It is shown before you send:

```bash
curl "https://api.autarcho.com/model/<model-hash>/price?for_client=true"
```

Billing is on generated tokens only. **Input tokens are not charged**, so a long document with a
short answer costs the answer. Operators keep 80% of each job; the remaining 20% is the protocol
fee.

Every request may carry `max_price_per_token_micro_usdc`. A request whose price exceeds that
ceiling is refused with `402` before any credit is reserved.

## What this is not

- Not confidential. A prompt runs on an independent operator's machine, which can read it.
- Not uniform. Latency and context window depend on which operator is serving.
- Not a guarantee of capacity. Models go offline when operators do.

## Deposits

Credit is bought with USDC on Solana. `POST /deposits/intent` returns a reference and an address;
the watcher credits the balance once the transfer confirms. Card payment is not supported.

## Documentation

- `openapi.yaml` — the HTTP surface.
- <https://autarcho.com/workers/> — how to serve models and be paid.
