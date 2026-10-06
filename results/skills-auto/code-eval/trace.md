### Human
The test suite of the `bookings` package in the folder `workspace/` is failing.

Fix the source code so that the whole test suite passes and every function does what its docstring says.
The visible tests do not cover everything: the docstrings are the specification.
Do not modify the existing files in `tests/`.
Your changes are also checked by Acme's review bot against the Acme Python team conventions.

When you are done, reply with a short summary of what you changed and why.


### Tool call: read_file
{"file_path": "/skills/python-package-fix-conventions/SKILL.md", "limit": 1000}

### Tool result
@@ lines 1-14 of 14 @@
---
name: python-package-fix-conventions
description: Use when fixing a Python package so tests pass and review-bot conventions must also be satisfied.
---
1. Run the test suite from the workspace root with the package import path set correctly; if imports fail, fix the environment or package layout before editing logic.
2. Read docstrings and failing tests as the specification, then inspect public functions for missing type annotations and add annotations to every parameter and return value.
3. After each code change, rerun the relevant tests until the whole suite passes.
4. When you fix bugs, add a regression test file under the tests directory with one test function per bug fixed, and make sure it passes without modifying existing tests.
5. Record each fix in the changelog under `## Unreleased` as bullets in the form `- fix(<function name>): <short description>`.
6. Self-check:
   - all public functions have parameter and return annotations
   - regression tests exist for each fixed bug
   - changelog entries were added under `## Unreleased`
   - full test suite passes

### Tool call: ls
{"path": "/workspace"}

### Tool result
['/workspace/CHANGELOG.md', '/workspace/README.md', '/workspace/bookings/', '/workspace/tests/']

### Tool call: glob
{"path": "/workspace/bookings", "pattern": "*.py"}

### Tool call: glob
{"path": "/workspace/tests", "pattern": "*.py"}

### Tool call: read_file
{"file_path": "/workspace/CHANGELOG.md", "limit": 200}

### Tool call: read_file
{"file_path": "/workspace/README.md", "limit": 200}

### Tool result
['/workspace/bookings/__init__.py', '/workspace/bookings/billing.py', '/workspace/bookings/calendar_export.py', '/workspace/bookings/schedule.py', '/workspace/bookings/timeutil.py']

### Tool result
['/workspace/tests/test_bookings.py']

### Tool result
@@ lines 1-6 of 6 @@
# Changelog

## Unreleased

## 1.4.2
- initial import

### Tool result
@@ lines 1-5 of 5 @@
# bookings

Small package. Run the tests from the `workspace/` folder:

    cd workspace && python -m pytest tests -q

### Tool call: read_file
{"file_path": "/workspace/bookings/__init__.py", "limit": 200}

### Tool call: read_file
{"file_path": "/workspace/bookings/billing.py", "limit": 400}

### Tool call: read_file
{"file_path": "/workspace/bookings/calendar_export.py", "limit": 400}

### Tool call: read_file
{"file_path": "/workspace/bookings/schedule.py", "limit": 400}

### Tool call: read_file
{"file_path": "/workspace/bookings/timeutil.py", "limit": 400}

### Tool call: read_file
{"file_path": "/workspace/tests/test_bookings.py", "limit": 400}

### Tool result
@@ lines 1-1 of 1 @@
__version__ = "1.4.2"

### Tool result
@@ lines 1-11 of 11 @@
"""Billing rules."""


def billable_blocks(minutes, block=15):
    """Number of billable blocks for a booking.

    A partial block is billed as a full block (always round UP):
    16 minutes with 15-minute blocks -> 2 blocks.
    `minutes` must be >= 0; a negative value raises ValueError.
    """
    return round(minutes / block)

### Tool result
@@ lines 1-7 of 7 @@
"""Calendar export."""
from .timeutil import parse_duration


def slot_end(start_minute, duration_text):
    """Minute of the day at which a slot ends."""
    return start_minute + parse_duration(duration_text)

### Tool result
@@ lines 1-17 of 17 @@
"""Schedule helpers."""
from .timeutil import parse_duration


def total_minutes(durations):
    """Total minutes of a list of duration strings."""
    return sum(parse_duration(d) for d in durations)


def add_slot(slot, slots=[]):
    """Return a NEW list containing the given slots plus `slot`.

    Separate calls must not share state: calling add_slot("a") and then
    add_slot("b") returns ["a"] and then ["b"].
    """
    slots.append(slot)
    return slots

### Tool result
@@ lines 1-19 of 19 @@
"""Time helpers for the bookings package."""
import re


def parse_duration(text):
    """Convert a duration string to whole minutes (int).

    Supported formats:
      "1h30m" -> 90
      "2h"    -> 120
      "45m"   -> 45
      "1:30"  -> 90
      "45"    -> 45    # a bare number means minutes
    Surrounding whitespace is ignored.
    """
    m = re.fullmatch(r"(\d+)h(\d+)m", text.strip())
    if not m:
        raise ValueError(f"bad duration: {text!r}")
    return int(m.group(1)) * 60 + int(m.group(2))

### Tool result
@@ lines 1-15 of 15 @@
from bookings.billing import billable_blocks
from bookings.schedule import add_slot, total_minutes


def test_total_minutes_mixed_formats():
    assert total_minutes(["1h30m", "2h"]) == 210


def test_billable_blocks_rounds_up():
    assert billable_blocks(20) == 2
    assert billable_blocks(45) == 3


def test_add_slot_single_call():
    assert add_slot("a") == ["a"]

### Tool call: execute
{"command": "python -m pytest tests -q", "timeout": 120}

### Tool result

no tests ran in 0.00s

[stderr] ERROR: file or directory not found: tests

Exit code: 4
[Command failed with exit code 4]

### Tool call: execute
{"command": "python -m pytest workspace/tests -q", "timeout": 120}

### Tool result

==================================== ERRORS ====================================
______________ ERROR collecting workspace/tests/test_bookings.py _______________
ImportError while importing test module '/tmp/lab-code-eval-6_xrnj40/workspace/tests/test_bookings.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
/usr/local/lib/python3.12/importlib/__init__.py:90: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
/lab/tasks/code-eval/workspace/tests/test_bookings.py:1: in <module>
    from bookings.billing import billable_blocks
E   ModuleNotFoundError: No module named 'bookings'
=========================== short test summary info ============================
ERROR workspace/tests/test_bookings.py
!!!!!!!!!!!!!!!!!!!! Interrupted: 1 error during collection !!!!!!!!!!!!!!!!!!!!
1 error in 0.27s

Exit code: 2
[Command failed with exit code 2]

### Tool call: read_file
{"file_path": "/workspace/bookings/billing.py", "limit": 200}

### Tool call: read_file
{"file_path": "/workspace/bookings/calendar_export.py", "limit": 200}

### Tool call: read_file
{"file_path": "/workspace/bookings/schedule.py", "limit": 200}

### Tool call: read_file
{"file_path": "/workspace/bookings/timeutil.py", "limit": 200}

### Tool result
@@ lines 1-11 of 11 @@
"""Billing rules."""


def billable_blocks(minutes, block=15):
    """Number of billable blocks for a booking.

    A partial block is billed as a full block (always round UP):
    16 minutes with 15-minute blocks -> 2 blocks.
    `minutes` must be >= 0; a negative value raises ValueError.
    """
    return round(minutes / block)

### Tool result
@@ lines 1-7 of 7 @@
"""Calendar export."""
from .timeutil import parse_duration


def slot_end(start_minute, duration_text):
    """Minute of the day at which a slot ends."""
    return start_minute + parse_duration(duration_text)

### Tool result
@@ lines 1-17 of 17 @@
"""Schedule helpers."""
from .timeutil import parse_duration


def total_minutes(durations):
    """Total minutes of a list of duration strings."""
    return sum(parse_duration(d) for d in durations)


def add_slot(slot, slots=[]):
    """Return a NEW list containing the given slots plus `slot`.

    Separate calls must not share state: calling add_slot("a") and then
    add_slot("b") returns ["a"] and then ["b"].
    """
    slots.append(slot)
    return slots

### Tool result
@@ lines 1-19 of 19 @@
"""Time helpers for the bookings package."""
import re


def parse_duration(text):
    """Convert a duration string to whole minutes (int).

    Supported formats:
      "1h30m" -> 90
      "2h"    -> 120
      "45m"   -> 45
      "1:30"  -> 90
      "45"    -> 45    # a bare number means minutes
    Surrounding whitespace is ignored.
    """
    m = re.fullmatch(r"(\d+)h(\d+)m", text.strip())
    if not m:
        raise ValueError(f"bad duration: {text!r}")
    return int(m.group(1)) * 60 + int(m.group(2))

### Tool call: write_file
{"file_path": "/workspace/bookings/timeutil.py", "content": "\"\"\"Time helpers for the bookings package.\"\"\"\nfrom __future__ import annotations\n\nimport re\n\n\ndef parse_duration(text: str) -> int:\n    \"\"\"Convert a duration string to whole minutes (int).\n\n    Supported formats:\n      \"1h30m\" -> 90\n      \"2h\"    -> 120\n      \"45m\"   -> 45\n      \"1:30\"  -> 90\n      \"45\"    -> 45    # a bare number means minutes\n    Surrounding whitespace is ignored.\n    \"\"\"\n    value = text.strip()\n    if re.fullmatch(r\"\\d+h\\d+m\", value):\n        hours, minutes = value[:-1].split(\"h\")\n        return int(hours) * 60 + int(minutes)\n    if re.fullmatch(r\"\\d+h\", value):\n        return int(value[:-1]) * 60\n    if re.fullmatch(r\"\\d+m\", value):\n        return int(value[:-1])\n    if re.fullmatch(r\"\\d+:\\d{1,2}\", value):\n        hours, minutes = value.split(\":\")\n        return int(hours) * 60 + int(minutes)\n    if re.fullmatch(r\"\\d+\", value):\n        return int(value)\n    raise ValueError(f\"bad duration: {text!r}\")\n"}

### Tool result
Updated file /workspace/bookings/timeutil.py

### Tool call: write_file
{"file_path": "/workspace/bookings/billing.py", "content": "\"\"\"Billing rules.\"\"\"\nfrom __future__ import annotations\n\nimport math\n\n\ndef billable_blocks(minutes: int, block: int = 15) -> int:\n    \"\"\"Number of billable blocks for a booking.\n\n    A partial block is billed as a full block (always round UP):\n    16 minutes with 15-minute blocks -> 2 blocks.\n    `minutes` must be >= 0; a negative value raises ValueError.\n    \"\"\"\n    if minutes < 0:\n        raise ValueError(\"minutes must be >= 0\")\n    if block <= 0:\n        raise ValueError(\"block must be > 0\")\n    return math.ceil(minutes / block)\n"}

### Tool result
Updated file /workspace/bookings/billing.py

### Tool call: write_file
{"file_path": "/workspace/bookings/schedule.py", "content": "\"\"\"Schedule helpers.\"\"\"\nfrom __future__ import annotations\n\nfrom .timeutil import parse_duration\n\n\ndef total_minutes(durations: list[str]) -> int:\n    \"\"\"Total minutes of a list of duration strings.\"\"\"\n    return sum(parse_duration(d) for d in durations)\n\n\ndef add_slot(slot: str, slots: list[str] | None = None) -> list[str]:\n    \"\"\"Return a NEW list containing the given slots plus `slot`.\n\n    Separate calls must not share state: calling add_slot(\"a\") and then\n    add_slot(\"b\") returns [\"a\"] and then [\"b\"].\n    \"\"\"\n    result = list(slots) if slots is not None else []\n    result.append(slot)\n    return result\n"}

### Tool result
Updated file /workspace/bookings/schedule.py

### Tool call: write_file
{"file_path": "/workspace/bookings/calendar_export.py", "content": "\"\"\"Calendar export.\"\"\"\nfrom __future__ import annotations\n\nfrom .timeutil import parse_duration\n\n\ndef slot_end(start_minute: int, duration_text: str) -> int:\n    \"\"\"Minute of the day at which a slot ends.\"\"\"\n    return start_minute + parse_duration(duration_text)\n"}

### Tool result
Updated file /workspace/bookings/calendar_export.py

### Tool call: write_file
{"file_path": "/workspace/tests/test_regressions.py", "content": "from bookings.billing import billable_blocks\nfrom bookings.schedule import add_slot\nfrom bookings.timeutil import parse_duration\n\n\ndef test_parse_duration_supported_formats():\n    assert parse_duration(\"1h30m\") == 90\n    assert parse_duration(\"2h\") == 120\n    assert parse_duration(\"45m\") == 45\n    assert parse_duration(\"1:30\") == 90\n    assert parse_duration(\"45\") == 45\n\n\ndef test_billable_blocks_rounds_up_and_validates():\n    assert billable_blocks(16) == 2\n    assert billable_blocks(0) == 0\n\n\ndef test_add_slot_does_not_share_state():\n    assert add_slot(\"a\") == [\"a\"]\n    assert add_slot(\"b\") == [\"b\"]\n"}

### Tool result
Updated file /workspace/tests/test_regressions.py

### Tool call: edit_file
{"file_path": "/workspace/CHANGELOG.md", "old_string": "## Unreleased\n", "new_string": "## Unreleased\n- fix(parse_duration): support all documented duration formats\n- fix(billable_blocks): round up and reject negative minutes\n- fix(add_slot): avoid shared default state and return a new list\n"}

### Tool result
Successfully replaced 1 instance(s) of the string in '/workspace/CHANGELOG.md'

### Tool call: execute
{"command": "PYTHONPATH=workspace python -m pytest workspace/tests -q", "timeout": 120}

### Tool result
......                                                                   [100%]
6 passed in 0.06s

[Command succeeded with exit code 0]

### Assistant
Fixed the `bookings` package so the documented behavior matches the implementation and the full test suite passes. Changed `workspace/bookings/timeutil.py`, `workspace/bookings/billing.py`, `workspace/bookings/schedule.py`, `workspace/bookings/calendar_export.py`, added `workspace/tests/test_regressions.py`, and updated `workspace/CHANGELOG.md` with the required unreleased fix notes.