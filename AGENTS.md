# AGENTS.md

Pure-Harn connector package for GitLab.com, GitLab Self-Managed, and GitLab
Dedicated.

Shared connector authoring rules live in the Harn guide:

- [Connector authoring guide](https://github.com/burin-labs/harn/blob/main/docs/src/connectors/authoring.md)

Put shared connector guidance in the Harn guide and keep only
provider-specific notes and local hazards here.

`CLAUDE.md` points here. Edit `AGENTS.md` only.

## Provider notes

- `X-Gitlab-Token` is a plain shared secret, not an HMAC signature. Compare it with constant-time
  equality.
- Outbound auth may use OAuth2 access tokens, personal access tokens, project tokens, or group
  tokens; all are sent as bearer tokens.
- Current GitLab rate-limit headers are `RateLimit-*`, and GraphQL lives at `/api/graphql` outside
  `/api/v4`.
- Do not add compatibility shims or deprecation aliases in this nascent package; cut over directly
  when behavior changes.
