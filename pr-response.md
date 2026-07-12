# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:** Rename `save_to_watchlist()` to `add_to_watchlist()`
**How I verified:** Searched through codebase for calls to `save_to_watchlist()`, and replaced all calls with `add_to_watchlist()`.

## Comment 2 — Deduplication
**What I did:** Added a duplicate check to `add_to_watchlist()` in `services/watchlist_service.py`, following the same pattern used in `add_to_collection()` (services/collection_service.py). Before creating a new `WatchlistEntry`, the function now queries for an existing entry with `WatchlistEntry.query.filter_by(user_id=user_id, film_id=film_id).first()`. If a match is found, it raises `AlreadyInWatchlistError` before any insert happens — mirroring the query-then-raise-then-create order used for collections.
**How I verified:** Ran `pytest tests/ -v` to confirm the existing suite still passed, then manually tested by POSTing the same `(user_id, film_id)` pair to the watchlist add endpoint twice via curl — the first call succeeded and created the entry, the second call returned `AlreadyInWatchlistError` instead of creating a duplicate row.


## Comment 3 — Missing test
**What I did:**
**How I verified:**

## Comment 4 — Default visibility
**My position:**
**Reasoning:**
**Tradeoff acknowledged:**

## Comment 5 — Sort order
**My position:**
**Reasoning:**
**Engagement with reviewer's point:**

## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->