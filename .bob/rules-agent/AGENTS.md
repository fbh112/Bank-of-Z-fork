# Agent Coding Rules (Non-Obvious)

## Commits
- ALWAYS use `git commit -s -m "..."` — the `-s` (DCO sign-off) is mandatory; CI will reject unsigned commits.

## COBOL
- CICS and SQL compiler suffixes are **auto-appended** by DBB based on source content — never add `CBL CICS` or `CBL SQL` directives manually unless the program genuinely needs them at edit time.
- `IBTRAN.cbl` requires unique link-edit parms (`DLL`, `JAVAIOP(JAVA64)`, etc.) — do not apply standard IMS link-edit stream to it.
- IMS COBOL batch programs need `ENTRY DLITCBL` in their link-edit stream (already in `dbb-app.yaml` for the known list; add new programs there too).
- Per-invocation data goes in LOCAL-STORAGE SECTION, not WORKING-STORAGE, for CICS re-entrancy.
- DB2 include structs via `EXEC SQL INCLUDE <name> END-EXEC`; COBOL copybooks via `COPY <name>.`

## API (z/OS Connect)
- Operations live in URL-encoded path directories under `src/api/src/main/operations/`. Add a new endpoint by creating a new URL-encoded directory + HTTP verb subdirectory containing `operation.yaml`, `request.yaml`, `response_*.yaml`, and `response_mapping.yaml`.
- The Gradle plugin is `com.ibm.zosconnect.gradle` v1.6.2 — build from `src/api/` with `gradle build`.

## Frontend
- `src/frontend/config.js` determines the API base URL at runtime from `window.location`. Never hardcode port 9080/9444 anywhere else.
- The frontend is packaged into a WAR by the `VanillaFrontend` DBB Groovy task — no npm/webpack.

## ZCodeScan Rules (enforced — will fail pipeline)
- Always check SQLCODE after every EXEC SQL statement.
- Never use `SELECT *` in embedded SQL.
- CICS programs must use RESP option (not HANDLE CONDITION).
- No unprotected credentials in source.
