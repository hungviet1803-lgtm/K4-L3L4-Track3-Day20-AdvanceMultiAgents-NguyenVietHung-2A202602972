---
name: log-triage-json-conventions
description: Use when turning messy log files into structured JSON with repeat counts and strict review-bot formatting.
---
1. Parse every log entry, keep only ERROR and CRITICAL levels regardless of capitalization, and ignore INFO, DEBUG, WARN, and WARNING entries.
2. Convert timestamps to UTC and format them exactly as `YYYY-MM-DDTHH:MM:SSZ`.
3. Extract the message text after `<service>: ` on the first line, capture the last traceback line as `exception`, and use `null` when no traceback exists.
4. Compute `repeat_count` as 1 plus every `-- last message repeated N times --` line that follows the entry, including repeated lines after any traceback.
5. Normalize service names to lower-case with hyphens replaced by underscores.
6. Sort the `errors` array by service first and timestamp_utc second, both ascending.
7. Write the top-level JSON object with `schema_version: 2`, `generated_by: "log-triage"`, the sorted `errors` array, and `counts_by_service` summed from `repeat_count` per normalized service.
8. Self-check:
   - only ERROR/CRITICAL entries included
   - timestamps are UTC strings
   - services are normalized and sorted
   - schema_version and generated_by are present
