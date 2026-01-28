---
name: gerrit-review
description: |
  Gerrit code review API operations for OpenStack's review.opendev.org.
  Use when working with Gerrit reviews: fetching comments, getting patches,
  checking review status, abandoning changes, or using git-review CLI.
  Triggers: "check gerrit", "review comments", "gerrit API", "git review",
  "opendev review", "abandon PR", "gerrit patch".
---

# Gerrit Review Operations

## Base URL
```
https://review.opendev.org
```

## Quick Reference

### Fetch Review Details (unauthenticated)
```bash
# Get change details with messages and labels
curl -s "https://review.opendev.org/changes/CHANGE_ID/detail" | tail -n +2 | jq '.'

# Get all comments on a change
curl -s "https://review.opendev.org/changes/CHANGE_ID/comments" | tail -n +2 | jq '.'

# Get current patch as diff
curl -s "https://review.opendev.org/changes/CHANGE_ID/revisions/current/patch" | base64 -d
```

Note: Gerrit API responses have a XSSI prevention prefix `)]}'\n` - use `tail -n +2` to strip it.

### Common Patterns
```bash
# List comments with author and message
curl -s "https://review.opendev.org/changes/CHANGE_ID/comments" | tail -n +2 | \
  jq -r 'to_entries[] | .key as $file | .value[] | "\($file): \(.author.name): \(.message)"'

# Get review votes/labels
curl -s "https://review.opendev.org/changes/CHANGE_ID/detail" | tail -n +2 | \
  jq '.labels'

# Check if change is mergeable
curl -s "https://review.opendev.org/changes/CHANGE_ID/revisions/current/mergeable" | tail -n +2 | jq '.'
```

## Authentication

For write operations (commenting, voting, abandoning), use HTTP credentials:

1. Generate HTTP password at: https://review.opendev.org/settings/#HTTPCredentials
2. Store in `~/.netrc`:
   ```
   machine review.opendev.org login YOUR_USERNAME password YOUR_HTTP_PASSWORD
   ```
3. Use `-n` flag with curl: `curl -n ...`

## git-review CLI

### Setup
```bash
pip install git-review
git review -s  # Setup repo for gerrit
```

### Common Commands
```bash
git review              # Submit current branch for review
git review -d CHANGE_ID # Download a change to local branch
git review -x CHANGE_ID # Cherry-pick a change
git review -R           # Submit without rebase
```

## API Endpoints Reference

See [references/api_endpoints.md](references/api_endpoints.md) for complete endpoint documentation.