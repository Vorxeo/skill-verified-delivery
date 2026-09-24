---
name: verified-delivery
description: >-
  Use at the end of every coding chunk before claiming done. Distinguishes
  Verified (command ran + output read from result file) from Written (exists on
  disk). Chains with karpathy-method step 6. Never claim green without seeing N
  passed / exit code in the result file.
---
# Verified Delivery

Prove behaviour, not appearance. A file on disk is not a passing test. A green claim without a result file is a lie.

## Core distinction

| State | Meaning | Allowed claim |
|-------|---------|---------------|
| **Written** | Artifact exists on disk (code, test, config) | "File X written" |
| **Verified** | Command ran; stdout/stderr/exit code read from a **result file** | "N passed, exit 0" |

Never upgrade Written → Verified by inference. Never say "tests pass" because the test file looks correct. Never trust a pipe, `echo`, or terminal scrollback alone — redirect to a file, then read that file.

## When this skill applies

- End of every coding chunk (implement, fix, refactor, migrate).
- Before any handoff to `handoff-faber-rigor` (Faber packet must cite Verified evidence).
- Before claiming bench progress (`bench-loop`) or gate hardening (`adversarial-qa`, `fail-closed-review`).
- Chains with **karpathy-method** step 6: measure after change; measurement must be Verified.

## Rules (enforceable)

1. **Redirect, then Read.** Run tests/commands with output redirected to a result file. Read the file with the Read tool (or equivalent). Do not summarize from memory of a pipe.
2. **Cite the numbers.** Claim format: `pytest: 47 passed, 0 failed, exit 0` — only if those strings appear in the result file. If the file shows `ERROR` / non-zero exit, claim failure.
3. **No green without N.** Forbidden phrases without a result-file cite: "all good", "tests pass", "looks fine", "should work", "verified locally".
4. **Exit code is evidence.** Capture `$?` (or process exit) into the same result file. A truncated log without exit code is incomplete — re-run.
5. **One result file per claim.** Name it clearly, e.g. `/tmp/verified-delivery-pytest.txt`. Prefer unique names when iterating.
6. **Written-only is explicit.** If you only wrote code and did not run checks, say: `Status: Written, not Verified. Pending: <command>.`
7. **Don't trust echo.** Scripts that `echo PASS` without running the suite are not Verified. The result file must contain the tool's own summary line (pytest, cargo test, go test, etc.).

## Failure modes (named)

| Failure mode | What it looks like | Fix |
|--------------|-------------------|-----|
| **Appearance-as-proof** | "I wrote the test, so it passes" | Run + redirect + Read |
| **Pipe amnesia** | Claimed pass from scrolled terminal, file never read | Always redirect; Read the file |
| **Echo-green** | Wrapper prints success regardless of child exit | Capture child exit; forbid fake PASS |
| **Partial log** | Truncated output, no "N passed" line | Re-run with full capture |
| **Wrong suite** | Ran unit tests, claimed integration | Name which suite; match claim to file |
| **SQLite-for-Postgres** | Green on SQLite, ship Postgres RLS claim | See `migration-and-data-safety`; wrong DB ≠ Verified for that control |
| **Cross-skill skip** | Claimed gate fixed without `adversarial-qa` matrix row | Delivery receipt must list attack checks or mark NOT tested |

## Procedure

```text
1. Identify the claim you want to make (behaviour, not file existence).
2. Choose the command that would falsify the claim if broken.
3. Run:  <command> > /tmp/vd-<slug>.txt 2>&1; echo EXIT:$? >> /tmp/vd-<slug>.txt
4. Read /tmp/vd-<slug>.txt end-to-end (or enough to see summary + EXIT).
5. Extract: N passed / failed / skipped, EXIT code, any ERROR.
6. Fill the delivery receipt (below). Attach or quote the receipt in the handoff.
7. If EXIT != 0 or failures > 0: Status = Failed. Fix before next claim.
```

## Delivery receipt template

Copy and fill. Do not invent fields.

```text
### Delivery receipt
- Claim: <one sentence behaviour>
- Status: Verified | Written | Failed
- Command: <exact command>
- Result file: <absolute path>
- Evidence (from file): <e.g. "47 passed, 0 failed"> / EXIT:<n>
- Surfaces touched: <paths or APIs>
- NOT tested: <list>
- Cross-skills: karpathy-method step 6 | adversarial-qa | migration-and-data-safety | none
- Risks left open: <list or "none known">
```

## Minimal good examples

**Verified**

```text
Status: Verified
Command: pytest tests/authz -q > /tmp/vd-authz.txt 2>&1; echo EXIT:$? >> /tmp/vd-authz.txt
Evidence: "12 passed in 0.41s" / EXIT:0
```

**Written only**

```text
Status: Written, not Verified
Pending: pytest tests/authz -q  (result file required before green claim)
```

**Failed (honest)**

```text
Status: Failed
Evidence: "1 failed, 11 passed" / EXIT:1
Next: fix failing test, re-run Verified loop
```

## Anti-patterns

- Claiming Verified because CI "usually" passes.
- Pasting only the last line of a log without the summary/`EXIT`.
- Re-using an old result file after editing code (stale evidence).
- Bundling unrelated commands so a pass in A hides a fail in B — one claim, one result file, one suite.

## Interaction with other skills

- **karpathy-method**: step 6 measurement must be Verified delivery, not vibes.
- **code-that-holds**: correctness claims require this receipt.
- **fail-closed-review** / **adversarial-qa**: review findings and attack matrix rows need Verified commands when asserting "now refuse".
- **handoff-faber-rigor**: Faber packet without Verified evidence → Rigor refuses review.
- **bench-loop**: bench % must be read from output file; inventing metrics is Overclaim (treat as 0% until Verified).

## Checklist before "done"

- [ ] Result file exists and was Read this turn
- [ ] Claim matches exact numbers / EXIT in that file
- [ ] NOT tested listed
- [ ] No "should / looks / probably" in the status line
