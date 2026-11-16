# AGENTS.md

This file provides guidance to agents when working with code in this repository.

## Output Convention

All generated documentation, reports, and diagrams (program docs, wiki pages, architecture diagrams, data dictionaries, refactor reports, optimization reports, etc.) **must be saved under the [`docs/`](docs/) folder**.

## Project Overview

Bank of Z is a hybrid mainframe banking application. Transactions route to **CICS** or **IMS** based on customer ID prefix. The API layer is **z/OS Connect** (Liberty/Gradle). The frontend is vanilla JS/HTML served by a **separate** Liberty instance.

## Architecture: Two Liberty Servers (Critical)

- `BAQ<APP_SHORT_NAME>` (z/OS Connect) — port **9080** HTTP / **9444** HTTPS — serves `/api`
- `FE<APP_SHORT_NAME>` (Frontend Liberty) — port **9081** HTTP / **9445** HTTPS — serves the frontend WAR

`src/frontend/config.js` auto-detects the protocol and constructs the API URL pointing at port 9444/9080. When running on Docker dev (port 3001) it uses relative `/api` instead.

## Source Layout

```
src/base/cics/cobol/    CICS COBOL programs (.cbl)
src/base/cics/copy/     CICS copybooks (.cpy)
src/base/ims/cobol/     IMS COBOL programs (.cbl)
src/base/ims/pli/       IMS PL/I programs (.pli)
src/base/ims/DBD/       IMS DBDs (assembled, deployType=DBDLOAD)
src/base/ims/PSB/       IMS PSBs (assembled, deployType=PSBLOAD)
src/base/batch/pli/     Batch PL/I programs (.pli)
src/base/batch/jcl/     Batch JCL
src/api/                z/OS Connect Gradle project (openapi.yaml → WAR)
src/frontend/           Vanilla JS/HTML frontend (packaged into WAR by VanillaFrontend DBB task)
```

## Build System: DBB (Dependency Based Build)

Build config: [`dbb-app.yaml`](dbb-app.yaml)

- COBOL compile parms default to `LIB`; `CICS` and `SQL` suffixes are **auto-appended** by DBB based on IS_CICS / IS_SQL flags — do not hardcode them in source CBL directives.
- `IBTRAN.cbl` (IMS Java bridge) requires special parms: `LP(32),JAVAIOP(JAVA64),DLL,RENT,PGMNAME(LONGMIXED)` — already in dbb-app.yaml; do not change without understanding the 31-bit→64-bit JVM bridge.
- IMS COBOL batch programs need `ENTRY DLITCBL` in the link-edit stream (already configured in dbb-app.yaml).
- The z/OS Connect WAR is built via Gradle (`com.ibm.zosconnect.gradle` plugin v1.6.2) from `src/api/build.gradle`. Run with `gradle build` from `src/api/`.
- Frontend WAR is built by the `VanillaFrontend` custom DBB Groovy task — **not** by npm/yarn.

Build entrypoints (run from z/OS USS via Zowe CLI orchestrated by `bash .setup/pipeline-local.sh`):
- Local setup: `bash .setup/setup-local.sh`
- Pipeline: `bash .setup/pipeline-local.sh`

Configuration: `.setup/config/config.yaml` (Jinja2-like templates). Copy from `config.yaml` template; real values go in `.setup/config/setenv.sh` (gitignored).

## Tests (Integration Only — Require a Running z/OS Environment)

Tests are bash scripts hitting live endpoints — there is **no unit test framework**.

```bash
# Run all tests
BASE_URL=http://<host>:9080/api FRONTEND_URL=http://<host>:9081 bash tests/run-all.sh

# Run a single test
BASE_URL=http://<host>:9080/api bash tests/test_get_customer_cics.sh

# Skip IMS tests
IMS_DISABLED=true BASE_URL=... bash tests/run-all.sh
```

Test scripts source `tests/test-setup.sh` which reads ports from `.setup/config/setenv.sh` if `BASE_URL` is not set directly.

## Static Analysis: ZCodeScan

Rules: [`zcodescan/zcodescan-rules.yaml`](zcodescan/zcodescan-rules.yaml). Configured in `zapp.yaml` under the `zcodescan` profile. Run via the Bob IDE ZCodeScan integration or the pipeline. Key enforced rules: `CheckSqlcodeAfterExecSqlRule`, `SqlAvoidSelectStarRule`, `CicsNoHandleRule`, `RespOptionRule`, `UnprotectedAuthCredentialRule`.

## Secret Scanning

Pre-commit hook uses `detect-secrets` (IBM fork). Before committing: `pre-commit run --all-files`. Baseline: `.secrets.baseline`.

## Git — DCO Sign-off is MANDATORY

**Every commit must use `-s`:**
```bash
git commit -s -m "message"
```
PRs with unsigned commits are rejected by CI. See [`.bob/git-commit-guidelines.md`](.bob/git-commit-guidelines.md).

## COBOL Code Style Conventions

- CBL compiler directives at top of file (`CBL CICS('SP,EDF')`, `CBL SQL`) — one per line in columns 8+.
- WORKING-STORAGE uses `77` for standalone scalars, `01` groups with `03`/`05` subordinates.
- LOCAL-STORAGE SECTION used (not WORKING-STORAGE) for re-entrant per-invocation data.
- Host variable group named `HOST-<ENTITY>-ROW` with `HV-` prefix on each field.
- CICS response codes captured into `WS-CICS-RESP` / `WS-CICS-RESP2` (PIC S9(8) COMP).
- Copybooks included via `EXEC SQL INCLUDE <name> END-EXEC` for DB2 structures; `COPY <name>.` for COBOL structures.

## Local Metadata Database

Scanned database location (43 files, stored in [`.bobz/local-settings.json`](.bobz/local-settings.json)):

```
/Users/tingwu/Library/Application Support/IBM Bob/User/workspaceStorage/aa0217907b78721a6130ca7bda121b23/IBM.bob-z-extension/.tmi/WCA4ZTMIOutput/ScannerOutput.db
```

Use `execute_sql_query` (no `databasePath` argument needed — read from `.bobz/local-settings.json`) to query program structure, dependencies, SQL tables, copybooks, and cross-program call relationships before reading source files.

## zapp.yaml — Copybook Search Paths

Remote CICS copybook library: `BANKZ.DBB.BMS.COPY` and `CICSTS63.CICS.SDFHCOB`. Local search uses `**` glob (inefficient for large repos — restrict if needed). TAZ unit test proclib: `SYS1.PROCLIB`, load lib: `BANKZ.V0R1M0.LOAD`.


## Workspace Configuration
Target Platform: IBM Z