# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single executable Bash script, `setup-vps`, that sets up a fresh Ubuntu/Debian VPS: system update, basic tools (including fail2ban), `~/.bashrc`/`~/.vimrc` tweaks, and Docker from Docker's official apt repo. It ends by printing a summary table. Users fetch it straight from `main` via `raw.githubusercontent.com/kalgo4444/load-script/main/setup-vps`, so whatever is on `main` ships right away.

## Checking changes

There is no build or test suite. The script exits immediately on anything that isn't Linux with `apt-get`, so it cannot run on macOS.

```bash
bash -n setup-vps          # syntax check
shellcheck setup-vps       # lint, if installed (the script already has shellcheck directives)
```

To run it end to end, use a throwaway Ubuntu/Debian VM or VPS. A plain Docker container will get through steps 1–3, but the fail2ban and Docker service checks need `systemctl`, so they report failures there.

## How the script is structured

- **Output helpers**: `header`/`info`/`ok`/`warn`/`fail` print colored tags, and colors are turned off when stdout isn't a TTY. `run "description" cmd args...` wraps a command, prints OK/FAIL and passes the exit code through. New steps should use `run`, not bare commands.
- **Privilege handling**: `SUDO` is an array, empty when running as root and `(sudo)` otherwise, so always call `"${SUDO[@]}" cmd`. A background loop keeps the sudo timestamp alive and is killed by the `EXIT` trap. `TARGET_USER`/`TARGET_HOME` point at the real user even under `sudo`, so dotfile edits go to their home, and the files are `chown`ed back to them.
- **apt**: always go through `apt_get`, which is non-interactive and keeps existing config files (`--force-confdef/--force-confold`).
- **Steps** are `step_*` functions (`step_update`, `step_tools`, `step_config`, `step_docker`) called in order from the "Main" section at the bottom. Each prints `header "Step N/4: ..."`.
- **Summary**: `record "name" "STATUS"` appends to `SUMMARY`, and `print_summary` colors each status by its prefix (`OK*` green, `WARNING*` yellow, anything else red). Main records the top-level steps, and `step_docker` records its own sub-steps (4.1–4.3).
- **Failure rules**: if step 1 (system update) fails, the script records the remaining steps as SKIPPED, prints the summary and exits 1. Later steps keep going when something fails and set `EXIT_CODE=1` instead. A tool that is missing or fails to install in step 2 is skipped and counts as a WARNING, not a failure.
- **Idempotency**: the script must be safe to run again. Tools already installed are detected with `dpkg-query` (`is_installed`), and dotfile lines are added with `add_line`, which appends only when the exact line isn't already there.

## Making changes

- To add a tool, append it to the `TOOLS` array. If it needs setup after install (like fail2ban), add a `<tool>_check` function and call it from Main only when `is_installed <tool>` passes, recording its result as `2.x`.
- Adding or removing a top-level step means updating the `Step N/4` headers, the SKIPPED records in the step-1 failure branch, and the header comment at the top of the script.
- Keep `README.md` in sync. Its "What it does" list mirrors the steps and the tool list, and past commits update the two together.
