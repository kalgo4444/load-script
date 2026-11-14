# load-script

`setup-vps` prepares a fresh Ubuntu or Debian VPS in one run.

## What it does

1. Updates the system (`apt update`, `upgrade`, `full-upgrade`, `autoremove`)
2. Installs basic tools: `curl`, `wget`, `unzip`, `git`, `htop`, `neofetch`, `ufw`, `vim`, `ca-certificates`, `tmux`, `fail2ban` (any tool that fails is skipped)
   - fail2ban is then enabled at boot, started, and its status is shown with `fail2ban-client status`
3. Adds aliases to `~/.bashrc` (`c` = clear, `update` = full system update) and turns on line numbers and syntax highlighting in `~/.vimrc`
4. Installs Docker from Docker's official repository and makes sure it is running
5. Configures SSH in `/etc/ssh/sshd_config` and restarts it
   - backs up `/etc/ssh/sshd_config` to `sshd_config.bak` once
   - root may log in with an SSH key only (`PermitRootLogin prohibit-password`), users may log in with a password (`PasswordAuthentication yes`), key login is on (`PubkeyAuthentication yes`)
   - the settings go at the top of the file so files in `sshd_config.d/` (e.g. cloud-init) cannot override them; old lines for these settings are commented out
   - the new config is checked with `sshd -t` and `sshd -T` before it is saved, so a broken config never reaches the server
   - skipped if you run it as root and root has no SSH key, because you would be locked out
6. Prints a summary of what worked, what was skipped and what failed

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

> **Root login:** after step 5, root can only log in over SSH with a key. Make sure you have a key for root or another user with `sudo` before you log out.

When it finishes, load the new aliases in your current terminal:

```bash
source ~/.bashrc
```
