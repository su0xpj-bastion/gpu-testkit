# gpu-testkit — agent guardrails

This repo is a test framework, so the framework's own tests are the product's tests.

## Code
- `from __future__ import annotations` on line 1 of every module.
- Module-level constants (`ENCODING`, `DEFAULT_TIMEOUT_S`, `MAX_*`). `MAX_ATTEMPTS`, never `MAX_RETRIES`.
- Files are read with `Path.open("r", encoding=ENCODING)`; JSON via `json.load`, TOML via `tomllib`.
- Bare `raise` to re-raise. Catch broad `Exception` only where a failure must be recorded instead of hiding
  the test's own failure (diagnostics collectors, NVML init), with a comment saying so.
- No pandas, no numpy in the package: numerics take anything with `.tolist()`.
- `mypy --strict` and `ruff` stay clean.

## Plugin rules
- `plugin.py` is glue. Decisions live in plain modules (`hardware`, `pool`, `classify`, `quarantine`,
  `diagnostics`, `results`, `report`) with their own unit tests.
- Anything the plugin does end to end gets a `pytester` test, including under `-n` (xdist) when it touches
  workers, locks or the result store.
- A failure is `infra` only with positive evidence. Never widen the infra rules to turn a red test green.
- Quarantine entries need an owner and an expiry. Never add an entry without both.

## Honesty
- Everything here runs under the fake backend. Don't claim real-GPU validation; the NVML backend is
  tested against a stubbed `pynvml`, and the `nvidia-smi` and dmesg parsers against fixtures written
  in the real formats.

## Before claiming done
`make lint && make test && make report`.
