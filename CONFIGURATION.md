*AI use: Claude drafted this document from my concept and midterm; I reviewed it and set the boundaries and approvals.*

# The Vizier's Room — Initial Configuration Document

*Status: Initial, G1 (16 Nov 2026). Update at G2/G3, Final at G4. Open questions are marked **[OPEN]**.*

## 1. Role-coverage map (planned at G1)
I am the Executive Producer and Creative Director. I make the design decisions and approve every merge. The agents recommend and build. Roles follow Chandler, *The Game Production Toolbox*, §2.2–2.3.

| Role | Executable artifact (planned) | Purpose and outputs | Context / tools | Handoff | Human approval boundary |
|---|---|---|---|---|---|
| Producer | Claude Code session + Asana | Weekly plan, task breakdown, Risk Register updates | This plan, Asana | Tasks to implementation | I approve priorities and cuts |
| Design lead | Claude Code skill `design-check` | Checks changes against the GDD pillars; drafts room layouts | GDD, rooms | Layouts to QA | I accept every layout |
| Tech lead / implementation | Claude Code (local), repo only | Engine port, Pyxel board, validator, Vizier endpoint, tests | Repo, `gh` | PRs to reviewer | I merge |
| Independent reviewer | Separate read-only Claude Code session | Reviews rules and the agent boundary before merges | GDD, diff | `review.md` to me | Blocking findings stop the merge |
| Art direction | Claude Code session (Pyxel palette and tiles) | Original tile set and legend; no Prince of Persia art | Pyxel, GDD §8 | Assets to implementation | I approve the look |
| QA / playtest analyst | `explain-episode` skill (from my midterm) | Turns attempt logs into summaries and route-variety numbers | Logs only | Report to me | I decide what changes |
| Release | Claude Code session + CI | Tags, archived build, run instructions | Repo, CI | Release notes to me | I approve Gold |
| Business intelligence | Claude (chat) | Cost and latency tracking against budgets | Call logs | Budget report to producer | I approve spend |
| **In-game agent: the Vizier** | Function-calling endpoint (agent extension artifact) | ≤ 3 tile edits + one line between attempts | See §3 | Edits to validator | Validator and fallback; I set the cap |

**[OPEN]** Which role becomes the required agent extension artifact (plugin, skill or MCP server) beyond the Vizier endpoint?

## 2. Engine, hosting and tools
- Python 3 with **Pyxel** (MIT); rules, solver and tests ported from my midterm `pop_room` package.
- **Vizier endpoint** runs server-side (Cloudflare, the course's taught path), so model credentials never reach the client. **[OPEN]** Cloudflare Worker calling Vertex AI, or a small Python server: decided by 22 Nov.
- Gemini via Vertex AI, from the course-funded Google Cloud project.
- GitHub Actions CI on every PR: deterministic tests with a fake model client, so CI makes no cloud calls.
- Asana for tasks, PPP and the Risk Register after G1.

## 3. The Vizier's configuration
| Setting | Value |
|---|---|
| Purpose | Rearrange up to 3 traps toward the player's last route, and say one line about it |
| Model | `gemini-3.8-flash` (alternative `gemini-3.5-flash-lite`), location `global` |
| Context sources | Current layout; summaries of the last 3 attempts (tiles visited, end cell, cause); edit rules |
| Memory policy | Per-session only (isolated session state); nothing kept across sessions or players |
| World state vs agent knowledge | The Vizier never sees live moves, the solver's route, or the hidden gate-timer count |
| Output | JSON: ≤ 3 `{row, col, tile}` edits + `line` (≤ 80 characters) |
| Prohibited actions | Editing start, exit or the Prince's cell; more than 3 edits; any tile type not in the list; any room with no escape |
| Validation | Schema; edit count; allowed tiles and cells; BFS escape ≤ 40 steps |
| Timeout / retry | 6 s; 1 retry on invalid edits; then the scripted room |
| Fallback | Fixed rotation of hand-made rooms (also the A/B baseline) |
| Spending limits | 15 calls and $0.10 per session; $5 total for prototype and playtests |
| Logging | Prompt and output tokens, latency, accepted or rejected, rejection reason, config version |
| Human override | A `--scripted` switch disables the Vizier; I can lock any room |
| Named failure modes | Timeout; invalid JSON; illegal edit; unsolvable room; edits that ignore the last route; a line that gives away the solution |
| Test command | `python -m unittest discover tests` (CI workflow path set when the repo is created) |

## 4. Modes
`--vizier` (agent) and `--scripted` (fixed rotation), with identical rules, validator and logging, so the A/B comparison is fair.

## 5. Data and privacy
- Attempt logs store moves and outcomes only. Playtesters are labelled P1, P2, … with a consent script; no names in the repo or in model prompts.
- No credentials, personal playtester data or grades in the repo.

## 6. Open questions
- **[OPEN]** Flash or Flash-Lite, decided after timing 50 calls (task 3 of the plan).
- **[OPEN]** Should the validator reject edits that leave the player's last route untouched?
- **[OPEN]** Hosting choice for the endpoint (see §2).
