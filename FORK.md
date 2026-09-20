# Fork notes — OpenRouter backend

This fork changes **one thing**: Jev is called through OpenRouter's Decisions
API instead of TypeSafe's System One API. Everything else tracks upstream
(`tamaratran/fast-jev-compaction`).

## The change

| | upstream | this fork |
| --- | --- | --- |
| endpoint | `https://api.typesafe.ai/v1/systemone` | `https://openrouter.ai/api/alpha/decisions` |
| model id | `jev-latest` | `~typesafe/jev-latest` (leading `~` is required) |
| key env var | `TYPESAFE_API_KEY` | `OPENROUTER_API_KEY` |

The request and response bodies are unchanged: OpenRouter's Decisions endpoint
accepts the same `{model, state, questions}` body and returns the same
`{answers, usage}` shape, so `buildJevRequest` / `parseJevResponse` needed no
edits.

Touched files (kept deliberately small so upstream merges stay clean):

- `src/request.ts` — the two constants
- `src/client.ts` — env var name
- `hooks/fast-jev.ts` — env var name
- `src/messages.ts` — doc comment
- `.claude-plugin/plugin.json` — `apiKey` / `model` labels and default
- `tests/*.ts` — assertions on the above

## Pulling upstream updates

```sh
git fetch upstream
git rebase upstream/main
npm run typecheck && npm test
git push --force-with-lease origin main
```

Conflicts should be limited to the lines in the table above. If upstream adds
a `baseUrl` plugin option, drop the `src/request.ts` constant change and
configure it instead.

## Caveats

- OpenRouter's Decisions API is on an `/api/alpha/` path and is in beta; the
  path may move. If requests start 404ing, check the OpenRouter docs and update
  `SYSTEM_ONE_URL` in `src/request.ts`.
- An explicitly configured `model` is passed through untouched. A TypeSafe-style
  id (`jev-latest`) sent to OpenRouter will fail — keep the `~typesafe/` prefix.
- Jev's context window is 32k on both backends, so the default
  `maxRequestTokens` of 30000 still applies.

## Marketplace name

The marketplace in `.claude-plugin/marketplace.json` is renamed to
**`fast-jev-openrouter`** so it does not collide with upstream's
`fast-jev-compaction` marketplace. Installing upstream's id
(`fast-jev-compaction@fast-jev-compaction`) would silently pull TypeSafe code
even with this fork added. The id for this fork is:

```
fast-jev-compaction@fast-jev-openrouter
```
