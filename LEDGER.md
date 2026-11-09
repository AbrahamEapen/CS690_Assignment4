# Provenance Ledger

Write one entry per reviewable change or experiment, when the work is done. Use exactly
the schema in HANDOUT.md. Put the prompts you typed into AI tools in `prompts/` and name
the file in the entry's `prompts` field.

For a run, most of the entry comes straight from its trace: what the agent did and in
what order, the commands that checked it, and the result. You add what only you know:
how you reviewed it, what you rejected, and what is still uncertain.

## Worked example (not graded; leave it here and add your entries under "My entries")

This shows the level of detail expected. The commit SHAs, dates, steps and numbers are
made up, and so is the practice run.

```
## Entry 2
artifact:  agent/loop.py, run_agent, at commit 9c3a71e
tool:      Claude Code, model Claude Sonnet 5.5, 2026-11-01
prompts:   asked for the loop body from the comment above run_agent;
           prompts/loop-01.md
review:    read every line against the comment. It returned as soon as is_stuck was
           true, which skipped the tool results for the other calls in the same turn;
           moved the stuck check after the for loop as the comment says. Rejected its
           suggestion to import a framework's message classes; kept the dicts from
           core.py
checks:    pytest tests/test_loop.py: 9 passed
evidence:  HANDOUT Part 3; the comment above run_agent, steps 4 and 5
risk:      no test of mine has three tool calls in one turn

## Entry 4
artifact:  runs/try1, a practice run, agent code at commit 4b9e2d1
tool:      my agent (python -m agent run try1), anthropic claude-haiku-5-5, 2026-11-02
prompts:   the default task and BASE_SYSTEM in agent/loop.py; prompts/try1-task.md
review:    read every event in trace.jsonl. At attempt 1, steps 4 and 6, the model
           copied old_text with the read_file line numbers, and edit_file refused both
           times. At step 7 it made an edit that changed nothing the tests check, and
           at step 8 the stuck rule stopped it. I approved the step 7 edit without
           reading it closely
checks:    python -m agent check-trace try1: complete
evidence:  HANDOUT Part 1 (tool descriptions) and Part 2 (is_stuck)
risk:      one practice run; a better description may still not stop the copying
dataset:   fixture/ at commit 4b9e2d1, task "Make the tests pass."
result:    stopped stuck after 8 model calls, 14,880 input tokens, about $0.002;
           runs/try1/trace.jsonl
changed:   wrote "copy it without the line numbers" into the read_file and edit_file
           descriptions, and read every diff before approving
```

## My entries

## Entry 1
artifact:  Part 0 setup: .venv, requirements.txt, .gitignore, .env (not committed)
tool:      none for the setup itself; provider OpenAI, model gpt-5.6-luna (the default),
           2026-11-08
prompts:   none (no AI tool was used to find the prices)
review:    looked up the price myself on OpenAI's pricing page,
           https://openai.com/business/pricing/#api : input $0.10 per 1M tokens,
           output $0.50 per 1M tokens (standard, not cached or batch). Wrote
           PRICE_INPUT_PER_MTOK=0.10 and PRICE_OUTPUT_PER_MTOK=0.50 in .env. Checked
           that git status lists .gitignore and not .env
checks:    .venv/bin/python -m agent check-setup: "All checks passed." (Python 3.12.3,
           fixture/ and judge/ unchanged 8aaab9373de615a5, prices $0.1 in / $0.5 out,
           tool-calling ping: 143 tokens in, 13 out)
evidence:  HANDOUT.md Part 0; README.md Setup steps 1 to 5
risk:      prices change; these are the prices on 2026-11-08 and summary's cost is
           only as right as they are

## Entry 2
artifact:  agent/tools.py, Part 1 implementation, commit pending
tool:      GitHub Copilot, model/version not exposed, 2026-11-08
prompts:   requested completion of Assignment 4 and ongoing ledger updates;
           prompts/part1-tools.md
review:    implemented the tool contracts and descriptions. The first focused test
           exposed relative paths resolving against the process directory; changed
           safe_path to resolve from the selected workspace. Reviewed errors and output
           formatting against the comments above each function.
checks:    .venv/bin/python -m pytest tests/test_tools.py -q: 16 passed after the fix;
           initial run: 6 failed due to the path-resolution defect
evidence:  HANDOUT.md Part 1; tests/test_tools.py
risk:      hidden edge cases and cross-platform behavior remain unverified

## Entry 3
artifact:  agent/policy.py, Part 2 implementation, commit pending
tool:      GitHub Copilot, model/version not exposed, 2026-11-08
prompts:   requested completion of Assignment 4 and ongoing ledger updates;
           prompts/part2-policy.md
review:    implemented unknown-tool refusal, normalized tests/ path denial, ordered
           budget checks, and the same-failure-after-edit rule; compared the branches
           with the function contracts and public tests
checks:    .venv/bin/python -m pytest tests/test_policy.py -q: 7 passed
evidence:  HANDOUT.md Part 2; tests/test_policy.py
risk:      hidden path-normalization cases remain unverified

## Entry 4
artifact:  agent/loop.py, build_system_prompt and run_agent (Part 3), commit pending
tool:      GitHub Copilot, model/version not exposed, 2026-11-08
prompts:   requested completion of Assignment 4 and ongoing ledger updates;
           prompts/part3-loop.md (same Copilot session prompt as Entries 2 and 3)
review:    code reviewed in Claude Code (Entry 5) against each numbered step of the
           comment above run_agent: budget check before each model call, AIError ->
           "error", deny/ask/allow branches, preview None runs without asking, the
           stuck check after all calls in a turn, one trace.stop per exit. Specs are
           filtered by gate_for(name, {}). No framework or SDK loop helper is used; the
           only model access is provider.next_turn. The deny branch called gate_for a
           second time to get the reason; changed it to use `why` (Entry 5)
checks:    .venv/bin/python -m pytest tests/test_loop.py -q: 9 passed
evidence:  HANDOUT.md Part 3; the comment above run_agent, steps 1 to 5
risk:      no public test has a turn with a denied call plus an approved edit; the
           real-model behavior is only seen in runA/runB

## Entry 5
artifact:  agent/reflexion.py (judge, write_lesson, reflexion), server.py, and a one-line
           cleanup in agent/loop.py; commit pending
tool:      Claude Code, model Claude Opus 5.5 (claude-opus-5-5), 2026-11-08
prompts:   asked to review the repo, complete Parts 0 to 8, ask before decisions, keep
           the ledger updated, and grade against RUBRIC.md; prompts/claude-code-session.md
review:    read the generated code against the comments in reflexion.py and the
           server.py docstring. Checked: notes are read fresh every attempt; an "error"
           run stops without judging; a newline goes in first if NOTES.md lacks a
           trailing one; the lesson is one line, at most 600 characters; mcp.run is only
           under __main__, so importing server.py starts nothing. I confirmed with the
           assistant that Parts 1 to 3 came from Copilot (Entries 2 to 4)
checks:    .venv/bin/python -m pytest -q: 40 passed (reflexion 5, server 3);
           .venv/bin/python -m agent check-mcp: all checks passed
evidence:  HANDOUT.md Parts 4 and 5; RUBRIC A4, A5, B5
risk:      hidden tests for reflexion edge cases (an AIError from write_lesson, a
           NOTES.md missing at the start) are only covered by reading the code

## Entry 6
artifact:  runs/runA (Run A, recorded), agent code at commit 31a3239; also the first,
           replaced recording kept unedited in evidence/runA-first-recording/
tool:      my agent (python -m agent run runA), openai gpt-5.6-luna, 2026-11-09 (UTC)
prompts:   the default task "Make the tests pass." and BASE_SYSTEM in agent/loop.py;
           prompts/runA-task.md
review:    read every event in runs/runA/trace.jsonl. Attempt 1: step 1 list_files;
           step 2 read discounts.py, spec.md and tests/test_discounts.py in one turn;
           step 3 proposed one edit (code.upper() lookup plus a today > last_day
           check). I rejected it with the reason "Add a comment that names the spec
           rule you are following." Step 4 retried the same edit with
           "# Spec rule 4: codes remain valid through their inclusive last day." added;
           I read that diff, checked the .upper() normalization, the inclusive
           today > last_day check and that tests/ was untouched, and approved it. Step 5
           run_tests passed; step 6 the model replied without a tool call.
           Disclosure: the first recording of runA (started 2026-11-09T00:11:54Z, same
           commit) did not follow HANDOUT Part 6 step 1. I approved its first edit
           (step 4) after only skimming it, instead of rejecting it. Its agent result
           was the same (done, judge PASS 6 of 6). I re-recorded runA once to follow the
           procedure, not to change the agent's result, and kept the first recording
           unedited at evidence/runA-first-recording/ (check-trace and replay pass on
           it). It was moved out of runs/ so that summary does not list two "runA" rows
checks:    python -m agent check-trace runA: complete, 9 events, 7 tool calls;
           python -m agent replay runA: matches its trace, 9 events;
           diff -r fixture/tests runs/runA/workspace/tests: no differences
evidence:  HANDOUT Part 6 Run A; RUBRIC B1, B2, B3
risk:      the visible tests never exercise rule 4; only the judge did. The rejection
           reason asked for a comment, so it says little about whether a rejection can
           steer the agent to a different fix
dataset:   fixture/ (fixture and judge hash 8aaab9373de615a5) at commit 31a3239, task
           "Make the tests pass."
result:    1 attempt, status done after 6 model calls and 7 tool calls; 9,817 input and
           685 output tokens, $0.0013; last run_tests (step 5) "PASSED: 4 of 4 tests
           passed."; judge "PASSED: 6 of 6 tests passed."; runs/runA/trace.jsonl
changed:   follow the recorded-run procedure exactly before answering the first
           approval prompt; read every diff in full before approving (the first
           recording's approval was a skim)

## Entry 7
artifact:  runs/runB (Run B, recorded) and NOTES.md, agent code at commit 31a3239
tool:      my agent (python -m agent run runB --tools mcp --judge), openai gpt-5.6-luna,
           tools served by server.py over MCP stdio, 2026-11-09 (UTC)
prompts:   the default task "Make the tests pass." and BASE_SYSTEM in agent/loop.py;
           prompts/runB-task.md
review:    read every event in runs/runB/trace.jsonl. Attempt 1: step 1 list_files;
           step 2 read discounts.py, spec.md and tests/test_discounts.py; step 3 one
           edit with code.upper() and a today > last_day check, which I approved after
           only skimming the diff; step 4 run_tests passed; step 5 the model replied
           without a tool call. The judge passed attempt 1, so no lesson was written and
           NOTES.md is still empty (0 bytes). The run never called run_tests before its
           edit
checks:    python -m agent check-trace runB: complete, 8 events, 6 tool calls;
           python -m agent replay runB: matches its trace, 8 events;
           diff -r fixture/tests runs/runB/workspace/tests: no differences
evidence:  HANDOUT Part 6 Run B; RUBRIC B1, B2, B4
risk:      my approval at step 3 was a skim, so the code was checked by the judge, not by
           me; the reviewer in Part 7 is a second check. Reflexion's lesson path was not
           exercised by a real model, only by tests/test_reflexion.py
dataset:   fixture/ (fixture and judge hash 8aaab9373de615a5) at commit 31a3239, task
           "Make the tests pass."
result:    1 of 2 allowed attempts used, status done after 5 model calls and 6 tool
           calls; 7,393 input and 452 output tokens, $0.0010; last run_tests (step 4)
           "PASSED: 4 of 4 tests passed."; judge attempt 1 "PASSED: 6 of 6 tests
           passed."; no lessons; runs/runB/trace.jsonl
changed:   read every diff in full before approving, as I did in the re-recorded runA

