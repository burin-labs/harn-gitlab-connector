# Changelog

All notable changes to `harn-gitlab-connector` will be documented in
this file. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Added

- `credential_environment` binds `gitlab/access-token` to the
  `GITLAB_ACCESS_TOKEN` environment variable, so a host that resolves
  connector credentials from the process environment can find the
  outbound GitLab token instead of reporting it missing.
- Connector contract v2 with product-facing service metadata: the
  manifest now declares the eight outbound GitLab operations
  (`merge_requests.list_changes`, `repository_files.get`,
  `notes.create_merge_request_note`, `notes.create_issue_note`,
  `merge_requests.update`, `merge_requests.approve`,
  `commit_status.set`, `graphql`) with their capability, purpose,
  effect, evidence, and redaction semantics.
- `methods()` export describing the outbound product methods and which
  ones mutate GitLab state. OAuth methods stay out of the inventory
  because they mint and rotate credentials and are reached through
  `harn connect`.

## [0.1.0]

### Added

- Merge-conflict surfacing on `merge_request` events: the normalized
  resource now carries `detailed_merge_status`, `is_conflict`, and a
  derived `merge_state`. A conflict is signalled when
  `detailed_merge_status == "conflict"` or the legacy
  `merge_status == "cannot_be_merged"`.
- `@`-mention command extraction on `note` events (MR and issue notes).
  The normalized payload gains a `mention` block
  (`{actor, candidates: [{handle, command, rest}], target_kind,
  target_id, url}`) parsed from the note body with CPU-only string
  scanning. Notes without a leading `@handle command` mention omit the
  block.

### Changed (initial scaffold, pre-0.1.0)

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

[Unreleased]: https://github.com/burin-labs/harn-gitlab-connector/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/burin-labs/harn-gitlab-connector/releases/tag/v0.1.0
