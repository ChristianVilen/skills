---
name: judo-review-batch
description: This skill should be used when the user runs /judo-review-batch with several PR numbers or URLs, or asks to "judo review these PRs", "mass review these PRs" or "review all these PRs and post pending reviews". It runs judo-code-review on each PR in its own background agent and git worktree, and each agent posts a pending GitHub review.
disable-model-invocation: true
---

# Judo review batch

Run the judo-code-review skill on several pull requests at the same time. One
background agent reviews one PR in its own git worktree and posts a pending
GitHub review. Only the user can see a pending review. The user edits, deletes
or submits the comments in GitHub.

## Input

The argument is a list of PR numbers or PR URLs, for example `5143 5144 5145`.
A URL gives the repo. For a bare number, use the repo of the current directory
(`gh repo view --json nameWithOwner --jq .nameWithOwner`).

## Step 1: Look up the PRs

Get the user's login once with `gh api user --jq .login`. Then, for each PR:

```sh
gh pr view <n> --repo <repo> --json number,title,author,headRefName,baseRefName,headRefOid,isDraft,state,additions,deletions,changedFiles
gh pr view <n> --repo <repo> --json files --jq '.files | sort_by(-.additions) | .[0:4][] | "\(.additions)\t\(.deletions)\t\(.path)"'
```

## Step 2: Write the notes for each PR

| Case | How to see it | What to do |
|---|---|---|
| Stacked PR | `baseRefName` is not the default branch | Note: "This PR is stacked on `<base>` (#<n>). Review only the commits of this PR." Find the base PR with `gh pr list --repo <repo> --head <base> --state open --json number`. |
| Large fixture or generated file | One JSON file, snapshot, lockfile or generated file has most of the added lines | Note: "Do not apply the 1,000-line file rule to `<path>`. It is a fixture, not code." |
| Mechanical PR | Version bump, lockfile only, or renames only | Suggest a skip (Step 3). |
| The user's own PR | `author.login` is the user's login | Suggest a skip (Step 3). A pending review on one's own PR has no use. Suggest `/judo-code-review <n>` in a worktree instead. |
| Not open | `state` is not `OPEN` | Skip it and say so. |

## Step 3: Start, or ask first

The slash command is the go. Start the agents at once (Step 4), unless the
batch has a mechanical PR or a PR by the user. In that case, show which PRs
will run and which you suggest to skip, and wait for the answer.

## Step 4: Start one agent per PR

Read `references/agent-prompt.md` and fill in the values for each PR. Start all
agents in one message, so that they run at the same time:

- `subagent_type`: `general-purpose`
- `isolation`: `worktree`
- `description`: `Judo review PR <n>`
- Do not set `model` or `effort`.

The reports go to the session scratchpad directory that the system prompt
names. Tell the user in one short message which PRs are running.

## Step 5: While the agents run

When an agent finishes, give a one-line update: how many are done and which
are still running. Do not read the agent output files.

If an agent reports something that the user must see before they submit, say
it in that update. Examples: the author deleted the user's tests, or the agent
did not post a review.

## Step 6: Check the reviews

When all agents finish, check each review against the PR head:

```sh
gh api repos/<repo>/pulls/<n>/reviews/<review-id> --jq '"\(.state) \(.user.login) \(.commit_id[0:9])"'
gh api repos/<repo>/pulls/<n> --jq '.head.sha[0:9]'
```

- The state must be `PENDING`, and the login must be the user's.
- If the review commit is not the PR head, the author pushed after the review.
  Run `git fetch origin pull/<n>/head`, then `git range-diff <review-commit>...<head>`.
  If the PR's own commits show `=`, the change is the same. Tell the user to
  check that the comments still show on the correct lines. If they show `!`,
  tell the user that the review is on an older version of the PR.

## Step 7: Remove the worktrees

Each agent leaves a worktree at `.claude/worktrees/agent-<id>` and a branch
`worktree-agent-<id>`. The task notification gives both. Make sure that each
worktree has no changes (`git -C <path> status --porcelain`), then remove it:

```sh
git worktree remove <path>
git branch -d worktree-agent-<id>
```

Never use `--force` or `-D`. If a command fails, keep that worktree and tell
the user.

## Step 8: Summary

Show one table with one row for each PR:

| PR | Author | Verdict | 🔴 / 🟠 / 🟡 | Comments | Main point |

Link each PR number to its pending review. Under the table, list what the user
must check before they submit: heads that moved, surprises from the agent
reports, and PRs that have no review. End with the path of the report folder.

## Files

- `references/agent-prompt.md`: the prompt for each agent, with the values to fill in.
