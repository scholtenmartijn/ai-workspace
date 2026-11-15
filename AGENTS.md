# AGENTS.md

Instructions for AI agents in this workspace. This file is generic. It is the same on each machine and
for each client. Client data is in the client overlay `<client>/AGENTS.md`.

Handoff mode: pr

## 1. Scope

This folder is an umbrella workspace. It holds one folder for each client, and each client folder holds
the repositories of that client.

| Path                                   | Content                                           |
| -------------------------------------- | ------------------------------------------------- |
| `<client>/AGENTS.md`                   | Client overlay: team, registry, handoff mode.     |
| `<client>/<repo>/`                     | One GitHub repository in each folder.             |
| `<client>/.worktrees/<repo>/<branch>/` | Git worktrees for agent tasks.                    |

There are two work modes:

- Repository mode. The session starts in `<client>/<repo>/`. Work in that repository only.
- Workspace mode. The session starts in this folder. Do cross-repository work. Before you start, read
  `<client>/AGENTS.md` for the client that the task is about.

## 2. Hard rules

### Git

- Work on a task branch. Do not commit on the default branch.
- A commit message is one line in the conventional commits format (`feat:`, `fix:`, `chore:`, `docs:`).
- You can push a task branch: `git push -u origin <branch>`. Do not push to the default branch.
- Do not run a destructive git command. Examples: `git push --force`, `git reset --hard`,
  `git clean -f`, `git branch -D`.
- Do not merge a pull request. The user merges it.

### Infrastructure

- Do not run `terraform apply` or `terraform destroy`.
- Do not run `kubectl` verbs that change the cluster. Examples: `apply`, `delete`, `patch`, `scale`.
- Use read commands only for cloud accounts.

### Secrets

- Do not write tokens, keys, or passwords to files.
- Do not show tokens in the output.

### Precedence

- A repository can have its own `AGENTS.md`. Read it before you work in that repository.
- The client overlay can add rules. It cannot remove a rule from this file.
- If two rules are different, obey the stricter rule.

## 3. Before work in a repository

1. Read the client overlay `<client>/AGENTS.md`.
2. Read the repository `AGENTS.md`, if it exists.
3. Read `README.md`, the `Makefile`, and `.github/workflows/`.
4. Find the test commands in the registry of the overlay. If the value is `TBD`, find it and update
   the row.
5. Make the changes in a worktree (section 4), not in the main checkout.

## 4. Worktrees

Use one worktree for each task. The main checkout stays clean, and tasks can run in parallel. Use
worktrunk (`wt`):

```bash
cd ~/projects/<client>/<repo>
git fetch origin
wt switch --create <branch>
wt list
```

- The path is `~/projects/<client>/.worktrees/<repo>/<branch>`.
- Use absolute paths for all reads and edits. Use `git -C <path>` for git commands.
- Do not remove a worktree. The user removes it after the merge.

## 5. Cross-repository work

In workspace mode, use one worker agent for each repository. Each worker has its own context, so the
details of one repository do not fill the context of the other. Give each worker this brief:

- The repository path and the worktree path.
- The task, and the contract that the repositories share (for example the API field and its type).
- "Read `<client>/AGENTS.md` and the repository agent file first."
- "Work only in this path. Use absolute paths."
- "Write the test first. Show that it fails, then make it pass."

Each worker reports:

- The files that it changed.
- The commands that it ran, and the results.
- The handoff for the mode (section 7).

## 6. Verification

The work is not ready until a command proves it.

1. Write a test that fails for the correct reason. Run it and keep the output.
2. Make the change. Run the test again until it passes.
3. Run the full test command of the repository.
4. For a change in more than one repository, run the integration or end-to-end test that uses the
   repositories together. The registry in the overlay gives the command.
5. If a test fails and you cannot fix it, say so in the handoff. Do not hide it and do not skip the test.

## 7. Handoff

Each task ends with a pull request that the user can show. The handoff mode tells you how the task gets
there. Use the first line `Handoff mode: <mode>` that you find: the client overlay first, then this
file. If there is no line, use `review`. An instruction from the user in the session has precedence, for
that task only.

### Mode `pr`

```bash
git -C <worktree> status && git -C <worktree> diff
git -C <worktree> add <files>
git -C <worktree> commit -m "<type>: <summary>"
git -C <worktree> push -u origin <branch>
gh pr create --title "<type>: <summary>" --body "<body>"
```

### Mode `review`

1. Leave the changes uncommitted in the worktree. Do not run `git add`, `git commit`, or `git push`.
2. Give the user the worktree path and the commands to examine and commit the changes.
3. Stop. Tell the user: "Ask for the pull request after the commit."
4. When the user asks for the pull request, write the text from what is in the branch now. Then push
   the branch and run `gh pr create`.

### Both modes

- If the commit or the push fails (signing, authentication, a hook), stop and give the user the
  commands to run. Do not go around the failure.
- For a task in more than one repository, put a link to the other pull request in each pull request,
  and say which one must merge first.

Then give the user:

- The pull request link (mode `review`: after the user asked for it).
- The branch name and the worktree path.
- The test result.
- The risks and the open questions.
