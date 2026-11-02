# load-script

`setup-vps` prepares a fresh Ubuntu or Debian VPS in one run.

## What it does

1. Updates the system (`apt update`, `upgrade`, `full-upgrade`, `autoremove`)
2. Installs basic tools: `curl`, `wget`, `unzip`, `git`, `htop`, `neofetch`, `ufw`, `vim` (any tool that fails is skipped)
3. Adds aliases to `~/.bashrc` (`c` = clear, `update` = full system update) and turns on line numbers and syntax highlighting in `~/.vimrc`
4. Installs Docker from Docker's official repository and makes sure it is running
5. Prints a summary of what worked, what was skipped and what failed

## Requirements

- Ubuntu or Debian (or a system based on them)
- Root access or a user with `sudo`

## Usage

Download the script on your VPS:

```bash
curl -fsSL -o setup-vps https://raw.githubusercontent.com/kalgo4444/load-script/main/setup-vps
```

Make it executable and run it:

```bash
chmod +x setup-vps
./setup-vps
```

You may be asked for your `sudo` password once at the start.

When it finishes, load the new aliases in your current terminal:

```bash
source ~/.bashrc
```
