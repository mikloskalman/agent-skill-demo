---
description: 'Add pytest tests for the open file (or the file the user names)'
agent: agent
tools: ['search/codebase', 'edit/editFiles', 'execute/runInTerminal', 'execute/runTests', 'read/problems']
---

# /add-tests

Write thorough pytest tests for a Python file. By default, test the file
the user currently has open; if they name a different file in their
message, test that instead.

Follow these four steps in order. Do not skip the plan step — printing
the plan first is what makes the result predictable.

## Step 1 — Plan

Read the target file. In chat, list:

- Every public function and class you intend to test.
- For each, the cases you'll cover:
  - Happy path (one test).
  - Edge cases (empty input, missing file, malformed data, boundary values).
  - Error cases (which exceptions should be raised and when).

Present the plan as a short bulleted list. Do not ask for confirmation —
just print the plan, then continue to step 2.

## Step 2 — Write the tests

- Put tests in a `tests/` folder next to the file under test. Create
  the folder if it doesn't exist.
- Name the file `test_<module>.py`.
- Use plain `def test_...` functions (no unittest classes).
- Use the `tmp_path` fixture for anything that touches the filesystem.
  **Never** write to the real working directory in tests.
- Use `monkeypatch` to fake `sys.argv`, environment variables, current
  working directory, or `datetime.now`.
- One assertion per test where reasonable. Use descriptive test names
  rather than comments.
- Add a short docstring on each test only if the name alone isn't clear.

## Step 3 — Run the tests

Run `pytest -q` from the project root.

If a test fails, decide carefully:

- Is the **test** wrong? Fix the test.
- Is the **production code** wrong? **Stop.** Tell the user what's broken
  and ask whether you should fix the code. Do not silently change
  production code from inside a test-writing task.

## Step 4 — Report

Finish with a short summary block:

- Number of tests added.
- Coverage gaps you intentionally skipped (and why).
- The exact command to re-run them: `pytest -q`.

## Variables

- Target file: `${file}` if available, otherwise infer from the user's
  message.
- Selected text (if any): `${selection}` — if the user has a specific
  function highlighted, focus on that function first.
