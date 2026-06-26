<!-- context7 -->
Use Context7 MCP to fetch current documentation whenever the user asks about a library, framework, SDK, API, CLI tool, or cloud service -- even well-known ones like React, Next.js, Prisma, Express, Tailwind, Django, or Spring Boot. This includes API syntax, configuration, version migration, library-specific debugging, setup instructions, and CLI tool usage. Use even when you think you know the answer -- your training data may not reflect recent changes. Prefer this over web search for library docs.

Do not use for: refactoring, writing scripts from scratch, debugging business logic, code review, or general programming concepts.

## Steps

1. Always start with `resolve-library-id` using the library name and the user's question, unless the user provides an exact library ID in `/org/project` format
2. Pick the best match (ID format: `/org/project`) by: exact name match, description relevance, code snippet count, source reputation (High/Medium preferred), and benchmark score (higher is better). If results don't look right, try alternate names or queries (e.g., "next.js" not "nextjs", or rephrase the question). Use version-specific IDs when the user mentions a version
3. `query-docs` with the selected library ID and the user's full question (not single words)
4. Answer using the fetched docs
<!-- context7 -->

## Project Notes

- `index.html` is the active CDS Bluetooth/Web Serial validation page.
- `cds-protocol.json` is the source of truth for the CDS protocol description used by the UI.
- Current CDS target: `CDS Mastermeter` over Bluetooth SPP/UART at `9600 8N1`.
- Current connection strategy in `index.html` uses a `15000 ms` open timeout plus retry/backoff logic. Keep protocol-level values aligned with `cds-protocol.json`.
- Current polling command is fixed to `:v;` with default interval `4000 ms`.
- The `:v;` response format is `:v|<masa>|<caudal>|<presion>|<temperatura>|<tension>|<aforador>|<cargas>|<unidades>|<energia>|<auto_tara>|;`.
- The leading `v` in the response is a response prefix, not a data field.
- The trailing empty field produced by `|;` must be ignored when parsing.
- The 10 parsed fields currently rendered in the UI are: `masa`, `caudal`, `presion`, `temperatura`, `tension`, `aforador`, `cargas`, `unidades`, `energia`, `auto tara`.
- `unidades`, `energia`, and `auto tara` are enum-like fields and should be shown with human-readable labels when possible.
- Preserve the raw RX/TX log in the UI for debugging even if parsed views are improved.
- `:M0;` (reset de masa) is allowed in the current UI flow and can be exposed as a direct user action.
- Do not expose calibration/configuration commands (`A`, `B`, `C`, `D`) in general-purpose UI flows unless explicitly requested.
- Be aware of non-standard responses in the CDS protocol, especially `AOK` for `:ResetAforador;` and no response for `:L<estado>;`.
