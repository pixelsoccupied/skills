# Gerrit API Endpoints Reference

Base: `https://review.opendev.org`

All responses prefixed with `)]}'\n` - strip with `tail -n +2`.

## Changes

| Operation | Method | Endpoint |
|-----------|--------|----------|
| Get change | GET | `/changes/{change-id}` |
| Get change detail | GET | `/changes/{change-id}/detail` |
| Get comments | GET | `/changes/{change-id}/comments` |
| Get messages | GET | `/changes/{change-id}/messages` |
| Abandon | POST | `/changes/{change-id}/abandon` |
| Restore | POST | `/changes/{change-id}/restore` |
| Submit | POST | `/changes/{change-id}/submit` |
| Rebase | POST | `/changes/{change-id}/rebase` |
| Set topic | PUT | `/changes/{change-id}/topic` |

## Revisions (Patchsets)

| Operation | Method | Endpoint |
|-----------|--------|----------|
| Get patch | GET | `/changes/{id}/revisions/{rev}/patch` |
| Get diff | GET | `/changes/{id}/revisions/{rev}/files/{file}/diff` |
| List files | GET | `/changes/{id}/revisions/{rev}/files` |
| Check mergeable | GET | `/changes/{id}/revisions/{rev}/mergeable` |
| Cherry-pick | POST | `/changes/{id}/revisions/{rev}/cherrypick` |

Use `current` for latest revision: `/changes/123/revisions/current/patch`

## Reviews

| Operation | Method | Endpoint |
|-----------|--------|----------|
| Set review | POST | `/changes/{id}/revisions/{rev}/review` |
| List reviewers | GET | `/changes/{id}/reviewers` |
| Add reviewer | POST | `/changes/{id}/reviewers` |
| Delete reviewer | DELETE | `/changes/{id}/reviewers/{account}` |

### Set Review Payload
```json
{
  "message": "Looks good!",
  "labels": {
    "Code-Review": 2,
    "Workflow": 1
  }
}
```

## Search

```bash
# Search changes
curl -s "https://review.opendev.org/changes/?q=QUERY" | tail -n +2 | jq '.'
```

### Common Queries
| Query | Description |
|-------|-------------|
| `owner:self` | My changes |
| `reviewer:self` | Changes I'm reviewing |
| `project:openstack/ironic` | Changes in project |
| `status:open` | Open changes |
| `is:mergeable` | Mergeable changes |
| `label:Verified+1` | Verified changes |
| `topic:bug-fix` | By topic |

Combine: `project:openstack/ironic+status:open+owner:self`

## Authenticated Operations

Require HTTP credentials in `~/.netrc`.

### Abandon a Change
```bash
curl -n -X POST -H "Content-Type: application/json" \
  -d '{"message":"Merged into other change"}' \
  "https://review.opendev.org/a/changes/CHANGE_ID/abandon"
```

### Add Comment
```bash
curl -n -X POST -H "Content-Type: application/json" \
  -d '{"message":"LGTM"}' \
  "https://review.opendev.org/a/changes/CHANGE_ID/revisions/current/review"
```

### Vote
```bash
curl -n -X POST -H "Content-Type: application/json" \
  -d '{"labels":{"Code-Review":"+1"}}' \
  "https://review.opendev.org/a/changes/CHANGE_ID/revisions/current/review"
```

Note: Authenticated endpoints use `/a/` prefix.

## CI Status (Zuul)

Zuul votes appear in labels. Check via:
```bash
curl -s "https://review.opendev.org/changes/CHANGE_ID/detail" | tail -n +2 | \
  jq '.labels.Verified'
```

For detailed Zuul status, check buildsets:
```
https://zuul.opendev.org/t/openstack/buildset/{buildset-id}
```