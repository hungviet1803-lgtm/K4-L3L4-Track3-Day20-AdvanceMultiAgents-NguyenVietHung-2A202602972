### Human
The test suite of the `bookings` package in the folder `workspace/` is failing.

Fix the source code so that the whole test suite passes and every function does what its docstring says.
The visible tests do not cover everything: the docstrings are the specification.
Do not modify the existing files in `tests/`.
Your changes are also checked by Acme's review bot against the Acme Python team conventions.

When you are done, reply with a short summary of what you changed and why.


### Tool call: ls
{"path": "/workspace"}

### Tool result
['/workspace/CHANGELOG.md', '/workspace/README.md', '/workspace/bookings/', '/workspace/tests/']

### Tool call: task
{"subagent_type": "explorer", "description": "Inspect the workspace/bookings package and tests to determine conventions, docstring specs, failing-edge cases, and likely bug locations. Read README.md, CHANGELOG.md, package source, and a sample of tests. Do not modify files. Return a factual report with file paths, key behaviors required by docstrings, and any notable edge cases such as validation rules or date/time handling. The workspace root is /workspace. Relevant files are under workspace/bookings/ and workspace/tests/. Use relative paths in your reasoning but report absolute sandbox paths if needed."}

### Tool call: ls
{"path": "/workspace/bookings"}

### Tool call: ls
{"path": "/workspace/tests"}

### Tool result
## Scope inspected
- `workspace/README.md`
- `workspace/CHANGELOG.md`
- `workspace/bookings/__init__.py`
- `workspace/bookings/billing.py`
- `workspace/bookings/calendar_export.py`
- `workspace/bookings/schedule.py`
- `workspace/bookings/timeutil.py`
- `workspace/tests/test_bookings.py`

## Stated conventions / rules (verbatim where useful)

### `workspace/README.md`
- “Small package.”
- “Run the tests from the `workspace/` folder:”
- Command shown: `cd workspace && python -m pytest tests -q`

### `workspace/CHANGELOG.md`
- `## Unreleased`
- `## 1.4.2`
- `- initial import`

### `workspace/bookings/billing.py`
Docstring for `billable_blocks(minutes, block=15)`:
- “Number of billable blocks for a booking.”
- “A partial block is billed as a full block (always round UP):”
- “16 minutes with 15-minute blocks -> 2 blocks.”
- “`minutes` must be >= 0; a negative value raises ValueError.”

### `workspace/bookings/calendar_export.py`
Docstring for `slot_end(start_minute, duration_text)`:
- “Minute of the day at which a slot ends.”

### `workspace/bookings/schedule.py`
Docstring for `total_minutes(durations)`:
- “Total minutes of a list of duration strings.”

Docstring for `add_slot(slot, slots=[])`:
- “Return a NEW list containing the given slots plus `slot`.”
- “Separate calls must not share state: calling add_slot("a") and then”
- “add_slot("b") returns ["a"] and then ["b"].”

### `workspace/bookings/timeutil.py`
Docstring for `parse_duration(text)`:
- “Convert a duration string to w

### Tool result
['/workspace/bookings/__init__.py', '/workspace/bookings/__pycache__/', '/workspace/bookings/billing.py', '/workspace/bookings/calendar_export.py', '/workspace/bookings/schedule.py', '/workspace/bookings/timeutil.py']

### Tool result
['/workspace/tests/__pycache__/', '/workspace/tests/test_bookings.py']

### Tool call: read_file
{"file_path": "/workspace/bookings/timeutil.py"}

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

### Tool call: read_file
{"file_path": "/workspace/bookings/billing.py"}

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

### Tool call: read_file
{"file_path": "/workspace/bookings/schedule.py"}

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

### Tool call: read_file
{"file_path": "/workspace/bookings/calendar_export.py"}

### Tool result
@@ lines 1-7 of 7 @@
"""Calendar export."""
from .timeutil import parse_duration


def slot_end(start_minute, duration_text):
    """Minute of the day at which a slot ends."""
    return start_minute + parse_duration(duration_text)

### Tool call: execute
{"command": "python - <<'PY'\nfrom pathlib import Path\np = Path('/workspace/bookings/timeutil.py')\np.write_text('''\"\"\"Time helpers for the bookings package.\"\"\"\nimport re\n\n\ndef parse_duration(text):\n    \"\"\"Convert a duration string to whole minutes (int).\n\n    Supported formats:\n      \"1h30m\" -> 90\n      \"2h\"    -> 120\n      \"45m\"   -> 45\n      \"1:30\"  -> 90\n      \"45\"    -> 45    # a bare number means minutes\n    Surrounding whitespace is ignored.\n    \"\"\"\n    text = text.strip()\n    if re.fullmatch(r\"\\d+\", text):\n        return int(text)\n    m = re.fullmatch(r\"(?:(\\d+)h)?(?:(\\d+)m)?\", text)\n    if m and text:\n        hours = int(m.group(1) or 0)\n        minutes = int(m.group(2) or 0)\n        if 'h' in text or 'm' in text:\n            return hours * 60 + minutes\n    if ':' in text:\n        hours, minutes = text.split(':', 1)\n        if hours.isdigit() and minutes.isdigit():\n            return int(hours) * 60 + int(minutes)\n    raise ValueError(f\"bad duration: {text!r}\")\n''')\n\np = Path('/workspace/bookings/billing.py')\np.write_text('''\"\"\"Billing rules.\"\"\"\n\nimport math\n\n\ndef billable_blocks(minutes, block=15):\n    \"\"\"Number of billable blocks for a booking.\n\n    A partial block is billed as a full block (always round UP):\n    16 minutes with 15-minute blocks -> 2 blocks.\n    `minutes` must be >= 0; a negative value raises ValueError.\n    \"\"\"\n    if minutes < 0:\n        raise ValueError(\"mi

### Tool result

no tests ran in 0.00s

[stderr] <stdin>:19: SyntaxWarning: invalid escape sequence '\d'
[stderr] Traceback (most recent call last):
[stderr]   File "<stdin>", line 3, in <module>
[stderr]   File "/usr/local/lib/python3.12/pathlib.py", line 1047, in write_text
[stderr]     with self.open(mode='w', encoding=encoding, errors=errors, newline=newline) as f:
[stderr]          ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
[stderr]   File "/usr/local/lib/python3.12/pathlib.py", line 1013, in open
[stderr]     return io.open(self, mode, buffering, encoding, errors, newline)
[stderr]            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
[stderr] FileNotFoundError: [Errno 2] No such file or directory: '/workspace/bookings/timeutil.py'
[stderr] ERROR: file or directory not found: /workspace/tests

Exit code: 4
[Command failed with exit code 4]

### Tool call: execute
{"command": "python - <<'PY'\nfrom pathlib import Path\nprint(Path('workspace/bookings').resolve())\nprint(Path('workspace/tests').resolve())\nPY"}

### Tool result
/tmp/lab-code-eval-dbswr5hy/workspace/bookings
/tmp/lab-code-eval-dbswr5hy/workspace/tests

[Command succeeded with exit code 0]

### Tool call: execute
{"command": "python - <<'PY'\nfrom pathlib import Path\np = Path('workspace/bookings/timeutil.py')\np.write_text('''\"\"\"Time helpers for the bookings package.\"\"\"\nimport re\n\n\ndef parse_duration(text):\n    \"\"\"Convert a duration string to whole minutes (int).\n\n    Supported formats:\n      \"1h30m\" -> 90\n      \"2h\"    -> 120\n      \"45m\"   -> 45\n      \"1:30\"  -> 90\n      \"45\"    -> 45    # a bare number means minutes\n    Surrounding whitespace is ignored.\n    \"\"\"\n    text = text.strip()\n    if re.fullmatch(r\"\\d+\", text):\n        return int(text)\n    m = re.fullmatch(r\"(?:(\\d+)h)?(?:(\\d+)m)?\", text)\n    if m and text:\n        hours = int(m.group(1) or 0)\n        minutes = int(m.group(2) or 0)\n        if 'h' in text or 'm' in text:\n            return hours * 60 + minutes\n    if ':' in text:\n        hours, minutes = text.split(':', 1)\n        if hours.isdigit() and minutes.isdigit():\n            return int(hours) * 60 + int(minutes)\n    raise ValueError(f\"bad duration: {text!r}\")\n''')\n\nPath('workspace/bookings/billing.py').write_text('''\"\"\"Billing rules.\"\"\"\n\nimport math\n\n\ndef billable_blocks(minutes, block=15):\n    \"\"\"Number of billable blocks for a booking.\n\n    A partial block is billed as a full block (always round UP):\n    16 minutes with 15-minute blocks -> 2 blocks.\n    `minutes` must be >= 0; a negative value raises ValueError.\n    \"\"\"\n    if minutes < 0:\n        raise ValueError(\"minutes mus

### Tool result

==================================== ERRORS ====================================
______________ ERROR collecting workspace/tests/test_bookings.py _______________
ImportError while importing test module '/tmp/lab-code-eval-dbswr5hy/workspace/tests/test_bookings.py'.
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
1 error in 0.23s

[stderr] <stdin>:19: SyntaxWarning: invalid escape sequence '\d'

Exit code: 2
[Command failed with exit code 2]

### Tool call: execute
{"command": "PYTHONPATH=workspace python -m pytest workspace/tests -q", "timeout": 120}

### Tool result
...                                                                      [100%]
3 passed in 0.04s

[Command succeeded with exit code 0]

### Assistant
Fixed the `bookings` package so the test suite passes:
- Updated `workspace/bookings/timeutil.py` to support all duration formats in the docstring.
- Updated `workspace/bookings/billing.py` to round billable blocks up and reject negative minutes.
- Updated `workspace/bookings/schedule.py` to avoid shared mutable default state and always return a new list.

