---
title: "How to check if a branch is ahead or behind its remote in Git"
date: 2021-10-30
lastmod: 2024-11-22
draft: false
slug: git-how-to-check-branch-ahead-or-behind
tags: ["git"]
---

## How to

Sometimes we want to check if a branch is ahead or behind its remote in Git, here is the script.

```sh
#!/bin/bash

# Exit on any error
set -euo pipefail

# Check if the repository has any commits
if ! git rev-parse HEAD >/dev/null 2>&1; then
    echo "The repository is empty or not a Git repository."
    exit 1
fi

# Detect detached HEAD state
current_branch=$(git rev-parse --abbrev-ref HEAD)
if [[ "$current_branch" == "HEAD" ]]; then
    echo "You are in a detached HEAD state."
    exit 1
fi

# Check if the current branch has an upstream branch
if ! git rev-parse --abbrev-ref --symbolic-full-name '@{u}' >/dev/null 2>&1; then
    echo "The current branch '$current_branch' does not have an upstream tracking branch."
    exit 1
fi

# Fetch the latest updates from the remote
echo "Fetching updates from remote..."
git fetch

# Get the count of commits behind the remote
behind_count=$(git rev-list --count "HEAD..@{u}" 2>/dev/null || echo 0)
# Get the count of commits ahead of the remote
ahead_count=$(git rev-list --count "@{u}..HEAD" 2>/dev/null || echo 0)

# Print the current branch's relationship with the remote
if [[ "$behind_count" -eq 0 && "$ahead_count" -eq 0 ]]; then
    echo "The current branch '$current_branch' is up to date."
elif [[ "$behind_count" -eq 0 && "$ahead_count" -gt 0 ]]; then
    echo "The current branch '$current_branch' is ahead by $ahead_count commit(s)."
elif [[ "$behind_count" -gt 0 && "$ahead_count" -eq 0 ]]; then
    echo "The current branch '$current_branch' is behind by $behind_count commit(s)."
elif [[ "$behind_count" -gt 0 && "$ahead_count" -gt 0 ]]; then
    echo "The current branch '$current_branch' is diverged: ahead by $ahead_count commit(s), behind by $behind_count commit(s)."
else
    echo "Unexpected status. Please check your git configuration."
fi
```

## Explanation

- [`git rev-list`](https://git-scm.com/docs/git-rev-list) list all commits of giving commits range. `--count` option output how many commits would have been listed, and suppress all other output.
- `HEAD` names current branch.
- [`@{u}`](https://www.git-scm.com/docs/gitrevisions#Documentation/gitrevisions.txt-emltbranchnamegtupstreamemegemmasterupstreamememuem) refers to the local upstream of current branch (configured with `branch.<name>.remote` and `branch.<name>.merge`). There is also [`@{push}`](https://www.git-scm.com/docs/gitrevisions#Documentation/gitrevisions.txt-emltbranchnamegtpushemegemmasterpushemempushem), it is usually points to the same as `@{u}`.
- [`<rev1>..<rev2>`](https://www.git-scm.com/docs/gitrevisions#Documentation/gitrevisions.txt-emltrev1gtltrev2gtem) specifies commits range that include commits that are reachable from `<rev2>` but exclude those that are reachable from `<rev1>`. When either `<rev1>` or `<rev2>` is omitted, it defaults to HEAD.
