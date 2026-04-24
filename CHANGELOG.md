# Changelog

All notable changes to `harn-gitlab-connector` will be documented in
this file. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Added

- Initial pure-Harn GitLab connector scaffold (connector interface
  skeleton, payload schema, lifecycle functions).
- `normalize_inbound` with constant-time token verification against
  `X-Gitlab-Token` and tagged-envelope return
  (`{type: "event"|"reject", ...}`) per the harn#346 contract.
- Normalization for `push`, `tag_push`, `merge_request`, `note`,
  `issue`, and `pipeline` event types.
- Outbound `call(method, args)` dispatch covering REST notes, MR
  updates/changes/approve, commit-status setting, repository file
  fetches, GraphQL passthrough, and OAuth2 authorize/exchange/refresh
  helpers.
- `RateLimit-Remaining` / `RateLimit-Reset` handling with a 60-second
  cap on sleep-and-retry.
- Smoke tests covering happy paths, tampered payloads, missing tokens,
  and unsupported methods.

[Unreleased]: https://github.com/burin-labs/harn-gitlab-connector/compare/main
