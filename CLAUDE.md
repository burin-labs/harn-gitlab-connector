# CLAUDE.md — harn-gitlab-connector

## Quick repo conventions

- File extension: `.harn`. Use `snake_case` for filenames.
- Repo directories use `kebab-case`.
- Entry point: `src/lib.harn`.
- Tests live under `tests/`. Recorded webhook fixtures live under
  `tests/fixtures/webhooks/`.

## How to test

Install the pinned Harn CLI:

```sh
cargo install harn-cli --version "$(cat .harn-version)" --locked
harn --version
```

Run checks from the repo root:

```sh
harn check src/lib.harn
harn lint src/lib.harn
harn fmt --check src/lib.harn tests/*.harn
for test in tests/*.harn; do
  harn run "$test" || exit 1
done
```

## GitLab-specific gotchas

- **`X-Gitlab-Token` is NOT HMAC.** GitLab echoes back the configured
  secret verbatim; verify with `constant_time_eq`, not `hmac_sha256`.
- **No "installation token" concept.** Outbound auth is OAuth2 access
  tokens, personal access tokens (PATs), project/group access tokens,
  or the upcoming CI job token — all sent as `Authorization: Bearer`.
- **Rate-limit headers are `RateLimit-*`** (no `X-` prefix in current
  GitLab), distinct from GitHub's `x-ratelimit-*`.
- **GraphQL endpoint is `/api/graphql`** (not versioned under `/api/v4`).

## Upstream conventions

For general Harn coding conventions and project layout, defer to
[`/Users/ksinder/projects/harn/CLAUDE.md`](/Users/ksinder/projects/harn/CLAUDE.md).

## Don't

- Don't bake an OpenAPI-codegen GitLab SDK into this repo. If you need a
  typed surface, propose `gitlab-sdk-harn` as a separate repo first.
- Don't hand-edit `LICENSE-*` or `.gitignore`.
- Don't introduce compat shims or deprecation aliases — delete cleanly.
