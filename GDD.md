*AI use: Claude drafted this document from my concept and midterm; I reviewed it and made the design decisions.*

# The Vizier's Room — Initial Game Design Document

*Status: Initial GDD for G1 (16 Nov 2026). Author: Yusuf Sheikh (Executive Producer and Creative Director). Open questions are marked **[OPEN]**.*

## 1. The X
You escape a trapped palace room one careful move at a time, and you feel outsmarted, then clever.

## 2. Player and motivation
- **Player:** people who play turn-based puzzle and puzzle-platformer games and are happy to die and retry.
- **Motivation:** mastery, meaning getting better at reading a dangerous room.
- **Observable behaviour if it works:** players restart within 10 seconds of dying, and take a different route on the next attempt instead of repeating their last one.

## 3. Pillars
1. **Readable deaths.** Every death names its cause on screen.
2. **The room answers you.** Changes target where the player actually went.
3. **Always escapable.** No room is shown unless the solver finds a route within the step limit.

## 4. Concepts considered
| Concept | Why it lost |
|---|---|
| Indiana Jones and the Fate of Atlantis puzzle (my Week 3 idea) | Inventory and dialogue state I couldn't verify from the source; reuse terms unchecked |
| More Prince of Persia rooms, no agent | A level designer does the same job, so there is no agent hypothesis to test |
| **The Vizier's Room (chosen)** | Reuses my tested midterm engine and gives the agent one bounded, testable job |

## 5. Core loop
1. **Act:** choose one of six moves: `left`, `right`, `jump_left`, `jump_right`, `climb`, `wait`.
2. **Respond:** the board shows where the Prince lands, his health, and whether the gate is open.
3. **Interpret:** read the drops, spikes and gate timer. After a death, read what the Vizier changed.
4. **Next action:** choose the next step, or a new route on the next attempt.

**Outer loop:** escape or die → the Vizier rearranges up to 3 traps → next attempt. A session is 15 attempts.

## 6. Rules (carried over from the midterm)
- Grid room, one floor per row. Tiles: start, exit, floor, empty, spikes, pressure plate, gate, wall.
- Falls: 1 floor is safe, 2 floors costs 1 health, 3 or more kills. The Prince has 2 health.
- Spikes kill on landing or walking in. Walls are solid.
- The pressure plate opens the gate for the next 3 turns.
- Step limit: 40 moves per attempt.
- Moves that kill are legal; impossible moves are refused.

## 7. The agent: the Vizier
- **Job:** between attempts, propose up to **3 tile edits** (move spikes, open or close a gap, move the plate) aimed at the route the player used, plus **one line** of in-world text saying what changed.
- **Context it receives:** the current layout and summaries of the last 3 attempts (tiles visited, where and how the attempt ended).
- **What it never sees:** live moves during an attempt, the solver's route, or the hidden gate-timer count.
- **Validation:** edits must be legal tile types on legal cells (never the start, the exit or the Prince's cell), at most 3, and the BFS solver must find an escape within 40 steps. If it fails twice, or the call takes longer than 6 s, the scripted room is used and the log records why.
- **Scripted baseline:** a fixed rotation of hand-made rooms, checked by the same validator.
- **Hypothesis:** with the Vizier, players feel the room noticed them, vary their routes more, and restart faster than with the fixed rotation.
- **Playtest question:** over 10 attempts, do Vizier players vary their route more, restart faster, and say the room "noticed" them, compared with the fixed rotation?
- **What the player loses with the baseline:** the room reacting to their favourite route.

## 8. Presentation
- Pyxel window: tile board with a **legend** and a clear **Prince marker** (fixing my midterm playtester's note that the map was confusing).
- HUD: health, gate open or closed, steps left, attempt number.
- Death or escape screen (about 2 s) shows the cause and then the Vizier's line. The Vizier's call runs behind this screen.

## 9. Scope
- **Included:** one room; six moves; falls, spikes, walls, plate and gate; the Vizier and validator; scripted rotation mode; attempt log; Pyxel board; 15-attempt sessions.
- **Excluded:** guards and combat, potions, multiple rooms, real-time movement, sound, and any original Prince of Persia code, art or name.
- **Cut first, in order:** (1) the Vizier's line; (2) pixel-art polish, using plain coloured tiles instead; (3) the end-of-session summary screen.

## 10. Budgets
- **Latency:** one model call per attempt, target ≤ 3 s at p90, hidden behind the ~2 s death or escape screen; hard timeout 6 s, then the scripted room. Moves never call the model.
- **Cost:** 15 calls × (≈2,000 input + ≈1,000 output tokens) at Gemini 3.8 Flash's $0.75 / $3.75 per 1M tokens (Google Cloud pricing page, checked 2 Nov 2026) ≈ **$0.08 per session**; cap $0.10. Gemini 3.5 Flash-Lite ($0.30 / $2.50) would be ≈ $0.05.
- **Largest uncertainty:** latency. In my midterm, Gemini calls hit the 30 s timeout up to 11 times in one room.

## 11. References and attribution
| Source | Borrowed → changed | Terms |
|---|---|---|
| Prince of Persia, Apple II (1989), Jordan Mechner's released source | Plate-opened timed gate, spikes, fall damage by floors → turn-based, one room, 2 health | Mechner allows studying and modifying the code, but grants no rights in Prince of Persia (Ubisoft). Rules only: no code, sprites or name |
| Left 4 Dead's AI Director | A director acting between encounters → visible, capped, solver-checked | Idea only |
| My COMS E6998 midterm (`midterms/alsjoee/`) | World rules, BFS solver, Gemini player, traces → reused as the engine | My own code |
| Pyxel | Engine | MIT licence |
| Gemini on Vertex AI | The Vizier's model | Google Cloud terms |

## 12. Scoring, win condition and edge cases
- **Session:** 15 attempts. **Score:** escapes in the session. An attempt is won by reaching the exit within 40 steps.
- **Edge cases:** the Vizier may not edit the start, the exit or the Prince's cell; if every proposal fails validation, the scripted room is used; a timed-out call never blocks play.
- **[OPEN]** Balance: how many escapes out of 15 should a first-time player get? Target to be set from the first playtest.

## 13. Open questions
- **[OPEN]** Should the Vizier also be able to *remove* a trap, so that it can help a player who keeps failing?
- **[OPEN]** Does a session end after 15 attempts, or after a set number of escapes?
- **[OPEN]** Is Gemini 3.8 Flash fast enough, or does Flash-Lite give acceptable edits at lower latency? This gets decided by timing 50 calls.
- **[OPEN]** Does the Vizier's line help players read the change, or give the answer away?
- **[OPEN]** What room size keeps 3 edits meaningful but not overwhelming?
