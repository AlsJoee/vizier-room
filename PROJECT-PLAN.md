*AI use: Claude drafted this plan from my concept, midterm and the course syllabus; I reviewed it and own the dates and priorities.*

# The Vizier's Room — Initial Project Plan and Risk Register

*Status: Initial, G1 (16 Nov 2026). Moves into Asana after G1, where owners, acceptance criteria, dependencies and the full Risk Register are added. The plan is Update at G2 and Final at G3 scope lock.*

## Gates and milestones
| Gate / milestone | Date | What I bring |
|---|---|---|
| G1 Concept | Mon 16 Nov (recording Sun 15 Nov) | Pitch, deck, Initial GDD, Configuration Document, this plan |
| Checkpoint | Mon 23 Nov | Deployed model call, isolated session state, forced-timeout fallback |
| G2 Prototype + G3 Production (combined) | Mon 30 Nov (recording Sat 28 Nov) | First playable with one external playtest, Initial Playtest Log, scope-lock plan, Update GDD and Configuration, updated Risk Register |
| Alpha / Beta | Wed 2 Dec / Fri 4 Dec | Feature-complete build / release candidate; external playtest after Alpha |
| G4 Final / Pre-Launch | Mon 7 Dec (Gold recording Sat 5 Dec) | Final GDD and Configuration, evaluation harness, Gold build |
| G5 Retrospective / Closure | Mon 14 Dec | Final game and AAR, Playtest Log |
| Finals | 17–23 Dec (slot TBD) | Retrospective, 8–10 min walkthrough, live presentation |

## First tasks through G2/G3
| # | Task | Due | Done when |
|---|---|---|---|
| 1 | Create the repo; port `world.py`, `rooms.py`, BFS solver and tests from my midterm; CI on every PR | Thu 19 Nov | Tests pass in CI |
| 2 | Vizier endpoint (server-side, credentials off the client) with per-session state, 6 s timeout and scripted fallback | Sun 22 Nov | Forced-timeout test returns the scripted room; demo ready for 23 Nov |
| 3 | Time 50 Vizier calls on Flash and Flash-Lite | Sun 22 Nov | p50/p90 latency and cost in the README |
| 4 | Pyxel board with legend, Prince marker and HUD | Tue 24 Nov | I can play a room with the keyboard |
| 5 | Attempt summaries, log, and scripted rotation mode | Wed 25 Nov | A 15-attempt scripted session writes a log |
| 6 | Validator for Vizier edits; independent rules review before merge | Thu 26 Nov | Invalid edits rejected in tests; `review.md` with no blocking findings |
| 7 | External playtest of the core loop (Vizier vs scripted) and Initial Playtest Log | Fri 27 Nov | Route variety, restart time and the "noticed" question recorded |
| 8 | Record the G2/G3 presentation | Sat 28 Nov | 3–5 min recording posted |

## Initial Risk Register
| ID | Dimension | Risk | Rating | Evidence | Mitigation | Trigger to re-rate |
|---|---|---|---|---|---|---|
| R1 | IP / Concept | Too close to Prince of Persia | Medium | Ubisoft holds the IP; Mechner grants no rights | Rules only; original art and name | Any reused asset or name |
| R2 | Business / Audience | Players can't read the room | Medium | Midterm playtester found the map confusing | Legend and Prince marker; test with external players | Playtester can't name a death cause |
| R3 | Technology | Model latency or invalid edits | **High** | Midterm calls hit the 30 s timeout up to 11× in one room | Call behind the death screen; 6 s timeout; validator; scripted fallback | p90 > 3 s in task 3 |
| R4 | Team | Solo with agent team; missed classes | Medium | Midterm needed a deadline-day CI fix | Weekly Asana tasks; reviewer agent before merges | Two tasks slip in a week |
| R5 | Genre / Mechanics | Adaptive rooms feel random or unfair | **High** | Untested; no evidence yet | 3-edit cap; the Vizier names its change; solver guarantee; A/B playtest | Playtesters call changes "random" |
| R6 | Timeline | Not enough time for G2/G3 | Medium | G1 submitted a day late; 2 weeks to G2/G3 | Reuse midterm engine; cut list in the GDD | Task 4 not done by 25 Nov |

**Shared cause of the Highs (R3, R5):** both come from the agent changing the room between attempts. Tasks 2, 3, 6 and 7 test that first.
