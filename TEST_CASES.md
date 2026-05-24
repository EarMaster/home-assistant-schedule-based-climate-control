# Blueprint Behaviour Test Cases

Each case lists the state at the moment the automation runs, what triggers it, and the expected outcome. "No action" means the automation completes without calling any climate service — the device keeps whatever state it is already in.

---

## Schedule

| ID | State at trigger time | Trigger | Expected outcome |
|----|-----------------------|---------|-----------------|
| SCH-1 | 14:00 (outside schedule), room above `limit_hot`, AC off | Temp sensor update | No action — automation is passive outside schedule hours |
| SCH-2 | 04:00 (schedule just ended), room still above `limit_hot`, AC running | Schedule turns off | AC turns off |
| SCH-3 | 11:00 (outside schedule), AC manually turned on, room above `limit_hot` | Temp sensor update | No action — manual state preserved |
| SCH-4 | 11:00 (outside schedule), AC manually turned on, room in comfortable range | Temp sensor update | No action — manual state preserved |
| SCH-5 | 18:00 (schedule just started), AC was manually running, room above `limit_hot` | Schedule turns on | Automation takes over: AC set to cool mode at `goal_cool` |
| SCH-6 | 18:00 (schedule just started), AC was manually running, room in comfort zone | Schedule turns on | AC turns off |
| SCH-7 | 20:00 (inside schedule), room rises above `limit_hot` | Temp sensor update | AC set to cool mode at `goal_cool` |
| SCH-8 | 02:00 (inside schedule), room drops below `limit_cold` | Temp sensor update | AC set to heat mode at `goal_heat` |

---

## Vacation Mode

| ID | State at trigger time | Trigger | Expected outcome |
|----|-----------------------|---------|-----------------|
| VAC-1 | Schedule active, AC running | Vacation mode turns on | AC turns off immediately |
| VAC-2 | Outside schedule, AC manually running | Vacation mode turns on | AC turns off immediately |
| VAC-3 | Schedule active, vacation mode was on | Vacation mode turns off | Automatic control resumes based on current temp |
| VAC-4 | Outside schedule, vacation mode was on | Vacation mode turns off | No action — schedule is still inactive |
| VAC-5 | Schedule active, vacation on, room above `limit_hot` | Temp sensor update | AC turns off (vacation overrides temperature) |

---

## Party Mode

| ID | State at trigger time | Trigger | Expected outcome |
|----|-----------------------|---------|-----------------|
| PTY-1 | Schedule active, AC off, room above `limit_hot` | Party mode turns on | No action — AC untouched |
| PTY-2 | Schedule active, AC running in cool mode | Party mode turns on | No action — AC continues running |
| PTY-3 | Schedule active, party mode was on, room above `limit_hot` | Party mode turns off | AC set to cool mode at `goal_cool` |
| PTY-4 | Schedule active, vacation ON, party mode ON | Temp sensor update | AC turns off — vacation takes priority over party mode (current behaviour) |

> **Note on PTY-4**: The hierarchy stated in the README is safety → party → automatic. However, since the safety cut-off is evaluated first in the `choose`, vacation mode wins even when party mode is also on. This may or may not be the desired behaviour.

---

## Window Sensors

| ID | State at trigger time | Trigger | Expected outcome |
|----|-----------------------|---------|-----------------|
| WIN-1 | Schedule active, AC running, window opens | Window open for ≥ `window_delay` seconds | AC turns off |
| WIN-2 | Schedule active, single window was open, window closes | Window closed for ≥ `window_delay` seconds | Automatic control resumes based on current temp |
| WIN-3 | Outside schedule, AC manually on, window opens | Window open for ≥ `window_delay` seconds | AC turns off |
| WIN-4 | Schedule active, two windows — one closes while the other stays open | Window A closes (after delay) | AC stays off — window B still open |
| WIN-5 | Schedule active, AC running, window opens and closes within `window_delay` | Window state bounces | No action — delay prevents reaction to brief openings |
| WIN-6 | No window sensors configured | Any trigger | Window state has no effect on AC |

---

## Temperature Thresholds & Comfort Zone

| ID | State at trigger time | Trigger | Expected outcome |
|----|-----------------------|---------|-----------------|
| TMP-1 | Schedule active, temp at exactly `limit_hot` | Temp sensor update | AC set to cool mode at `goal_cool` |
| TMP-2 | Schedule active, temp at exactly `limit_cold` | Temp sensor update | AC set to heat mode at `goal_heat` |
| TMP-3 | Schedule active, AC cooling, temp drops below `limit_hot` but is above the comfort zone | Temp sensor update | No action — AC continues cooling |
| TMP-4 | Schedule active, AC cooling, temp enters comfort zone (`goal_heat + 0.5 < temp < goal_cool - 0.5`) | Temp sensor update | AC turns off |
| TMP-5 | Schedule active, AC off, temp is between `limit_cold` and `limit_hot`, outside comfort zone | Temp sensor update | No action — AC stays off |
| TMP-6 | Schedule active, AC off, temp rises from comfort zone back up toward `limit_hot` | Temp sensor update | No action until `limit_hot` is reached again |
| TMP-7 | Schedule active, `goal_cool` ≤ `goal_heat` (e.g. defaults: 21 and 22) | Temp sensor update | Comfort zone is an empty range — turn-off condition never fires |

---

## Hybrid Inputs (Helper entity vs. static value)

| ID | State at trigger time | Trigger | Expected outcome |
|----|-----------------------|---------|-----------------|
| INP-1 | `threshold_hot_ent` configured, entity state is a valid number | Temp sensor update | `limit_hot` read from the entity |
| INP-2 | `threshold_hot_ent` configured, entity state is `unavailable` or non-numeric | Temp sensor update | `limit_hot` falls back to the static `threshold_hot` value |
| INP-3 | No optional helper entities configured | Temp sensor update | All thresholds and targets use their static values |
| INP-4 | `target_temp_cool_ent` value is changed by the user on the dashboard | Input number changes state | Automation re-evaluates on next temp sensor trigger with new value |
