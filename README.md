# T-Next

**T-Next** is a pytest-based regression framework for **BusinessWorks 6** applications. It is the next
generation of the classic shell-script test harness ("t-tests"): the same test case folders (EARs, gold files,
`.cfg` files, old `t_setup.sh` / `t_action.sh` / `t_verify.sh` scripts) run unchanged, with a desktop app,
one-click setup, parallel runs, HTML reports and a gold-file workflow on top.

> This repository contains the **framework only** - no test suites, no test data, no passwords or hosts.
> Each user imports the suites from their own test case folder (see [Getting your suites](#getting-your-suites)).

## Features

**Setup and environment**
- One-click setup on Windows: `setup\setup-bwtest.bat` (installs into a virtual environment, asks the machine
  settings, runs the health check). No admin rights needed.
- BW installs found automatically in any folder layout (`C:/D/6.12.1/V12/bw/6.12`, `C:/bw6/BW6121/bw/6.12` ...);
  the version is read from the install. Versions may be written `6.12.1`, `BW 6.12.1` or `BW6121`.
- **Doctor**: checks BW home, bwadmin, Git Bash, test folder, domain folder and every suite file, with a fix hint.

**Tests**
- Imports classic `.testset` suites to readable YAML (`bwtest import <folder>`); old `.sh` scripts keep working
  through the legacy bridge (Git Bash).
- **Sync with test folder**: picks up new suites and new test cases; existing tests and hand-made fixes are kept.
- Suites grouped in folders (e.g. `engine/http`, `engine/jdbc/oracle`); add a test case or a suite in the UI.

**Execution**
- Desktop app (Windows, Edge WebView2; falls back to the browser): New run wizard (build, suites, single tests),
  live progress, stop / kill.
- Same BW steps as before: create domain, AppSpace, AppNode, deploy, action, stop + undeploy, verify.
- All `bwadmin` commands in **one bwadmin shell** - milliseconds per command instead of a new JVM (3-8 s) each.
- Parallel workers and up to 4 runs at the same time, each with its own domain and port block.
- Dry run (shows every bwadmin command without touching BW); fail-fast checks (e.g. Git Bash missing).

**Results**
- HTML report, JUnit XML, `summary.json`, per-test `commands.log` and module properties, time per phase.
- **Gold changes**: when the output differs from the gold file, review the diff and accept the new output as gold -
  per test or in bulk - without a re-run (old gold file backed up).
- Known failures linked to a ticket; rerun all / failed only; optional classic `.sql` results report.

## Requirements
- Windows 10/11 (Linux works for the command line), Python 3.12 or 3.13, Git for Windows (Git Bash).
- A BusinessWorks 6 installation and a test case folder (`<folder>/func/...` with EARs and gold files).
- About 1.5-2 GB RAM per parallel worker.

## Quick start
```bat
git clone <this repo> C:\T-Next
C:\T-Next\setup\setup-bwtest.bat          :: install + machine settings + Doctor
C:\T-Next\start-bwtest.bat                :: desktop app
```
Command line: `.venv\Scripts\bwtest --help` (`run`, `list`, `doctor`, `import`, `accept-gold`, `ui` ...).

## Getting your suites
`suites/` is empty in this repository. After setup:
- Desktop app: **Suites > Sync with test folder...** (or *File > Sync with test folder...*), or
- `bwtest import <your test folder>/func`

## Configuration
- `config/bwtest.yaml` - defaults; `config/local.yaml` - this machine (created by setup, git-ignored);
  `config/profiles/<version>.yaml` - one per BW version.
- `config/databases.yaml` / `config/drivers.yaml` - databases reset and JDBC/JMS drivers installed before a suite.
  Hosts and passwords come from `.env` (`BWTEST_HOST_<ENTRY>`, `BWTEST_PW_...`, see `config/.env.example`) -
  never commit `.env`.
- Old scripts that read the install folder under another variable name: add it to `bw.install_home_vars`
  in `config/local.yaml`, e.g. `bw: {install_home_vars: [INSTALL_HOME, MY_HOME]}`.

## Documentation
See [docs/README.md](docs/README.md): overview, user guide, writing tests, UI reference, architecture, code map,
migrating from t-tests, contributing.

## Development
```bat
cd source
..\.venv\Scripts\python -m pytest tests -q        :: self-tests (a stand-in for bwadmin - no BW needed)
..\.venv\Scripts\python -m ruff check .
```
