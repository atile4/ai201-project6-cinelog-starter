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
**What conflicted:** I ran `git fetch origin` and `git rebase origin/main` to rebase `feature/watchlist` on top of the merged UUID refactor (`07ca580: refactor: migrate film IDs from integer to UUID`). The rebase completed without prompting for a manual conflict resolution — but this masked a real problem rather than indicating a clean rebase. My commit that originally added the `WatchlistEntry` model (`77d8958`) had been written against the pre-refactor version of `models.py`. When replayed on top of `origin/main`'s already-refactored `models.py`, git's merge algorithm didn't recognize my model addition as conflicting with the surrounding UUID changes and silently dropped it instead of applying it or flagging a conflict.

**How I resolved it:** I used `git reflog` to find the commit hash my branch pointed to immediately before the rebase started (`594e6ed`), then ran `git show 594e6ed:models.py` to recover the original `WatchlistEntry` class definition as it existed pre-rebase. Rather than reapplying that old definition as-is, I manually adapted it to match the post-refactor convention already established by `CollectionEntry`: changing `film_id` from `db.Integer` to `db.String(36)` (matching `Film.id`'s UUID type) and re-adding the class to the current `models.py`. I also updated `add_to_watchlist()`'s docstring, which still described `film_id` as `(int)` with a `pre-refactor` note, and updated my Comment 3 test's fake ID from an integer (`999999`) to a UUID-shaped string, matching the pattern already used in `test_add_to_collection_nonexistent_film_raises`.

Here, I chose to leave the unique-constraint decision app level, via `AlreadyInWatchlistError`. Doing so keeps migrations simpler and handling errors would be more flexible as opposed to SQLAlchemy raising a generic `IntegrityError`. 

**How I verified no conflict remains:** Ran `pytest tests/ -v` for the full suite after the model fix and confirmed all tests pass, including the previously-broken `test_watchlist.py`. Ran `git grep -n "film_id" -- routes/ services/` to check for any other lingering integer-`film_id` assumptions elsewhere in the codebase (e.g. manual type coercion in route handlers). Confirmed via `git log --oneline --graph` that the branch history contains no merge commits — only a linear sequence of rebased commits on top of `origin/main`.


## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->