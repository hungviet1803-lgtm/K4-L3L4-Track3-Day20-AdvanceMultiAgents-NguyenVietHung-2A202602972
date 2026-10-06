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

### Tool call: ls
{"path": "/workspace/bookings"}

### Tool result
['/workspace/bookings/__init__.py', '/workspace/bookings/__pycache__/', '/workspace/bookings/billing.py', '/workspace/bookings/calendar_export.py', '/workspace/bookings/schedule.py', '/workspace/bookings/timeutil.py']

### Tool call: read_file
{"file_path": "/workspace/bookings/billing.py"}

### Tool call: read_file
{"file_path": "/workspace/bookings/calendar_export.py"}

### Tool call: read_file
{"file_path": "/workspace/bookings/schedule.py"}

### Tool call: read_file
{"file_path": "/workspace/bookings/timeutil.py"}

### Tool call: read_file
{"file_path": "/workspace/bookings/__init__.py"}

### Tool call: glob
{"path": "/workspace/tests", "pattern": "*.py"}

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
@@ lines 1-1 of 1 @@
__version__ = "1.4.2"

### Tool result
['/workspace/tests/test_bookings.py']

### Tool call: read_file
{"file_path": "/workspace/tests/test_bookings.py"}

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
{"command": "python - <<'PY'\nfrom pathlib import Path\nbase=Path('/workspace/bookings')\n(base/'timeutil.py').write_text('''\"\"\"Time helpers for the bookings package.\"\"\"\nimport re\n\n\ndef parse_duration(text):\n    \"\"\"Convert a duration string to whole minutes (int).\n\n    Supported formats:\n      \"1h30m\" -> 90\n      \"2h\"    -> 120\n      \"45m\"   -> 45\n      \"1:30\"  -> 90\n      \"45\"    -> 45    # a bare number means minutes\n    Surrounding whitespace is ignored.\n    \"\"\"\n    text = text.strip()\n    patterns = [\n        (r\"(\\\\d+)h(\\\\d+)m\", lambda m: int(m.group(1)) * 60 + int(m.group(2))),\n        (r\"(\\\\d+)h\", lambda m: int(m.group(1)) * 60),\n        (r\"(\\\\d+)m\", lambda m: int(m.group(1))),\n        (r\"(\\\\d+):(\\\\d+)\", lambda m: int(m.group(1)) * 60 + int(m.group(2))),\n        (r\"(\\\\d+)\", lambda m: int(m.group(1))),\n    ]\n    for pattern, parser in patterns:\n        m = re.fullmatch(pattern, text)\n        if m:\n            return parser(m)\n    raise ValueError(f\"bad duration: {text!r}\")\n''')\n(base/'billing.py').write_text('''\"\"\"Billing rules.\"\"\"\n\n\ndef billable_blocks(minutes, block=15):\n    \"\"\"Number of billable blocks for a booking.\n\n    A partial block is billed as a full block (always round UP):\n    16 minutes with 15-minute blocks -> 2 blocks.\n    `minutes` must be >= 0; a negative value raises ValueError.\n    \"\"\"\n    if minutes < 0:\n        raise ValueError('minutes must be >= 0')\

### Tool result
[stderr] Traceback (most recent call last):
[stderr]   File "<stdin>", line 3, in <module>
[stderr]   File "/usr/local/lib/python3.12/pathlib.py", line 1047, in write_text
[stderr]     with self.open(mode='w', encoding=encoding, errors=errors, newline=newline) as f:
[stderr]          ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
[stderr]   File "/usr/local/lib/python3.12/pathlib.py", line 1013, in open
[stderr]     return io.open(self, mode, buffering, encoding, errors, newline)
[stderr]            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
[stderr] FileNotFoundError: [Errno 2] No such file or directory: '/workspace/bookings/timeutil.py'

Exit code: 1
[Command failed with exit code 1]

### Tool call: execute
{"command": "python - <<'PY'\nfrom pathlib import Path\nbase=Path('workspace/bookings')\n(base/'timeutil.py').write_text('''\"\"\"Time helpers for the bookings package.\"\"\"\nimport re\n\n\ndef parse_duration(text):\n    \"\"\"Convert a duration string to whole minutes (int).\n\n    Supported formats:\n      \"1h30m\" -> 90\n      \"2h\"    -> 120\n      \"45m\"   -> 45\n      \"1:30\"  -> 90\n      \"45\"    -> 45    # a bare number means minutes\n    Surrounding whitespace is ignored.\n    \"\"\"\n    text = text.strip()\n    patterns = [\n        (r\"(\\\\d+)h(\\\\d+)m\", lambda m: int(m.group(1)) * 60 + int(m.group(2))),\n        (r\"(\\\\d+)h\", lambda m: int(m.group(1)) * 60),\n        (r\"(\\\\d+)m\", lambda m: int(m.group(1))),\n        (r\"(\\\\d+):(\\\\d+)\", lambda m: int(m.group(1)) * 60 + int(m.group(2))),\n        (r\"(\\\\d+)\", lambda m: int(m.group(1))),\n    ]\n    for pattern, parser in patterns:\n        m = re.fullmatch(pattern, text)\n        if m:\n            return parser(m)\n    raise ValueError(f\"bad duration: {text!r}\")\n''')\n(base/'billing.py').write_text('''\"\"\"Billing rules.\"\"\"\n\n\ndef billable_blocks(minutes, block=15):\n    \"\"\"Number of billable blocks for a booking.\n\n    A partial block is billed as a full block (always round UP):\n    16 minutes with 15-minute blocks -> 2 blocks.\n    `minutes` must be >= 0; a negative value raises ValueError.\n    \"\"\"\n    if minutes < 0:\n        raise ValueError('minutes must be >= 0')\n

### Tool result

==================================== ERRORS ====================================
______________ ERROR collecting workspace/tests/test_bookings.py _______________
ImportError while importing test module '/tmp/lab-code-eval-cu4ce2th/workspace/tests/test_bookings.py'.
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
1 error in 0.21s

Exit code: 2
[Command failed with exit code 2]

### Tool call: execute
{"command": "PYTHONPATH=workspace python -m pytest -q workspace/tests", "timeout": 120}

### Tool result
...                                                                      [100%]
3 passed in 0.04s

[Command succeeded with exit code 0]

### Assistant
Updated `workspace/bookings/billing.py`, `workspace/bookings/schedule.py`, and `workspace/bookings/timeutil.py` to match the docstrings: added proper rounding/validation for billing, fixed `add_slot` to avoid shared mutable state, and expanded duration parsing to all documented formats. I also verified the test suite passes.