Read `TEST_CASES.md` and `schedule_based_climate_control.yaml` in full.

From the YAML, extract the exact decision logic — the outer `choose` conditions and the inner `choose` conditions — as they are currently written (including any `trigger.entity_id` checks or template expressions).

Then, for every test case in `TEST_CASES.md`, trace through that logic step by step using the inputs described in the "State at trigger time" and "Trigger" columns. Determine which branch fires (or that no branch fires) and compare the result to the "Expected outcome" column.

Use these mappings for trigger types:
- "Temp sensor update" → `trigger.entity_id` is the temp sensor (not the schedule entity)
- "Schedule turns on/off" → `trigger.entity_id` IS the schedule entity
- "Vacation mode turns on/off" → `trigger.entity_id` is the vacation entity (not the schedule entity)
- "Party mode turns on/off" → `trigger.entity_id` is the party entity (not the schedule entity)
- "Window open/closed (after delay)" → `trigger.entity_id` is a window sensor (not the schedule entity)

Use these mappings for expected outcomes:
- "AC turns off" / "AC stays off" → outer branch 1 fires (`climate.turn_off`)
- "AC set to cool mode" → inner branch 1 fires (`climate.set_temperature` with `hvac_mode: cool`)
- "AC set to heat mode" → inner branch 2 fires (`climate.set_temperature` with `hvac_mode: heat`)
- "No action" / "AC stays on" / "AC continues" → no branch fires, or automation is passive
- "No action — AC untouched" / "AC continues running" → outer branch 2 fires (`stop`) or no branch fires

For every test case, attempt to verify it from the YAML alone. A case is verifiable when:
- The trigger type and "State at trigger time" together supply all inputs needed to evaluate every template expression and every `for:` / `to:` trigger constraint.
- The expected outcome maps to a determinate branch in the action logic (or to "no branch fires").

If and only if a case cannot be verified this way — because the test description leaves a required input ambiguous, or the outcome depends on HA runtime behaviour that is not encoded in the YAML at all — emit a ⚠ WARNING row instead of SKIP, with a one-sentence explanation of what information is missing. Do not pre-judge any case as unverifiable; work through the logic first.

Also check version consistency: the `*Version:*` string in the blueprint description must match the `**Current Version:**` string in `README.md`. Report PASS or FAIL for this check too.

Output a result table with columns: ID | Expected | Result | PASS/FAIL.
After the table, print a summary line: "X/Y passed" where Y excludes SKIPs.
If any case fails, explain what the logic actually produces and why it differs from the expected outcome.
