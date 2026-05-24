---
name: postgres-dont-do-this
description: >
  PostgreSQL anti-patterns and common mistakes to avoid. Use when writing SQL queries,
  creating or altering tables, designing database schemas, choosing column data types,
  handling timestamps, configuring pg_hba.conf, or reviewing Postgres code.
when_to_use: |
  - User writes CREATE TABLE, ALTER TABLE, or any DDL statement
  - User chooses data types for columns (timestamp, char, varchar, serial, money)
  - User writes NOT IN with a subquery
  - User uses BETWEEN with timestamp columns
  - User configures PostgreSQL authentication or pg_hba.conf
  - User asks about text storage types, date/time types, or encoding
  - User writes or reviews SQL migrations
  - Do NOT use for: query performance tuning, index strategy, or replication setup
---

# PostgreSQL: Don't Do This

Apply these rules when writing SQL queries, designing schemas, choosing data types,
or configuring PostgreSQL.

**Source:** https://wiki.postgresql.org/wiki/Don't_Do_This

---

## Database Encoding

### Don't use SQL_ASCII

SQL_ASCII means "no conversions" — the database stores raw bytes without encoding
awareness. You'll end up with a mixture of encodings and no way to recover original
characters. Use **UTF8** instead.

---

## Tool Usage

### Don't use `psql -W` or `--password`

Forces a password prompt before connecting, even if the server doesn't require one.
Misleads you into thinking password auth works when peer auth is actually in use.
Never needed — psql prompts automatically when the server requires a password.

### Don't use rules

Rules rewrite queries rather than applying conditional logic. All non-trivial rules
are incorrect. Use **triggers** instead. The only valid use of the rewriter is as an
internal implementation detail of VIEWs.

### Don't use table inheritance

A remnant of object-relational coupling that never delivered. Use **foreign keys** for
relationships. Use **native table partitioning** (PG 10+) instead of inheritance-based
partitioning.

---

## SQL Constructs

### Don't use NOT IN

**Critical.** `NOT IN` breaks silently with NULLs:

```sql
-- BROKEN: returns 0 rows if ANY value in subquery is NULL
SELECT * FROM foo WHERE col NOT IN (SELECT x FROM bar);

-- CORRECT: use NOT EXISTS instead
SELECT * FROM foo WHERE NOT EXISTS (SELECT 1 FROM bar WHERE foo.col = bar.x);
```

`col NOT IN (1, NULL)` can never return TRUE because `col IN (1, NULL)` returns
TRUE or NULL (never FALSE), and `NOT NULL` is still NULL.

Performance: `NOT IN (SELECT ...)` can't be optimized to an anti-join. It degrades
from O(N) to O(N²) past a size threshold.

**Safe exception:** `NOT IN (literal, list, ...)` with no NULLs possible.

### Don't use upper case table or column names

PostgreSQL folds all unquoted identifiers to lower case. `CREATE TABLE Foo` creates
table `foo`. `CREATE TABLE "Bar"` creates table `Bar` — requiring double quotes forever.

Use only `a-z`, `0-9`, and `_` in identifiers. Use column aliases for display names:

```sql
SELECT character_name AS "Character Name" FROM characters;
```

### Don't use BETWEEN with timestamps

BETWEEN is a closed interval (includes both endpoints). This double-counts midnight:

```sql
-- BROKEN: includes 2018-06-08 00:00:00.000000 but not later that day
SELECT * FROM t WHERE ts BETWEEN '2018-06-01' AND '2018-06-08';

-- CORRECT: half-open interval
SELECT * FROM t WHERE ts >= '2018-06-01' AND ts < '2018-06-08';
```

BETWEEN is safe for discrete types (integers, dates) but still a bad habit.

---

## Date/Time Storage

### Don't use `timestamp` (without time zone)

Use **`timestamptz`** (timestamp with time zone). Despite the name, `timestamptz`
stores a point in time (microseconds since 2000-01-01 UTC), not a timestamp + zone.
It handles DST transitions and cross-timezone arithmetic correctly.

`timestamp` (without tz) is just a picture of a clock — without knowing the timezone,
you can't determine what moment it represents.

### Don't use `timestamp` to store UTC times

Even if you "know" all values are UTC, the database doesn't. Timezone-aware calculations
become needlessly complex:

```sql
-- With timestamp (painful)
date_trunc('day', x.datecol AT TIME ZONE 'UTC' AT TIME ZONE u.timezone)
  AT TIME ZONE u.timezone AT TIME ZONE 'UTC'

-- With timestamptz (simple)
date_trunc('day', x.datecol AT TIME ZONE u.timezone)
```

### Don't use `timetz`

The SQL standard itself calls this type of questionable usefulness. Use `timestamptz`
instead. Never.

### Don't use CURRENT_TIME

Returns `timetz` (see above). Use instead:

| Need | Function |
|------|----------|
| Timestamp with tz | `CURRENT_TIMESTAMP` or `now()` |
| Timestamp without tz | `LOCALTIMESTAMP` |
| Date | `CURRENT_DATE` |
| Time | `LOCALTIME` |

### Don't use `timestamp(0)` or `timestamptz(0)`

Precision specification **rounds** fractional seconds instead of truncating. Storing
`now()` can produce a value half a second in the future.

```sql
-- WRONG
SELECT now()::timestamptz(0);

-- CORRECT
SELECT date_trunc('second', now());
```

### Don't use `+/-HH:mm` as timezone name

PostgreSQL interprets POSIX timezone offsets with inverted signs: `+04` means 4 hours
**west** (opposite of ISO convention). Use named timezones or interval syntax:

```sql
-- CORRECT: interval follows ISO convention
SELECT ts AT TIME ZONE INTERVAL '04:00';
```

---

## Text Storage

### Don't use `char(n)`

Pads values with spaces to width `n`. Wastes storage, no performance benefit over
`text`, causes surprising comparison behavior with trailing spaces. Use **`text`**.

### Don't use `char(n)` for fixed-length identifiers

`char(n)` doesn't reject short values — it silently pads with spaces. Use a domain
or check constraint instead:

```sql
-- CORRECT: validates both length and format
ALTER TABLE t ADD CONSTRAINT chk_code CHECK (length(code) = 3 AND code ~ '^[A-Z]{3}$');
```

Bonus: `char(n)` vs `text`/`varchar` type mismatches can prevent index usage.

### Don't use `varchar(n)` by default

`varchar(n)`, `varchar`, and `text` use identical storage. The length limit only adds
a risk of production errors when real-world data exceeds your arbitrary limit.

Use **`text`** by default. Add a **`CHECK` constraint** if you need validation — it
can enforce min length, max length, and format in one place.

Use `varchar(n)` only when you specifically want a hard length limit and don't need
other validation.

---

## Other Data Types

### Don't use `money`

Fixed-point, no fractional cents, no currency storage. Changing `lc_monetary` silently
reinterprets all values. Use **`numeric`** with a currency column instead.

### Don't use `serial`

Use **identity columns** (PG 10+):

```sql
-- WRONG (legacy)
CREATE TABLE t (id serial PRIMARY KEY);

-- CORRECT
CREATE TABLE t (id integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY);
```

`serial` creates implicit sequences with awkward ownership and permission semantics.

---

## Authentication

### Don't use `trust` over TCP/IP

This line allows anyone on the Internet to log in as any user including superuser:

```
-- NEVER DO THIS
host all all 0.0.0.0/0 trust
```

Use **`scram-sha-256`** (PG 10+) for password auth. Use **peer** authentication for
local Unix socket connections. `trust` is acceptable only for localhost in development
or isolated CI environments.

---

## Quick Decision Table

| Tempted to use | Use instead |
|----------------|-------------|
| `SQL_ASCII` | `UTF8` |
| `psql -W` | (just omit it) |
| Rules | Triggers |
| Table inheritance | Foreign keys / native partitioning |
| `NOT IN (SELECT)` | `NOT EXISTS (SELECT)` |
| `BETWEEN` with timestamps | `>= AND <` |
| `timestamp` | `timestamptz` |
| `timetz` | `timestamptz` |
| `CURRENT_TIME` | `now()` / `CURRENT_TIMESTAMP` |
| `timestamp(0)` | `date_trunc('second', ...)` |
| `char(n)` | `text` |
| `varchar(n)` | `text` + `CHECK` constraint |
| `money` | `numeric` |
| `serial` | `GENERATED ALWAYS AS IDENTITY` |
| `trust` over TCP | `scram-sha-256` |
