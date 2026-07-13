# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- We will fill this in at the end to document our collaboration -->

## Comment 1 — Rename
**What I did:** Renamed `save_to_watchlist()` to `add_to_watchlist()` in `services/watchlist_service.py` and updated its call site inside `routes/watchlist/watchlist.py` to preserve the codebase's uniform verb-to-noun naming standards.
**How I verified:** Conducted a global project search for any remaining instances of `save_to_watchlist` and verified that application routes execute seamlessly.

## Comment 2 — Deduplication
**What I did:** Implemented an explicit deduplication check inside `add_to_watchlist()` by querying `WatchlistEntry` first. If a match is found, it raises a new domain-specific `AlreadyOnWatchlistError` exception, cleanly replicating the deduplication design seen in `add_to_collection()`.
**How I verified:** Wrote and executed automated unit tests checking that sequential calls with identical arguments throw an `AlreadyOnWatchlistError` and assert that duplicate rows are blocked from saving.

## Comment 3 — Missing test
**What I did:** Created `tests/test_watchlist.py` and implemented `test_add_to_watchlist_nonexistent_film_raises`. This test isolates database operations using an in-memory SQL session fixture and validates that looking up an invalid film ID gracefully propagates a `FilmNotFoundError`.
**How I verified:** Ran `pytest tests/ -v` to confirm both the new testing target and existing collection suites pass successfully.

## Comment 4 — Default visibility
**My position:** Retain `public=True` as the system default configuration for new watchlist entries.
**Reasoning:** CineLog is positioned structurally as a community-centric social platform, meaning its core user retention loops rely heavily on shared discovery, viral activity feeds, and social interactions. Defaulting to public visibility maximizes social engagement, fuels the platform's network flywheel, and encourages organic conversation around shared cinema interests.
**Tradeoff acknowledged:** This design choice intentionally introduces minor friction for privacy-conscious users, who must explicitly opt out to hide entries. However, optimizing for community vibrancy out-of-the-box yields a higher engagement payout than a siloed, closed-by-default architecture.
## Comment 5 — Sort order
**My position:** Maintain alphabetical sorting (`Film.title.asc()`) for the watchlist, intentionally diverging from the collection's chronological sorting pattern.
**Reasoning:** The behavioral intent of these two features is completely distinct. A film collection is a chronological record of historical events ("What did I watch last week?"). A watchlist is an actionable utility log used to answer an immediate question: "What should I watch right now?" When a user scans a large watchlist, chronological ordering forces cognitive strain as titles shift unpredictably. Alphabetical sorting enables rapid, predictable visual indexing ($O(1)$ scanning friction), helping users quickly find specific films.
**Engagement with reviewer's point:** While consistency across endpoints is generally preferred, forcing chronological sorting here compromises the core UX utility of a watchlist. Good system design means prioritizing explicit feature-level user behaviors over dogmatic architectural symmetry.