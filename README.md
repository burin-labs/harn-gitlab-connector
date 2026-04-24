# harn-gitlab-connector

Pure-Harn GitLab connector for the Harn orchestrator. Verifies inbound
webhook tokens, normalizes GitLab event payloads to the canonical
tagged `NormalizeResult` envelope, and dispatches outbound REST/GraphQL
calls with optional OAuth2 helpers.

> **Status: pre-alpha** — actively developed in tandem with
> [burin-labs/harn](https://github.com/burin-labs/harn). See the
> [Pure-Harn Connectors Pivot epic #350](https://github.com/burin-labs/harn/issues/350)
> and the connector interface contract in
> [#346](https://github.com/burin-labs/harn/issues/346).

This is an **inbound + outbound** connector implementing the Harn
Connector interface. Compatible with GitLab.com (SaaS), GitLab
Self-Managed, and GitLab Dedicated (17.x+).

## Install

Via Harn package management ([harn#345](https://github.com/burin-labs/harn/issues/345)):

```sh
harn add github.com/burin-labs/harn-gitlab-connector@v0.1.0
```

Until a version is tagged, depend on this repo via a path import:

```toml
[dependencies]
harn-gitlab-connector = { path = "../harn-gitlab-connector" }
```

## Usage

### Webhook trigger (MR-opened responder)

```harn
import gitlab_connector from "harn-gitlab-connector"

trigger mr_welcome on gitlab {
  source = {
    kind: "webhook",
    events: ["merge_request"],
  }
  on event {
    if event.kind == "merge_request" && event.payload.object_attributes.action == "open" {
      gitlab_connector.call("notes.create_merge_request_note", {
        project_id: event.payload.project.id,
        merge_request_iid: event.payload.object_attributes.iid,
        body: "Thanks for the MR! The Harn bots will take a look.",
      })
    }
  }
}
```

### Outbound GraphQL

```harn
gitlab_connector.call("graphql", {
  query: "query ($path: ID!) { project(fullPath: $path) { id name } }",
  variables: {path: "my-group/my-project"},
})
```

### Commit status

```harn
gitlab_connector.call("commit_status.set", {
  project_id: 1234,
  sha: "abc123",
  state: "success",
  name: "harn-ci",
  target_url: "https://ci.example.com/run/42",
})
```

## Webhook verification

GitLab webhooks carry an optional shared-secret token in the
`X-Gitlab-Token` header. Unlike GitHub, **GitLab does not HMAC-sign the
body** — verification is a constant-time equality check between the
header value and the configured secret. When no secret is configured on
the binding, the event is accepted and emitted with
`signature_status: "unsigned"`, matching GitLab's upstream model.

If a secret is configured and the header is missing or mismatches, the
connector returns a `{type: "reject", reject: {status: 401, ...}}`
envelope.

## Authentication (outbound)

In priority order, the connector looks for:

1. `args.access_token` (OAuth2 access token)
2. `args.personal_access_token` or `args.project_access_token`
3. `args.token` (generic fallback)
4. `env("GITLAB_ACCESS_TOKEN")` / `env("GITLAB_TOKEN")`

All tokens are sent as `Authorization: Bearer <token>`, which GitLab
accepts uniformly for OAuth tokens, PATs, project access tokens, and
group access tokens.

## OAuth2 helpers

The connector exposes three `call()` methods that compose against the
existing HTTP + JSON builtins for OAuth2 code flow without requiring a
dedicated stdlib surface:

- `oauth.authorize_url` — build `/oauth/authorize` URL
- `oauth.exchange_code` — POST `/oauth/token` with `grant_type=authorization_code`
- `oauth.refresh_token` — POST `/oauth/token` with `grant_type=refresh_token`

See `SKILL.md` for a full example wiring a redirect handler.

## Rate limiting

The connector honors GitLab's `RateLimit-Remaining` and `RateLimit-Reset`
headers. When `RateLimit-Remaining` is `0`, it sleeps until `Reset`
(capped at 60 seconds; beyond that it returns a `rate_limited` error so
the caller decides).

## Development

Install the pinned Harn CLI:

```sh
cargo install harn-cli --version "$(cat .harn-version)" --locked
harn --version
```

Run the local CI equivalent:

```sh
harn check src/lib.harn
harn lint src/lib.harn
harn fmt --check src/lib.harn tests/*.harn
for test in tests/*.harn; do
  harn run "$test" || exit 1
done
```

## License

Dual-licensed under MIT and Apache-2.0.

- [LICENSE-MIT](./LICENSE-MIT)
- [LICENSE-APACHE](./LICENSE-APACHE)
