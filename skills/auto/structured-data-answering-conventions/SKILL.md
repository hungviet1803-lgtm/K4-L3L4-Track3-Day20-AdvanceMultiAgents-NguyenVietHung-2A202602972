---
name: structured-data-answering-conventions
description: Use when answering exact questions from CSV or JSON data and a review bot requires a strict output schema.
---
1. Parse the input data carefully, deduplicate exactly as instructed, and exclude missing-value sentinels from all aggregates.
2. Normalize categorical fields to the canonical spellings required by the task before grouping or comparing.
3. Write the requested JSON file with exactly the required top-level keys and any review-bot metadata keys, using the exact file name specified.
4. Store monetary outputs in integer cents, not decimals or strings, and include any required metadata object with the source file name, input row count, and rows used count.
5. If a cleaned CSV is required, write it with the exact requested header order, one row per distinct valid record, normalized categories, UTC timestamps formatted as `YYYY-MM-DDTHH:MM:SSZ`, and amounts in integer cents.
6. Self-check:
   - duplicates removed correctly
   - missing-value rows excluded from totals
   - required metadata present
   - required output files written in the exact formats
