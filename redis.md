# Redis — DS2 test cheat sheet

Commands below run in `redis-cli`. Redis keys have **types**: string, hash, set, list, sorted set. Different types require different commands.

## Keys, search and TTL

| Command | Meaning |
|---|---|
| `PING` | Connection check (`PONG`). |
| `TYPE user:1` | Key's data type. |
| `EXISTS user:1` | Whether key exists (1/0). |
| `DEL user:1` | Delete key. |
| `SCAN 0 MATCH user:* COUNT 100` | Iterate keys matching pattern; **continue with returned cursor** until 0. |
| `KEYS user:*` | List matching keys; avoid on large/production databases. |
| `EXPIRE user:1 60` | Expire after 60 seconds. |
| `PEXPIRE user:1 18000` | Expire after 18,000 milliseconds. |
| `TTL user:1` | Seconds remaining. `-1` no expiry, `-2` missing. |
| `PTTL user:1` | Milliseconds remaining; `-1`/`-2` same meanings. |
| `PERSIST user:1` | Remove expiration. |
| `RENAME old new` | Rename key (overwrites destination). |

**Important:** `SCAN 0 MATCH ...` can return **zero results with a nonzero cursor**. Keep scanning. `SCAN` is for key names; `FT.SEARCH` is for querying a RediSearch index (Redis Stack), not a replacement for `SCAN`.

## Strings and counters

| Command | Meaning |
|---|---|
| `SET name Alice` | Store string (overwrites previous key/value and clears its TTL unless `KEEPTTL`). |
| `GET name` | Read string. |
| `MSET a 1 b 2` | Set several strings. |
| `MGET a b` | Read several strings. |
| `SET session token EX 60` | Set string with 60-second TTL. |
| `SET session token PX 5000` | Set with 5000-ms TTL. |
| `SET lock value NX EX 30` | Only set if missing, with TTL. |
| `SET key value XX` | Only set if key exists. |
| `INCR visits` | Add 1; missing key starts at 0, becomes 1. |
| `INCRBY visits 5` | Add 5. |
| `DECR visits` / `DECRBY visits 2` | Subtract. |
| `APPEND name _test` | Append text. |

`INCR` works on an integer-valued **string**, not on sets/hashes/lists. `HINCRBY` increments a hash field.

## Hashes — field/value object

| Command | Meaning |
|---|---|
| `HSET student:101 name Alice age 22 faculty FEL` | Set hash fields. |
| `HGET student:101 faculty` | Read one field. |
| `HMGET student:101 name age` | Read several fields. |
| `HGETALL student:101` | Read all fields/values. |
| `HKEYS student:101` | Field names. |
| `HVALS student:101` | Field values. |
| `HEXISTS student:101 email` | Does field exist? |
| `HDEL student:101 age` | Remove field. |
| `HLEN student:101` | **Number of fields** (not `HCOUNT`). |
| `HINCRBY student:101 score 2` | Increment numeric hash field. |

## Sets — unique, unordered members

| Command | Meaning |
|---|---|
| `SADD course:redis:students Alice Bob Charlie` | Add members (no duplicates). |
| `SMEMBERS course:redis:students` | Read all members (unordered). |
| `SISMEMBER course:redis:students Alice` | Membership test (1/0). |
| `SREM course:redis:students Bob` | Remove member. |
| `SCARD course:redis:students` | Count members. |
| `SINTER setA setB` | Members in **both** sets. |
| `SUNION setA setB` | Members in **either** set. |
| `SDIFF setA setB` | Members in A but **not** B. |
| `SINTERSTORE result setA setB` | Store intersection in `result`. |
| `SUNIONSTORE result setA setB` | Store union. |
| `SDIFFSTORE result setA setB` | Store difference (order matters). |
| `SMOVE setA setB Alice` | Move member between sets. |
| `SPOP setA` | Remove/return random member. |
| `SRANDMEMBER setA` | Return random member without removal. |

`*STORE` overwrites the destination key with the result. For empty results the destination key is deleted. TTL of a destination overwritten by `*STORE` is cleared; apply `EXPIRE result ...` again if needed.

## Lists — ordered, duplicates allowed

| Command | Meaning |
|---|---|
| `RPUSH queue 101 102 103` | Append on **right** → `[101,102,103]`. |
| `LPUSH queue 101 102 103` | Push each on **left** → `[103,102,101]`. |
| `LRANGE queue 0 -1` | All items, left to right. |
| `LRANGE queue 0 2` | First 3 items (end index inclusive). |
| `LLEN queue` | Number of items. |
| `LINDEX queue 1` | Item at index 1 (second item). |
| `LPOP queue` / `RPOP queue` | Remove/return leftmost / rightmost. |
| `LSET queue 1 999` | Replace item at index 1. |
| `LINSERT queue BEFORE 102 400` | Insert before **value** 102. |
| `LINSERT queue AFTER 102 400` | Insert after **value** 102. |
| `LREM queue 1 400` | Remove first 400 from head. |
| `LREM queue 0 400` | Remove **all** 400 values. |
| `LTRIM queue 0 4` | Keep only first 5 items. |
| `LPOS queue 102` | Index of first matching value. |

`LINSERT` uses a **pivot value**, not an index. To insert by index, lists have no direct command; reconstruct or use another approach. Negative indices: `-1` last, `-2` second last. `LRANGE` and `ZRANGE` use **inclusive** end indices.

## Sorted sets (ZSET) — unique members + numeric scores

| Command | Meaning |
|---|---|
| `ZADD leaderboard 10 Alice 20 Bob -5 Charlie` | Add members with scores (score **before** member). |
| `ZRANGE leaderboard 0 -1` | All members sorted by score ascending. |
| `ZRANGE leaderboard 0 -1 WITHSCORES` | Members plus scores. |
| `ZREVRANGE leaderboard 0 -1 WITHSCORES` | Descending by score. |
| `ZSCORE leaderboard Alice` | Score of Alice. |
| `ZRANK leaderboard Alice` | Zero-based ascending rank. |
| `ZREVRANK leaderboard Alice` | Zero-based descending rank. |
| `ZCARD leaderboard` | Number of members. |
| `ZCOUNT leaderboard -inf 10` | Count scores <=10. |
| `ZRANGE leaderboard -inf 10 BYSCORE` | Members with scores <=10 (Redis 6.2+). |
| `ZRANGE leaderboard 10 +inf BYSCORE` | Members with scores >=10. |
| `ZINCRBY leaderboard 5 Alice` | Increase Alice's score by 5. |
| `ZREM leaderboard Bob` | Remove Bob. |
| `ZPOPMIN leaderboard` / `ZPOPMAX leaderboard` | Remove lowest / highest score member. |
| `ZUNIONSTORE combined 2 zsetA zsetB` | Store union of **2** sorted sets; shared scores summed by default. |
| `ZINTERSTORE common 2 zsetA zsetB` | Store intersection; shared scores summed by default. |
| `ZUNIONSTORE combined 2 zsetA zsetB WEIGHTS 2 1 AGGREGATE MAX` | Weight scores; use max score when duplicate. |

**Crucial:** `ZRANGE key 0 -1` uses **positions/ranks**, not scores. A member with score `-5` still appears. To filter by score use `ZRANGE key -inf -1 BYSCORE` (or `ZRANGEBYSCORE key -inf -1` on older Redis). `-inf`/`+inf` are **score bounds**, not index bounds.

## Redis Stack: JSON and Search (if installed)

| Command | Meaning |
|---|---|
| `JSON.SET user:1 $ '{"name":"Alice","age":22}'` | Store JSON document. |
| `JSON.GET user:1 $` | Read JSON root. |
| `JSON.GET user:1 $.name` | Read a JSON field. |
| `JSON.NUMINCRBY user:1 $.age 1` | Increment JSON number. |
| `FT.CREATE idx:users ON JSON PREFIX 1 user: SCHEMA $.name AS name TEXT $.age AS age NUMERIC` | Create search index on JSON documents. |
| `FT.SEARCH idx:users 'Alice'` | Full-text search. |
| `FT.SEARCH idx:users '@age:[20 30]'` | Numeric range query. |
| `FT.INFO idx:users` | Inspect index. |
| `FT.DROPINDEX idx:users` | Drop index (keeps documents by default). |

`FT.SEARCH` needs an existing index; `SCAN` doesn't. JSON/FT commands require Redis Stack or appropriate modules.

## Test pitfalls

- `SET`/`GET` = string; `HSET`/`HGET` = hash; `SADD`/`SMEMBERS` = set; `LPUSH`/`LRANGE` = list; `ZADD`/`ZRANGE` = sorted set.
- `HCOUNT` is **not** the hash-size command: use `HLEN`. `SCARD` for sets, `LLEN` for lists, `ZCARD` for sorted sets.
- `LPUSH key a b c` yields `c,b,a`; `RPUSH key a b c` yields `a,b,c`.
- `LREM key count value`: positive from left, negative from right, zero = all.
- `TTL`: `-1` key exists with no expiry; `-2` key missing.
- `EXPIRE` is **seconds**, `PEXPIRE` is **milliseconds**.
- `ZADD key score member`; `ZRANGE key start stop` normally uses **ranks**.
- `ZUNIONSTORE destination numkeys key1 key2` — don't forget `numkeys`.
- Changing list/set/hash contents usually preserves TTL; overwriting a key using `SET` or `*STORE` normally clears it.

Docs: https://redis.io/docs/latest/commands/ and https://redis.io/learn/howtos/quick-start/cheat-sheet
