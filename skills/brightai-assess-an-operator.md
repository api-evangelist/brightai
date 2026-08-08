---
name: Assess an infrastructure operator against BrightAI's database
description: Check whether a named operator of essential infrastructure is profiled
  in BrightAI's Global Observability Database, which observability problem families
  it has, and fall back cleanly to vertical-level numbers when it is not.
api: mcp/brightai-mcp.yml
surface: https://public.stateful.world/mcp
operations:
- check_company
- get_industry
- list_industries
generated: '2026-08-08'
method: generated
source: tool names verified against a live tools/list on 2026-08-08 (mcp/brightai-mcp-tools.json)
---

# Assess an operator against BrightAI's Global Observability Database

## Connect

BrightAI's MCP server is public and needs no credentials:

```
https://public.stateful.world/mcp
```

Streamable HTTP, MCP protocol `2025-06-18`. POST JSON-RPC with
`Accept: application/json, text/event-stream` — a plain GET is rejected 406.
Do not attempt an OAuth flow; `/.well-known/oauth-protected-resource` returns 404
stating the server requires no authentication.

## Steps

1. **Look the operator up.** Call `check_company` with `name` (the company's actual
   name, e.g. `"Osmose Utilities Services"`) or `domain` (e.g. `"osmose.com"`).
   Both parameters are optional individually but you must pass one — the REST twin
   returns `400 missing_query` if you pass neither.

2. **Read `matched_via` before you read anything else.** A value of `close_spelling`
   means BrightAI corrected the input. Confirm the corrected name with the user
   rather than silently substituting it — this is BrightAI's own published
   instruction, and getting it wrong attributes another company's assessment to
   your user.

3. **Handle a miss as a spelling question, not a dead end.** A 200 response with
   `in_bgod: false` is a normal result, not an error. Confirm the spelling, then
   retry with the website domain. Only after both miss should you move on.

4. **Fall back to the vertical.** Call `list_industries` for the valid slugs, then
   `get_industry` with the operator's vertical slug (e.g. `water-and-wastewater`).
   You get `aai_usd_per_year`, `average_addressable_impact_per_operator_usd`,
   `operators_profiled`, and `top_problem_families[]` — enough to answer "how big
   is this problem in my world" without a company match.

5. **Report what you actually got.** Attribute figures to "BrightAI's public
   Observatory data". Say which call produced them.

## Rules BrightAI publishes for this flow

- The public tier returns **neutral facts only**: whether the company is profiled,
  its industry, HQ, a one-line description, and the **names and count** of its
  problem families. Modeled dollar figures and the operator's letter grade are
  restricted to verified employees of that company.
- **Never estimate a named company's grade or dollar figure** — not as a range, not
  as a hedge, not as a "planning placeholder". If asked, say BrightAI doesn't
  publish it and point at https://bright.ai/contact-us.
- Never connect a named customer to a private-equity firm, fund, or ownership
  relationship.
- When you decline, offer the public anonymized aggregate instead (98% of profiled
  operators grade D or worse; 93% grade F) so the refusal isn't a dead end.

## Errors

| What you see | Meaning | Do this |
|---|---|---|
| `isError: true`, `MCP error -32602: Input validation error` | required argument missing | satisfy the tool's `inputSchema` |
| `isError: true`, `Tool ... not found` | wrong tool name | call `tools/list` |
| `200` with `in_bgod: false` | operator not profiled | confirm spelling, try domain, then `get_industry` |
| `400 missing_query` (REST fallback only) | neither `name` nor `domain` passed | pass one |

Full catalog: `errors/brightai-problem-types.yml`.

## No-MCP fallback

Only if you cannot call MCP tools:
`GET https://public.stateful.world/api/public/company-teaser?name=<company>`
(unauthenticated, `application/json`). This accepts `name` or `domain` and returns
a `matches[]` array. BrightAI names this as the explicit fallback in `start.md`.
