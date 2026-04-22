# OpenClaw — Core Memory Foundation

SQLite-backed memory module for the OpenClaw AI coding assistant.

## Goals

- Persist conversation messages with rich metadata (role, content, **topic**, timestamps).
- Support disk persistence via `saveToDisk` / `loadFromDisk`.
- Organize memory hierarchically across three tiers:
  - **Short-term** — last N raw messages, cheapest to query.
  - **Mid-term** — compact summaries produced when short-term fills up.
  - **Long-term** — persistent, durable storage for summaries and pinned facts.
- Auto-prune: when short-term exceeds the configured threshold, oldest messages
  are summarized into mid-term and discarded from short-term.

## Architecture

```
+-------------------+      spill over threshold       +-------------------+
|   Short-term      |  ----------------------------> |    Mid-term       |
|   (raw messages)  |          summarize              |   (summaries)     |
+-------------------+                                 +---------+---------+
                                                                |
                                                                v
                                                      +-------------------+
                                                      |    Long-term      |
                                                      |  (persistent DB)  |
                                                      +-------------------+
```

All three tiers live in the same SQLite database; the tier is a column so we
can query across tiers cheaply and promote/demote rows without copying data to
separate files.

## Schema

`messages` table:

| column       | type      | notes                                      |
| ------------ | --------- | ------------------------------------------ |
| id           | INTEGER   | primary key, autoincrement                 |
| role         | TEXT      | `user` / `assistant` / `system` / `tool`   |
| content      | TEXT      | raw message body                           |
| topic        | TEXT      | topic/tag label (new in this step)         |
| tier         | TEXT      | `short` / `mid` / `long`                   |
| summary_of   | TEXT JSON | ids of source messages if this is a summary |
| created_at   | INTEGER   | unix ms                                    |

## Plan

- [ ] **Step 1** — Add `topic` field to the message storage schema.
- [ ] **Step 2** — Add `saveToDisk` and `loadFromDisk` methods.
- [ ] **Step 3** — Implement hierarchical tiers (short / mid / long).
- [ ] **Step 4** — Auto-pruning: summarize + demote when threshold exceeded.

## Done

(Updated as each step lands.)
