### Human
The test suite of the `inventory` package in the folder `workspace/` is failing.

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
['/workspace/CHANGELOG.md', '/workspace/README.md', '/workspace/inventory/', '/workspace/tests/']

### Tool call: glob
{"path": "/workspace/inventory", "pattern": "**/*.py"}

### Tool call: glob
{"path": "/workspace/tests", "pattern": "**/*.py"}

### Tool call: read_file
{"file_path": "/workspace/CHANGELOG.md", "limit": 200}

### Tool call: read_file
{"file_path": "/workspace/README.md", "limit": 200}

### Tool result
['/workspace/inventory/__init__.py', '/workspace/inventory/export.py', '/workspace/inventory/pricing.py', '/workspace/inventory/report.py']

### Tool result
['/workspace/tests/test_report.py']

### Tool result
@@ lines 1-6 of 6 @@
# Changelog

## Unreleased

## 0.3.1
- initial import

### Tool result
@@ lines 1-5 of 5 @@
# inventory

Small package. Run the tests from the `workspace/` folder:

    cd workspace && python -m pytest tests -q

### Tool call: read_file
{"file_path": "/workspace/inventory/__init__.py", "limit": 200}

### Tool call: read_file
{"file_path": "/workspace/inventory/export.py", "limit": 200}

### Tool call: read_file
{"file_path": "/workspace/inventory/pricing.py", "limit": 200}

### Tool call: read_file
{"file_path": "/workspace/inventory/report.py", "limit": 400}

### Tool call: read_file
{"file_path": "/workspace/tests/test_report.py", "limit": 400}

### Tool result
@@ lines 1-1 @@
System reminder: File exists but has empty contents

### Tool result
@@ lines 1-12 of 12 @@
"""Export helpers."""
from .pricing import parse_price


def to_csv_row(item):
    """Format one item as a CSV row: name,price,qty (price with 2 decimals).

    A name that contains a comma or a double quote is wrapped in double quotes,
    and every double quote inside it is doubled (RFC 4180).
    """
    price = parse_price(item["price"])
    return f'{item["name"]},{price:.2f},{item["qty"]}'

### Tool result
@@ lines 1-25 of 25 @@
"""Price utilities for the inventory package."""
from decimal import Decimal


def parse_price(text):
    """Convert a price string to a Decimal.

    Supported formats:
      "$1,299.50" -> Decimal("1299.50")
      "  12 "     -> Decimal("12")
      "1299.5"    -> Decimal("1299.5")
      "(12.00)"   -> Decimal("-12.00")   # accounting style: parentheses mean negative
    """
    cleaned = text.strip().replace("$", "")
    return Decimal(cleaned)


def apply_discount(price, percent):
    """Return `price` after a percentage discount.

    The result is a Decimal rounded to 2 decimal places using the
    commercial rule "round half up" (0.005 -> 0.01).
    """
    discounted = price * (100 - percent) / 100
    return discounted.quantize(Decimal("0.01"))

### Tool result
@@ lines 1-25 of 25 @@
"""Inventory reports."""
from decimal import Decimal

from .pricing import apply_discount, parse_price


def total_value(items):
    """Total value of `items` after per-item discounts.

    Each item is a dict: {"name": str, "price": str, "qty": int, "discount": percent (optional)}.
    """
    total = Decimal("0")
    for item in items:
        price = parse_price(item["price"])
        price = apply_discount(price, item.get("discount", 0))
        total += price * item["qty"]
    return total


def low_stock(items, threshold=5):
    """Names of the items whose quantity is strictly below `threshold`.

    The names are sorted alphabetically, ignoring case.
    """
    return [item["name"] for item in items if item["qty"] <= threshold]

### Tool result
@@ lines 1-32 of 32 @@
from decimal import Decimal

from inventory.export import to_csv_row
from inventory.pricing import apply_discount
from inventory.report import low_stock, total_value


def test_total_value_plain():
    items = [{"name": "pen", "price": "$2.50", "qty": 4}]
    assert total_value(items) == Decimal("10.00")


def test_total_value_thousands_separator():
    items = [{"name": "laptop", "price": "$1,299.50", "qty": 2}]
    assert total_value(items) == Decimal("2599.00")


def test_discount_rounding_half_up():
    assert apply_discount(Decimal("10.05"), 10) == Decimal("9.05")


def test_total_value_with_discount():
    items = [{"name": "desk", "price": "100.00", "qty": 1, "discount": 15}]
    assert total_value(items) == Decimal("85.00")


def test_low_stock_basic():
    assert low_stock([{"name": "pen", "qty": 2}, {"name": "ink", "qty": 9}]) == ["pen"]


def test_csv_row_plain_name():
    assert to_csv_row({"name": "pen", "price": "2.5", "qty": 4}) == "pen,2.50,4"

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
_______________ ERROR collecting workspace/tests/test_report.py ________________
ImportError while importing test module '/tmp/lab-code-learn-7eist02y/workspace/tests/test_report.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
/usr/local/lib/python3.12/importlib/__init__.py:90: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
/lab/tasks/code-learn/workspace/tests/test_report.py:3: in <module>
    from inventory.export import to_csv_row
E   ModuleNotFoundError: No module named 'inventory'
=========================== short test summary info ============================
ERROR workspace/tests/test_report.py
!!!!!!!!!!!!!!!!!!!! Interrupted: 1 error during collection !!!!!!!!!!!!!!!!!!!!
1 error in 0.23s

Exit code: 2
[Command failed with exit code 2]

### Tool call: execute
{"command": "PYTHONPATH=workspace python -m pytest workspace/tests -q", "timeout": 120}

### Tool result
.FF...                                                                   [100%]
=================================== FAILURES ===================================
_____________________ test_total_value_thousands_separator _____________________

    def test_total_value_thousands_separator():
        items = [{"name": "laptop", "price": "$1,299.50", "qty": 2}]
>       assert total_value(items) == Decimal("2599.00")
               ^^^^^^^^^^^^^^^^^^

/lab/tasks/code-learn/workspace/tests/test_report.py:15: 
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 
workspace/inventory/report.py:14: in total_value
    price = parse_price(item["price"])
            ^^^^^^^^^^^^^^^^^^^^^^^^^^
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

text = '$1,299.50'

    def parse_price(text):
        """Convert a price string to a Decimal.
    
        Supported formats:
          "$1,299.50" -> Decimal("1299.50")
          "  12 "     -> Decimal("12")
          "1299.5"    -> Decimal("1299.5")
          "(12.00)"   -> Decimal("-12.00")   # accounting style: parentheses mean negative
        """
        cleaned = text.strip().replace("$", "")
>       return Decimal(cleaned)
               ^^^^^^^^^^^^^^^^
E       decimal.InvalidOperation: [<class 'decimal.ConversionSyntax'>]

workspace/inventory/pricing.py:15: InvalidOperation
________________________ test_discount_rounding_half_up ________________________

    def test_discount_round

### Tool call: execute
{"command": "python - <<'PY'\nfrom decimal import Decimal, ROUND_HALF_UP\nprint((Decimal('10.05')*Decimal('90')/Decimal('100')).quantize(Decimal('0.01')))\nprint((Decimal('10.05')*Decimal('90')/Decimal('100')).quantize(Decimal('0.01'), rounding=ROUND_HALF_UP))\nPY", "timeout": 120}

### Tool result
9.04
9.05

[Command succeeded with exit code 0]

### Tool call: write_file
{"file_path": "/workspace/inventory/pricing.py", "content": "\"\"\"Price utilities for the inventory package.\"\"\"\nfrom decimal import Decimal, ROUND_HALF_UP\n\n\ndef parse_price(text: str) -> Decimal:\n    \"\"\"Convert a price string to a Decimal.\n\n    Supported formats:\n      \"$1,299.50\" -> Decimal(\"1299.50\")\n      \"  12 \"     -> Decimal(\"12\")\n      \"1299.5\"    -> Decimal(\"1299.5\")\n      \"(12.00)\"   -> Decimal(\"-12.00\")   # accounting style: parentheses mean negative\n    \"\"\"\n    cleaned = text.strip().replace(\"$\", \"\").replace(\",\", \"\")\n    negative = cleaned.startswith(\"(\") and cleaned.endswith(\")\")\n    if negative:\n        cleaned = cleaned[1:-1]\n    value = Decimal(cleaned)\n    return -value if negative else value\n\n\ndef apply_discount(price: Decimal, percent: int) -> Decimal:\n    \"\"\"Return `price` after a percentage discount.\n\n    The result is a Decimal rounded to 2 decimal places using the\n    commercial rule \"round half up\" (0.005 -> 0.01).\n    \"\"\"\n    discounted = price * (Decimal(\"100\") - Decimal(percent)) / Decimal(\"100\")\n    return discounted.quantize(Decimal(\"0.01\"), rounding=ROUND_HALF_UP)\n"}

### Tool result
Updated file /workspace/inventory/pricing.py

### Tool call: write_file
{"file_path": "/workspace/inventory/export.py", "content": "\"\"\"Export helpers.\"\"\"\nfrom .pricing import parse_price\n\n\ndef to_csv_row(item: dict) -> str:\n    \"\"\"Format one item as a CSV row: name,price,qty (price with 2 decimals).\n\n    A name that contains a comma or a double quote is wrapped in double quotes,\n    and every double quote inside it is doubled (RFC 4180).\n    \"\"\"\n    name = item[\"name\"]\n    if \",\" in name or '\"' in name:\n        name = '\"' + name.replace('\"', '\"\"') + '\"'\n    price = parse_price(item[\"price\"])\n    return f'{name},{price:.2f},{item[\"qty\"]}'\n"}

### Tool result
Updated file /workspace/inventory/export.py

### Tool call: write_file
{"file_path": "/workspace/inventory/report.py", "content": "\"\"\"Inventory reports.\"\"\"\nfrom decimal import Decimal\n\nfrom .pricing import apply_discount, parse_price\n\n\ndef total_value(items: list[dict]) -> Decimal:\n    \"\"\"Total value of `items` after per-item discounts.\n\n    Each item is a dict: {\"name\": str, \"price\": str, \"qty\": int, \"discount\": percent (optional)}.\n    \"\"\"\n    total = Decimal(\"0\")\n    for item in items:\n        price = parse_price(item[\"price\"])\n        price = apply_discount(price, item.get(\"discount\", 0))\n        total += price * item[\"qty\"]\n    return total\n\n\ndef low_stock(items: list[dict], threshold: int = 5) -> list[str]:\n    \"\"\"Names of the items whose quantity is strictly below `threshold`.\n\n    The names are sorted alphabetically, ignoring case.\n    \"\"\"\n    return sorted(\n        [item[\"name\"] for item in items if item[\"qty\"] < threshold],\n        key=str.lower,\n    )\n"}

### Tool result
Updated file /workspace/inventory/report.py

### Tool call: write_file
{"file_path": "/workspace/tests/test_regressions.py", "content": "from decimal import Decimal\n\nfrom inventory.export import to_csv_row\nfrom inventory.pricing import apply_discount, parse_price\nfrom inventory.report import low_stock\n\n\ndef test_parse_price_thousands_and_accounting():\n    assert parse_price(\"$1,299.50\") == Decimal(\"1299.50\")\n    assert parse_price(\"(12.00)\") == Decimal(\"-12.00\")\n\n\ndef test_apply_discount_rounds_half_up():\n    assert apply_discount(Decimal(\"10.05\"), 10) == Decimal(\"9.05\")\n\n\ndef test_low_stock_sorted_and_strict_threshold():\n    items = [{\"name\": \"Ink\", \"qty\": 5}, {\"name\": \"pen\", \"qty\": 2}]\n    assert low_stock(items, threshold=5) == [\"pen\"]\n\n\ndef test_to_csv_row_quotes_problematic_names():\n    assert to_csv_row({\"name\": 'a,\"b', \"price\": \"2.5\", \"qty\": 4}) == '\"a,\"\"b\",2.50,4'\n"}

### Tool result
Updated file /workspace/tests/test_regressions.py

### Tool call: edit_file
{"file_path": "/workspace/CHANGELOG.md", "old_string": "## Unreleased\n", "new_string": "## Unreleased\n- fix(parse_price): support thousand separators and accounting-style negatives\n- fix(apply_discount): round half up to two decimal places\n- fix(low_stock): respect strict threshold and sort names case-insensitively\n- fix(to_csv_row): quote names that need CSV escaping\n"}

### Tool result
Successfully replaced 1 instance(s) of the string in '/workspace/CHANGELOG.md'

### Tool call: execute
{"command": "PYTHONPATH=workspace python -m pytest workspace/tests -q", "timeout": 120}

### Tool result
..........                                                               [100%]
10 passed in 0.10s

[Command succeeded with exit code 0]

### Assistant
Fixed the inventory package so the full test suite passes.

Changed:
- `workspace/inventory/pricing.py` — added support for հազար separators and accounting negatives in `parse_price`, and made `apply_discount` round half up.
- `workspace/inventory/export.py` — added CSV escaping for names and type annotations.
- `workspace/inventory/report.py` — made `low_stock` use a strict threshold and return case-insensitively sorted names; added type annotations.
- `workspace/tests/test_regressions.py` — added regression tests for the bugs I fixed.
- `workspace/CHANGELOG.md` — documented the fixes under `## Unreleased`.

