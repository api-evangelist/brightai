---
name: Size an essential-services vertical and its observability problems
description: Walk BrightAI's Global Observability Database from the worldwide rollup
  down to a single named observability problem family, using the public MCP tools
  and the unauthenticated JSON endpoint.
api: mcp/brightai-mcp.yml
surface: https://public.stateful.world/mcp
operations:
- list_industries
- get_industry
- list_problem_families
- get_problem_family
generated: '2026-08-08'
method: generated
source: tool names verified against a live tools/list on 2026-08-08 (mcp/brightai-mcp-tools.json)
---

# Size a vertical and drill into its observability problems

## Connect

`https://public.stateful.world/mcp` — public, no authentication, MCP `2025-06-18`,
streamable HTTP. POST JSON-RPC with `Accept: application/json, text/event-stream`.

## Steps

1. **List the verticals.** Call `list_industries` (no arguments). You get every
   mapped vertical with `slug`, `name`, `aai_usd_per_year`, `operators_profiled`
   and `problem_families`.

   **Read the count from the response.** BrightAI's own tool description says the
   catalog size is "whatever the live catalog holds — read it from the response
   rather than assuming a fixed number." Do not hard-code 15 verticals or 160
   problem families.

2. **Drill into one vertical.** Call `get_industry` with `industry` — either the
   slug (`oil-and-gas`, `water-and-wastewater`) or a close name (`Oil & Gas`);
   it fuzzy-matches. Required parameter, minimum length 2. You additionally get
   `description`, `average_addressable_impact_per_operator_usd`, `impact_framing`
   and a ranked `top_problem_families[]`.

   Note `top_problem_families[]` is a **partial projection** — the row's
   `problem_families` count is larger than the returned array.

3. **Browse the taxonomy.** Call `list_problem_families`. All three arguments are
   optional: `industry` (vertical slug/name filter), `search` (substring on family
   name), `limit` (integer 1–200, default 40). This is the only paginated call on
   the surface, and it is limit-only — there is no cursor, offset or next-page
   token.

4. **Get the full definition.** Call `get_problem_family` with `family_id` taken
   verbatim from step 2 or 3. Returns the formal definition, the anatomy
   (manifestations / measurement / intervention), typical assets, and which
   verticals it applies to.

   **Never construct a `family_id`.** The catalog mixes kebab-case and snake_case
   in the same namespace — `storage-tank-floor-shell-and-roof` sits alongside
   `locate_811_ticket_intelligence` and
   `factory_production_quality_and_first_pass_yield`. Treat the id as opaque and
   always carry it forward from a list call.

5. **Attribute.** Figures are BrightAI's benchmark opinion derived from public
   sources, not verified fact. Methodology:
   https://public.stateful.world/methodology. Corrections:
   https://public.stateful.world/data-disputes.

## Conventions that matter here

- Responses carry an `as_of` ISO-8601 timestamp; aggregates are recomputed
  continuously, so two calls minutes apart can differ. Quote the `as_of` alongside
  any figure you report.
- `get_industry` returning `{"error":"unknown_industry","hint":"Call list_industries
  for the valid slugs."}` arrives as a **200 with `isError`/`structuredContent`**,
  not an HTTP error. Read the body.
- Every tool is `readOnlyHint: true`, `destructiveHint: false`, `openWorldHint:
  false` and `execution.taskSupport: forbidden` — safe to retry, never long-running,
  and answers come from BrightAI's closed database rather than the open web.

## No-MCP fallback

`GET https://public.stateful.world/api/public/industries` — unauthenticated JSON
covering steps 1 and part of 2. It is the superset for aggregates: it adds a
`worldwide{}` rollup (`total_aai_usd_per_year`, `operators_profiled`,
`problem_families`, `priced_problem_instances`, `verticals`). There is **no** REST
equivalent for the problem-family taxonomy — steps 3 and 4 are MCP-only. See
`mcp/brightai-tool-crosswalk.yml`.
