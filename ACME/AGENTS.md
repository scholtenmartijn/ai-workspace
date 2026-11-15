# AGENTS.md: ACME overlay

Client overlay for `~/projects/ACME/`. It adds to `~/projects/AGENTS.md`. It cannot remove a rule from
that file. If two rules are different, obey the stricter rule.

ACME is a fictional client. Copy this file for a real client and keep the copy local. Do not commit
real client data to a public repository.

## 1. Client

| Item                | Value                                 |
| ------------------- | ------------------------------------- |
| Client              | ACME                                  |
| GitHub organization | `acme`                                |
| Team                | Platform                              |
| Primary project     | `acme/acme-api`                       |

Handoff mode: review

ACME wants a human to make each commit. The agent leaves the changes uncommitted in the worktree, and
opens the pull request after the user made the commit and asked for it.

## 2. Repository registry

Keep this table current. Add a row when you onboard a repository.

| Folder              | GitHub              | Purpose                  | Stack  | Test command               |
| ------------------- | ------------------- | ------------------------ | ------ | -------------------------- |
| `ACME/acme-api`     | `acme/acme-api`     | Invoice API              | Go     | `make test`                |
| `ACME/acme-reports` | `acme/acme-reports` | Monthly invoice reports  | Python | `uv run pytest`            |

Integration test for the two repositories together: `make integration` in `ACME/acme-reports`. It starts
`acme-api` from a local checkout with `docker compose` and runs the report against it. Set
`ACME_API_PATH` to the worktree of `acme-api` to test a change that is not merged.

`TBD` in a row means: find the value during the first work in that repository, then update the row.

## 3. Client rules

- The Platform team owns the two repositories in the registry. For other ACME repositories, do not run
  install hooks.
- `acme-api` must merge before `acme-reports` when the two change together.
