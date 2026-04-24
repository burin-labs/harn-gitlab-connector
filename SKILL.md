# SKILL: harn-gitlab-connector

Trigger recipes and outbound helpers for GitLab via the pure-Harn
`harn-gitlab-connector` package. Drop-in GitLab analog to
`harn-github-connector`.

## What you get

- **Webhook inbound** with constant-time token verification.
- **Normalized events** for `push`, `tag_push`, `merge_request`,
  `note`, `issue`, and `pipeline`.
- **REST outbound** for MR comments, MR description updates, MR file
  diffs, commit status, repository file fetches, and MR approvals.
- **GraphQL passthrough** for anything not covered by REST.
- **OAuth2 helpers** to build authorize URLs and exchange/refresh
  tokens without a dedicated stdlib surface.

## Trigger recipe — welcome comment on new merge requests

```harn
import gitlab_connector from "harn-gitlab-connector"

trigger mr_welcome on gitlab {
  source = {
    kind: "webhook",
    events: ["merge_request"],
  }
  on event {
    let attrs = event.payload.object_attributes
    if event.kind == "merge_request" && attrs.action == "open" {
      gitlab_connector.call("notes.create_merge_request_note", {
        project_id: event.payload.project.id,
        merge_request_iid: attrs.iid,
        body: "Thanks for the MR, @" + event.payload.user.username + "!",
      })
    }
  }
}
```

## Trigger recipe — auto-comment on failed pipelines

```harn
trigger pipeline_failed on gitlab {
  source = {
    kind: "webhook",
    events: ["pipeline"],
  }
  on event {
    let pipeline = event.payload.object_attributes
    if pipeline.status == "failed" {
      gitlab_connector.call("commit_status.set", {
        project_id: event.payload.project.id,
        sha: pipeline.sha,
        state: "failed",
        name: "harn-watchdog",
        description: "Pipeline #" + to_string(pipeline.id) + " failed",
        target_url: event.payload.project.web_url + "/-/pipelines/" + to_string(pipeline.id),
      })
    }
  }
}
```

## OAuth2 flow

```harn
// 1. Redirect the user to GitLab to authorize.
let url = gitlab_connector.call("oauth.authorize_url", {
  client_id: env("GITLAB_OAUTH_CLIENT_ID"),
  redirect_uri: "https://orchestrator.example.com/oauth/callback",
  scope: "api read_user",
  state: generated_nonce,
})
// Serve a 302 to `url.authorize_url`.

// 2. In the callback, exchange the code for tokens.
let tokens = gitlab_connector.call("oauth.exchange_code", {
  client_id: env("GITLAB_OAUTH_CLIENT_ID"),
  client_secret: env("GITLAB_OAUTH_CLIENT_SECRET"),
  code: req.query.code,
  redirect_uri: "https://orchestrator.example.com/oauth/callback",
})
// Store tokens.access_token, tokens.refresh_token, tokens.expires_in.

// 3. Refresh near expiry.
let refreshed = gitlab_connector.call("oauth.refresh_token", {
  client_id: env("GITLAB_OAUTH_CLIENT_ID"),
  client_secret: env("GITLAB_OAUTH_CLIENT_SECRET"),
  refresh_token: stored_refresh_token,
})
```

## Required secrets per binding

| Secret                         | Used for                                        |
| ------------------------------ | ----------------------------------------------- |
| `signing_secret` (optional)    | Matches `X-Gitlab-Token` on inbound webhooks    |
| `access_token` / `token`       | Outbound REST + GraphQL as `Authorization: Bearer` |
| `client_id` / `client_secret`  | OAuth2 helpers                                  |

When `signing_secret` is omitted, inbound events are still accepted but
emit with `signature_status: "unsigned"`. Configure a secret in the
webhook UI (Settings → Webhooks → Secret token) for signed verification.

## Self-hosted GitLab

Set `api_base_url` on each call to target self-hosted instances:

```harn
gitlab_connector.call("notes.create_merge_request_note", {
  api_base_url: "https://gitlab.example.com/api/v4",
  // ... rest of args
})
```

Or set `GITLAB_API_BASE_URL` as an environment variable.
