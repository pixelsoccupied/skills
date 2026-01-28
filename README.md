# Claude Code Skills

A collection of Claude Code skills by pixelsoccupied.

## Installation

```bash
/plugin marketplace add pixelsoccupied/skills
/plugin install gerrit-tools@pixelsoccupied-skills
```

Then enable via `/plugins`.

## Available Skills

### gerrit-review
Gerrit code review API operations for OpenStack's review.opendev.org.

- Fetch review comments, patches, status
- git-review CLI commands  
- Authenticated operations (abandon, vote, comment)
- Search queries and CI status

**Triggers:** "check gerrit", "review comments", "gerrit API", "git review", "opendev review"
