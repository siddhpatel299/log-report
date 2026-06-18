# log-report

A small Terminal-Bench 3 (Harbor) task. The agent gets an Apache-style access log and has to summarize it into a short JSON report.

## The task

Read `/app/access.log` and write `/app/report.json` with three fields:

- `total_requests`: number of request lines (blank lines don't count)
- `unique_ips`: number of distinct client IPs (the first field on each line)
- `top_path`: the path that was requested most often

The exact prompt the agent sees is in `instruction.md`.

## Layout

- `task.toml`: manifest with the artifacts, resource limits, separate-mode verifier, and metadata
- `instruction.md`: the prompt handed to the agent (the only thing it sees)
- `environment/`: the agent container (`Dockerfile`) plus the input `access.log`
- `solution/solve.sh`: reference solution the oracle agent runs
- `tests/`: the verifier. `test.sh` runs pytest and `test_output.py` checks the three values

## Environment

Pinned to `python:3.13-slim-bookworm`. Only the input log is copied into the image. No solution or tests are baked in, and nothing needs the network at runtime.

## How it's verified

The verifier runs in its own container (`environment_mode = "separate"`). After the agent stops, `tests/test.sh` runs `tests/test_output.py` and writes `reward.txt` and `ctrf.json` to `/logs/verifier/`. The tests load `/app/report.json` and check the exact values for this log (6, 3, and `/index.html`), one assertion per numbered criterion in `instruction.md`, so an empty, malformed, or wrong report scores 0.

## Run it

```
harbor run -p . --agent oracle   # reference solution, reward 1.0
harbor run -p . --agent nop      # do-nothing agent, reward 0.0
```

Calibrated locally: oracle = 1.0, nop = 0.0.
