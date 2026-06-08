# Day 3 — `uv` & Reproducible Dependencies

> **Goal of the day:** keep Python dependencies *reproducible* — the exact same
> versions on every machine, so "it works on my laptop" never becomes a problem.

---

## 1. The core idea: two files, two jobs

| File | Nickname | What it holds | Who writes it |
|------|----------|---------------|----------------|
| `requirements.in` | the **wishlist** | only the packages *I* care about, with loose rules | a human |
| `requirements.txt` | the **lockfile** | *every* package, pinned to one exact version | the tool (`uv`) |

Analogy: `.in` = "I want pizza." `.txt` = the full itemized receipt with every
ingredient and exact quantity.

---

## 2. What `uv` is

`uv` is a fast tool for managing Python packages and environments (a modern,
quicker alternative to plain `pip` + `pip-tools`). The key command this day:

```bash
uv pip compile requirements.in -o requirements.txt
```

- `compile` = turn the loose wishlist into a fully pinned lockfile
- `-o` = **o**utput to this file

`uv` reads my loose wishes, solves "which exact versions all work together,"
and writes the pinned result.

---

## 3. Two kinds of dependencies

- **Top-level** = packages *I* listed myself (e.g. scikit-learn, mlflow, pandas, numpy).
- **Transitive** = packages *those* need to work (e.g. scikit-learn pulls in scipy).

I once installed **4** packages but `pip freeze` showed **17** — the extra 13
were transitive. A good lockfile pins **both** kinds.

---

## 4. Version constraints

A constraint is a rule about which version is allowed:

| Constraint | Meaning |
|------------|---------|
| `numpy` | any version (too loose — avoid in a strict spec) |
| `numpy>=1.20` | 1.20 or newer (a "floor") |
| `numpy==2.1.3` | **exactly** this version (pinned) |

- **Satisfiable** = a version that *actually exists on PyPI* fits the rule.
  `numpy>=1.20` ✅ satisfiable. `mlflow>=100.0` ❌ — no such version exists.
- **Pinned** = locked with `==` to one exact version. This is what makes a
  lockfile reproducible.

> In the `.in` wishlist, loose floors (`>=`) are fine.
> The **pinning (`==`) happens automatically** when you compile to `.txt`.

---

## 5. PyPI — the source of truth

**PyPI** (Python Package Index, **pypi.org**) is the online store every Python
package is downloaded from. So two questions always reduce to "check PyPI":

- *Is the name correct?* → Does PyPI have a package by that exact name?
  - ⚠️ Classic trap: you **install** `scikit-learn` but **import** `sklearn`.
    `sklearn` is NOT the right name to put in a requirements file.
- *Is the version satisfiable?* → Does PyPI actually host a version that fits?
  - Use a package's **"Release history"** tab on pypi.org to find real versions.

---

## 6. My diagnosis checklist (for fixing a bad `requirements.in`)

Run each line through these four questions:

1. **Count** — exactly the required number of top-level packages? (none missing/extra)
2. **Names** — every name spelled the way PyPI knows it? (watch `sklearn` vs `scikit-learn`)
3. **Constraints** — does *every* package carry a version rule? (no bare names)
4. **Satisfiable** — does a real version exist for each constraint? (no `>=100.0`)

---

## 7. The debugging loop

The tool itself is the grader:

```text
read file → fix what looks wrong → run compile
   → if ERROR: read the message (it names the bad package/version) → fix → re-run
   → if SUCCESS: open requirements.txt and verify
```

"`uv` will choke on it" just means **uv fails and stops with an error** instead
of guessing. The error message is a gift — it points right at the problem.

---

## 8. How I know I'm done

After a successful compile:

```bash
cat requirements.txt
```

✅ all my top-level packages appear, each pinned with `==`
✅ there are **extra** lines too — the transitive deps `uv` resolved
❌ if I see *only* my top-level packages, something's wrong

---

## Quick command reference

```bash
cd <project-dir>
cat requirements.in                                  # read the wishlist
# ...edit it to fix names / constraints...
uv pip compile requirements.in -o requirements.txt   # compile to lockfile
cat requirements.txt                                 # verify pins + transitives
```

---

### Production vs sandbox judgment
Pinning everything matters most in **production / shared** setups — it's what
guarantees a teammate's machine matches mine exactly. Reproducibility is the
whole point of the lockfile.
