# Ask Mode Documentation Context (Non-Obvious)

## Counterintuitive Structure
- `src/api/` is a **Gradle project** (not a Java source tree you edit directly) — it generates the z/OS Connect WAR from `openapi.yaml`.
- `src/base/ims/cobol/IBTRAN.cbl` is not a regular IMS program — it is a 31-bit COBOL bridge to 64-bit JVM (`JAVAIOP(JAVA64)`).
- `src/base/ims/DBD/` and `src/base/ims/PSB/` contain HLASM assembler source, not COBOL — they are assembled and deployed to separate IMS control-block libraries (`DBDLOAD`/`PSBLOAD`).
- The frontend (`src/frontend/`) is pure vanilla JS/HTML with no build step locally; it gets packaged on z/OS by a custom DBB Groovy task.

## Routing Logic (CICS vs IMS)
- Customer IDs prefixed `C` → CICS path; numeric IDs without prefix → IMS path. This routing happens in the frontend (`parseCustomerId` logic) and in `config.js`.

## Configuration
- `.setup/config/config.yaml` uses Jinja2-like `{{ }}` template syntax rendered by the setup pipeline, not standard YAML anchors.
- Actual runtime secrets/ports live in `.setup/config/setenv.sh` (gitignored); the template is `.zdx.json.template`.

## Tests
- All tests in `tests/` are **integration tests** requiring a live z/OS system. There is no local unit test runner.
- `IMS_DISABLED=true` env var skips IMS tests in `run-all.sh` (useful when IMS is not provisioned).

## Documentation
- Full installation and architecture docs at https://ibm.github.io/Bank-of-Z/ (not in this repo's `docs/`).
