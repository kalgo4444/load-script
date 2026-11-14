# load-script

`setup-vps` prepares a fresh Ubuntu or Debian VPS in one run.

## What it does

1. Updates the system (`apt update`, `upgrade`, `full-upgrade`, `autoremove`)
2. Installs basic tools: `curl`, `wget`, `unzip`, `git`, `htop`, `neofetch`, `ufw`, `vim`, `ca-certificates`, `tmux`, `fail2ban` (any tool that fails is skipped)
   - fail2ban is then enabled at boot, started, and its status is shown with `fail2ban-client status`
3. Adds aliases to `~/.bashrc` (`c` = clear, `update` = full system update) and turns on line numbers and syntax highlighting in `~/.vimrc`
4. Installs Docker from Docker's official repository and makes sure it is running
5. Creates a new user and adds it to the `sudo` group
   - asks for a name (empty means `noname`), turns it into lowercase and rejects `root`, `admin`, `ubuntu`, `daemon`
   - the name must start with a letter, use only `a-z 0-9 _ -` and be at most 32 characters
   - if the user does not exist it is created with `adduser`, which asks for the password
6. Configures SSH without touching the main config
   - backs up `/etc/ssh/sshd_config` to `sshd_config.bak` once
   - writes `/etc/ssh/sshd_config.d/10-ttt.conf`: root may log in with an SSH key only (`PermitRootLogin prohibit-password`), users may log in with a password (`PasswordAuthentication yes`), key login is on (`PubkeyAuthentication yes`)
   - checks the config with `sshd -t` and removes the file if it is broken, so you are not locked out
   - checks with `sshd -T` that password login is really on (another config such as cloud-init may turn it off) and stops if it is not
   - restarts the SSH service
   - skipped if step 5 failed, because root would lose password login without a sudo user to fall back on
7. Prints a summary of what worked, what was skipped and what failed

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

You may be asked for your `sudo` password once at the start. Near the end it asks for the new user's name and password.

> **Before you run it as root over SSH with a password:** after step 6, root can only log in with an SSH key. Log in as the new user from then on (`ssh <name>@<server-ip>`) and use `sudo`.

When it finishes, load the new aliases in your current terminal:

```bash
source ~/.bashrc
```
