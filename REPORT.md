# Assignment 4 Report

Name: Abraham Eapen
Commit graded: the latest commit on main (agent code at 31a3239; runA and runB were recorded on it)

Answer from your own traces. Give step numbers as `attempt 1, step 4`. Keep to the word
limits; a short answer with the right evidence earns full points.

## 1. The runs

Run `python -m agent summary` and paste its table here exactly as printed.

| Run | Tools | Judge | Stopped (per attempt) | Model calls | Tool calls | Input tokens | Output tokens | Cost | Judge verdict |
| --- | --- | --- | --- | ---: | ---: | ---: | ---: | ---: | --- |
| runA | local | after | 1: done | 6 | 7 | 9817 | 685 | $0.0013 | 1: PASS |
| runB | mcp | reflexion | 1: done | 5 | 6 | 7393 | 452 | $0.0010 | 1: PASS |

## 2. A rejection is feedback (Run A, at most 100 words)

Which step proposed the edit you rejected, and what was your reason? What did the agent
do next, and at which step? Did it follow your reason?

Attempt 1, step 3 proposed one edit to `discounts.py`: a `code.upper()` lookup and a
`today > last_day` check. I rejected it with the reason "Add a comment that names the
spec rule you are following." At attempt 1, step 4 the agent proposed the same edit,
now with `# Spec rule 4: codes remain valid through their inclusive last day.` above
the expiry check. It followed the reason, and I approved it. It named only
rule 4, not rule 2. (This is the second recording of runA; see LEDGER Entry 6.)

## 3. The visible tests and the judge (Run A, at most 120 words)

What did the last `run_tests` in runA report, and what did the judge say after the run?
If the judge failed, which rule in `spec.md` did the code break, at which step did the
agent stop, and why did it stop there?

The last `run_tests`, at attempt 1, step 5, reported "PASSED: 4 of 4 tests passed." The
agent stopped at step 6 with status done (it replied without a tool call). The judge
then reported "PASSED: 6 of 6 tests passed.", so no spec rule was broken. The visible
tests check only rules 1 to 3. Rule 4 (expiry) is checked only by the judge, and the
agent got it right anyway because it read `spec.md` at step 2 and fixed rules 2 and 4
in the same edit.

## 4. The lessons (Run B, at most 150 words)

Quote every lesson runB wrote to `NOTES.md`. For each one, say whether it names the real
cause, points at evidence, and says what to do next. Did attempt 2 follow it? Give the
step that shows it. If attempt 1 passed, say which step shows why no lesson was needed.

runB wrote no lessons: attempt 1 passed the judge ("PASSED: 6 of 6 tests passed."), so
there was no attempt 2 and `NOTES.md` is empty. Attempt 1, step 3 shows why no lesson
was needed. Its one edit added both `normalized_code = code.upper()` (rule 2) and
`if today > last_day: raise ValueError(...)` (rule 4, the rule only the judge checks),
after the agent read `spec.md` at step 2. Step 4's `run_tests` then passed 4 of 4, and
the agent stopped at step 5.

## 5. Memory grows (Run A, at most 60 words)

Give `input_tokens` from the first and the last tool event in `runs/runA/trace.jsonl`,
with their step numbers. Why does the number grow from step to step?

The first tool event (attempt 1, step 1, list_files) has 684 `input_tokens`. The last
(attempt 1, step 5, run_tests) has 2277. It grows because every model call resends the
whole history: the task, every earlier reply, and every tool result, including the three
files read at step 2 and the rejected edit.

## 6. MCP (at most 120 words)

Quote one of your tool descriptions exactly as `python -m agent check-mcp` showed it in
the tools/list reply. Then name one risk of installing someone else's MCP server, and say
which part of your agent would stop it, or that nothing in your agent would.

`run_tests`, from the tools/list reply:

    "Run the project's unit tests in its tests/ folder and summarize the result.\n\n    Takes no inputs. Returns a stable, short report listing failing tests, or the number\n    that passed. A failing test is reported normally so you can inspect and fix the code.\n    The test process has a 20-second limit; if it times out, inspect the last change for\n    an endless loop. This tool only runs tests; it does not change project files.\n    "

Risk: a third-party server runs its own code. A malicious `read_file` could read
`.env` and send my API key away, and its description could tell the model to do this.
`gate_for` does refuse any tool name not in `GATES`. But `read_file` and `run_tests` are
"allow", so they run without asking, and `safe_path` only protects my own server's code.
Nothing in my agent would stop it.

## 7. The reviewer and the judge (at most 100 words)

What did your reviewer find in runB's final `discounts.py`? Did it find anything the
judge did not, or miss anything the judge caught? In the tool you used, was the
reviewer's read-only rule a lock or a request, and how do you know?

The reviewer found all five rules met ("Verdict: PASS"). This agrees with the judge's
6 of 6, so it missed nothing the judge caught. It also listed issues the spec and the
judge do not cover: negative totals round the discount the wrong way, `code=None` raises
AttributeError, a `datetime` for `today` raises TypeError, and spaces are not trimmed.
In Claude Code the rule was a lock. The header
says `tools: Read, Grep, Glob`, and the subagent's transcript shows it was offered only
those (plus internal handback tools), with no Edit or Bash. It called Read 4 times.

## 8. Your agent and a vendor's agent (at most 120 words)

What did the fixer change, and what did `python -m agent judge vendor` report for the
visible tests and for the judge? Did it change anything in `tests/`? Compare this with
runA, where your own agent had the same task on the same project. Then name one thing you
could control in your own agent that the fixer file could not set, and one thing that was
easier with the vendor's agent.

The fixer added `code = code.upper()` before the lookup, using `sed` through Bash
rather than Edit. `judge vendor`: visible tests "PASSED: 4 of 4", judge "FAILED: 1 of
6" (`test_rule_4_code_is_rejected_after_its_last_day`). The only changed file is
`discounts.py`; nothing in `tests/` changed. It read `spec.md` and named the expiry bug,
but left it alone as outside "the smallest change". In runA, my agent fixed rules 2 and
4, and the judge passed 6 of 6. One control I had: `gate_for` refuses edits to `tests/`
in code. The fixer file can only ask, since its Bash tool can write anywhere. My agent
also always runs the judge. Easier with the vendor's agent: one file with no code, and
built-in shell and file tools.
