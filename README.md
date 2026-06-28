# TDD Aircon Settings with GitHub Actions using ZOMBIES

This tutorial demonstrates **Test-Driven Development (TDD)** using the **ZOMBIES** concept and then connects the project to **GitHub Actions** so tests run automatically on every push.

ZOMBIES is a simple guide for deciding what test to write next:

- **Z** = Zero
- **O** = One
- **M** = Many
- **B** = Boundaries
- **I** = Interfaces
- **E** = Exceptions
- **S** = Simple scenarios, simple solutions

The idea is to grow the code in very small steps. Start with the simplest case, write a failing test, make it pass with the smallest possible code, and then continue.[web:43][web:124][web:45][web:131]

This tutorial uses a single production function:

```python
def aircon_settings() -> str:
    return "Mode: COOL, Temp: 24°C, Fan: AUTO"
```

---

## What is GitHub Actions?

GitHub Actions is GitHub’s built-in automation platform. It can automatically run tasks such as tests, builds, linting, packaging, and deployment whenever something happens in your repository, such as a push or a pull request.

In this tutorial, GitHub Actions is used as a simple **continuous integration (CI)** tool. That means every time code is pushed to GitHub, GitHub automatically runs the test suite and reports whether it passed or failed.

Useful terms:

- **Workflow**: a YAML file that defines the automation
- **Event**: what triggers the workflow, such as `push`
- **Job**: a group of steps that runs on a machine
- **Step**: one command or action inside a job
- **Runner**: the machine that executes the workflow

---

## What you need

Before starting, make sure you have:

- A GitHub account
- Git installed
- Python 3 installed
- A terminal or command prompt
- A code editor such as VS Code

---

## Step 1: Create the GitHub repository

1. Sign in to GitHub.
2. Click the `+` icon at the top right.
3. Choose `New repository`.
4. Enter the repository name:

   ```text
   tdd-aircon-settings
   ```

5. Optionally add a description.
6. Choose `Public`.
7. Check `Add a README file`.
8. Click `Create repository`.

---

## Step 2: Clone the repository locally

On the repository page:

1. Click the green `Code` button.
2. Copy the HTTPS URL.

Example:

```text
https://github.com/your-username/tdd-aircon-settings.git
```

Open a terminal and run:

```bash
git clone https://github.com/your-username/tdd-aircon-settings.git
cd tdd-aircon-settings
```

---

## Step 3: Install pytest and create folders

Create a file named `requirements.txt`:

```text
pytest
```

Install it with:

```bash
pip install -r requirements.txt
```

If `pip` does not work on your computer, try:

```bash
python -m pip install -r requirements.txt
```

Create the folders for source code and tests:

```bash
mkdir src
mkdir tests
```

---

## Step 4: Understand ZOMBIES before coding

ZOMBIES helps you choose test cases in a sensible order.

### Z — Zero

What should happen in the most basic or default case?

### O — One

What should happen for one simple valid case?

### M — Many

What should happen when behavior is checked repeatedly or across more than one case?

### B — Boundaries

What rules define the edges of acceptable behavior?

### I — Interfaces

What should the function return? What should the public API look like?

### E — Exceptions

How should invalid or unexpected situations be handled?

### S — Simple scenarios, simple solutions

Always choose the smallest next step. Avoid building extra logic before tests demand it.[web:43][web:124][web:45][web:131]

For this tutorial, we will apply ZOMBIES to one simple function that returns default aircon settings.

---

## Step 5: Z + O — start with the simplest requirement

### Requirement

The system should return a default aircon setting string.

### Expected output

```text
Mode: COOL, Temp: 24°C, Fan: AUTO
```

This is both the **Zero** and **One** case for this tiny exercise:

- **Zero**: there are no inputs
- **One**: there is one default expected output

---

## Step 6: RED — write the first failing test

Create `tests/test_aircon_exact.py`:

```python
from src.aircon import aircon_settings


def test_aircon_returns_default_configuration():
    assert aircon_settings() == "Mode: COOL, Temp: 24°C, Fan: AUTO"
```

Run the test:

```bash
pytest
```

If `pytest` is not found, try:

```bash
python -m pytest
```

This should fail because the implementation does not exist yet.

This is the **RED** phase.

---

## Step 7: GREEN — write the smallest code to pass

Create `src/aircon.py`:

```python
def aircon_settings() -> str:
    return "Mode: COOL, Temp: 24°C, Fan: AUTO"
```

Run the tests again:

```bash
pytest
```

Now the test should pass.

This is the **GREEN** phase.

---

## Step 8: S — keep the solution simple

Do not add extra parameters, classes, or parsing logic yet.

At this stage, the simplest correct implementation is:

```python
def aircon_settings() -> str:
    return "Mode: COOL, Temp: 24°C, Fan: AUTO"
```

That is exactly what **Simple scenarios, simple solutions** means.

---

## Step 9: I — define the interface clearly

Now add tests that make the public interface more explicit.

### New requirement

The function should:

- return a string
- expose a readable format with `Mode:`, `Temp:`, and `Fan:`

Update `tests/test_aircon_exact.py`:

```python
from src.aircon import aircon_settings


def test_aircon_returns_default_configuration():
    assert aircon_settings() == "Mode: COOL, Temp: 24°C, Fan: AUTO"


def test_aircon_settings_type_is_str():
    result = aircon_settings()
    assert isinstance(result, str)
```

Create `tests/test_aircon_structure.py`:

```python
from src.aircon import aircon_settings


def test_aircon_settings_contains_mode_temp_fan_labels():
    s = aircon_settings()
    assert "Mode:" in s
    assert "Temp:" in s
    assert "Fan:" in s
```

Run the tests:

```bash
pytest
```

These tests define the public shape of the function more clearly.

---

## Step 10: M — test repeatability

Now apply **Many**.

### New requirement

The function should behave consistently across repeated calls.

Add this to `tests/test_aircon_structure.py`:

```python
from src.aircon import aircon_settings


def test_aircon_settings_is_stable_across_calls():
    assert aircon_settings() == aircon_settings()


def test_aircon_settings_contains_mode_temp_fan_labels():
    s = aircon_settings()
    assert "Mode:" in s
    assert "Temp:" in s
    assert "Fan:" in s
```

Run:

```bash
pytest
```

This checks that the function is stable and predictable.

---

## Step 11: B — add boundary thinking

Now apply **Boundaries**.

### New requirement

The temperature in the returned string should stay inside a reasonable comfort range.

Create `tests/test_aircon_semantics.py`:

```python
from src.aircon import aircon_settings


def test_default_temp_is_in_comfort_range():
    s = aircon_settings()
    temp_part = s.split("Temp:").split(",").strip()[5]
    temp_value = int(temp_part.replace("°C", ""))
    assert 22 <= temp_value <= 26
```

Run:

```bash
pytest
```

This test checks a boundary rule around the temperature value.

---

## Step 12: E — think about exceptions

For this very small example, the function has **no input**, so there is not much exception handling yet.

That itself is a useful lesson: not every ZOMBIES letter will always produce a big code change. Sometimes the correct conclusion is:

- there is no invalid input yet
- exception tests can be postponed
- keep the solution simple until the design grows

If later you change the function to accept user input such as a temperature or mode, that is where exception tests would become more important.

---

## Step 13: REFACTOR — clean up while staying green

Now that you have several tests, do a small cleanup pass.

### Final production code

`src/aircon.py`

```python
def aircon_settings() -> str:
    return "Mode: COOL, Temp: 24°C, Fan: AUTO"
```

### Final test files

- `tests/test_aircon_exact.py`
- `tests/test_aircon_structure.py`
- `tests/test_aircon_semantics.py`

Refactoring can include:

- improving file names
- grouping related tests
- removing duplication
- improving readability

Run tests again after refactoring:

```bash
pytest
```

If tests still pass, your refactoring is safe.

---

## Step 14: Add GitHub Actions after local TDD works

Once the local ZOMBIES-driven TDD flow is working, automate it with GitHub Actions.

Create the workflow folder:

```bash
mkdir -p .github/workflows
```

Create `.github/workflows/tdd-aircon.yml`:

```yaml
name: TDD Aircon Settings CI

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Run pytest suites
        run: pytest
```

### What this workflow does

- Runs on every push to `main`
- Also runs on pull requests targeting `main`
- Starts a fresh Ubuntu runner
- Downloads your repository
- Installs Python
- Installs project dependencies
- Runs `pytest`

This means the same tests you run locally are also verified automatically in GitHub.

---

## Step 15: Commit and push the project

Check the file status:

```bash
git status
```

Add everything:

```bash
git add .
```

Commit:

```bash
git commit -m "Add ZOMBIES TDD aircon settings project with GitHub Actions"
```

Push:

```bash
git push origin main
```

---

## Step 16: View the GitHub Actions run

After pushing:

1. Open the repository on GitHub
2. Click the `Actions` tab
3. Open the workflow named `TDD Aircon Settings CI`
4. Open the `test` job
5. Review each step

If everything is correct, you should see a green check mark.

---

## Project structure

```text
tdd-aircon-settings/
├── .github/
│   └── workflows/
│       └── tdd-aircon.yml
├── src/
│   └── aircon.py
├── tests/
│   ├── test_aircon_exact.py
│   ├── test_aircon_structure.py
│   └── test_aircon_semantics.py
├── requirements.txt
└── README.md
```

---

## Full file contents

### `requirements.txt`

```text
pytest
```

### `src/aircon.py`

```python
def aircon_settings() -> str:
    return "Mode: COOL, Temp: 24°C, Fan: AUTO"
```

### `tests/test_aircon_exact.py`

```python
from src.aircon import aircon_settings


def test_aircon_returns_default_configuration():
    assert aircon_settings() == "Mode: COOL, Temp: 24°C, Fan: AUTO"


def test_aircon_settings_type_is_str():
    result = aircon_settings()
    assert isinstance(result, str)
```

### `tests/test_aircon_structure.py`

```python
from src.aircon import aircon_settings


def test_aircon_settings_is_stable_across_calls():
    assert aircon_settings() == aircon_settings()


def test_aircon_settings_contains_mode_temp_fan_labels():
    s = aircon_settings()
    assert "Mode:" in s
    assert "Temp:" in s
    assert "Fan:" in s
```

### `tests/test_aircon_semantics.py`

```python
from src.aircon import aircon_settings


def test_default_temp_is_in_comfort_range():
    s = aircon_settings()
    temp_part = s.split("Temp:").split(",").strip()[5]
    temp_value = int(temp_part.replace("°C", ""))
    assert 22 <= temp_value <= 26
```

### `.github/workflows/tdd-aircon.yml`

```yaml
name: TDD Aircon Settings CI

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Run pytest suites
        run: pytest
```

---

## ZOMBIES recap

Use this checklist whenever you do TDD:

- **Zero**: what happens in the empty or default case?
- **One**: what is the simplest valid case?
- **Many**: what happens repeatedly or across more than one case?
- **Boundaries**: what are the edge conditions?
- **Interfaces**: what should the public API look like?
- **Exceptions**: what should happen for invalid situations?
- **Simple**: solve only the next small problem, not the whole future design

That is the core learning goal of this workshop.
