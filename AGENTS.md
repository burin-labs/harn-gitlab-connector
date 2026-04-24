# AGENTS.md — harn-gitlab-connector

## Quick repo conventions

- File extension: `.harn`. Use `snake_case` for filenames.
- Repo directories use `kebab-case`.
- Entry point: `src/lib.harn`.
- Tests live under `tests/`. Recorded webhook fixtures live under
  `tests/fixtures/webhooks/`.

## How to test

```sh
cargo install harn-cli --version "$(cat .harn-version)" --locked
harn check src/lib.harn
harn lint src/lib.harn
harn fmt --check src/lib.harn tests/*.harn
for test in tests/*.harn; do
  harn run "$test" || exit 1
done
```

## GitLab-specific gotchas

- `X-Gitlab-Token` is NOT HMAC — it's a plain shared secret. Use
  `constant_time_eq`, not `hmac_sha256`.
- No installation-token concept — OAuth2 / PAT / PRJ / GRP tokens all
  flow through `Authorization: Bearer`.
- Rate-limit headers are `RateLimit-*` (no `X-` prefix).
- GraphQL endpoint: `/api/graphql` (outside `/api/v4`).

## Upstream conventions

For general Harn conventions, see
[`/Users/ksinder/projects/harn/AGENTS.md`](/Users/ksinder/projects/harn/AGENTS.md).

## Don't

- Don't build an OpenAPI-codegen GitLab SDK here. Propose
  `gitlab-sdk-harn` as a separate repo first.
- Don't hand-edit `LICENSE-*` or `.gitignore`.
- Don't add compat shims or deprecation aliases.
