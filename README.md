# Claude Code Skills

A collection of Claude Code skills by pixelsoccupied.

## Installation

```bash
/plugin marketplace add pixelsoccupied/skills
/plugin install gerrit-tools@pixelsoccupied-skills
```

Restart Claude Code to activate.

## Available Skills

### gerrit-review
Gerrit code review API operations for OpenStack's review.opendev.org.

- Fetch review comments, patches, status
- git-review CLI commands  
- Authenticated operations (abandon, vote, comment)
- Search queries and CI status

**Triggers:** "check gerrit", "review comments", "gerrit API", "git review", "opendev review"

### postgres-dont-do-this
PostgreSQL anti-patterns and common mistakes to avoid when writing SQL, designing schemas, or configuring Postgres.

- Data type pitfalls (timestamp vs timestamptz, char vs text, serial vs identity)
- SQL construct traps (NOT IN with NULLs, BETWEEN with timestamps)
- Schema design rules (encoding, naming, authentication)
- Quick decision table for common substitutions

**Triggers:** writing SQL, creating tables, schema design, Postgres data types, timestamp handling
