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
**What I did:** Created `tests/test_watchlist.py` and added `test_add_to_watchlist_nonexistent_film_raises`, modeled directly on `test_add_to_collection_nonexistent_film_raises` in `tests/test_collection.py`. The test asserts that calling `add_to_watchlist()` with a nonexistent `film_id` raises `FilmNotFoundError` (imported from `services.collection_service`, the same shared error class `watchlist_service.py` already uses for that check) rather than surfacing a raw database integrity error.

**How I verified:** First hit a `fixture 'app' not found` error, which surfaced that fixtures weren't shared across test files. After adding local `app` and `sample_user` fixtures to `test_watchlist.py`, ran `pytest tests/test_watchlist.py -v` to confirm the new test passes, then ran `pytest tests/ -v` to confirm `test_collection.py` and the full suite still pass with no regressions.


## Comment 4 — Default visibility
**My position:** The default visibility for user lists should be set to public.
**Reasoning:** Because this app is a community film tracking/social app, the ability to socialize and share should be the default. Because many users usually don't touch default settings, if we set watchlist visibilty to private by default, most users wouldn't share their watchlists, which is the point of the app.
**Tradeoff acknowledged:** There may be many users who don't think about privacy settings, and may end up broadcasting/sharing their watchlist by default, which could include movies they don't want others to know they watch.

## Comment 5 — Sort order
**My position:** I agree, watchlists should default to "date added" order.
**Reasoning:** The order that a film was added is more important than its alphabetical positioning, because the recency allows users to prioritize which films to watch first. In addition, films that have stayed in a watchlist for longer periods of time may no longer have caught the interest of the user, and could be an incentive for removal.
**Engagement with reviewer's point:** There may be users that use watchlists purely to build a list of movies to watch. Ordering by title would be more organized and efficient for finding a specific film in a long list. That said, this is a narrower use case than deciding what to watch next, which is what most users are doing when they open their watchlist — so I'm implementing the date-added default as proposed.

## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->