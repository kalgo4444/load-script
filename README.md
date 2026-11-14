# load-script

`setup-vps` prepares a fresh Ubuntu or Debian VPS in one run.

## What it does

1. Updates the system (`apt update`, `upgrade`, `full-upgrade`, `autoremove`)
2. Installs basic tools: `curl`, `wget`, `unzip`, `git`, `htop`, `neofetch`, `ufw`, `vim`, `ca-certificates`, `tmux`, `fail2ban` (any tool that fails is skipped)
   - fail2ban is then enabled at boot, started, and its status is shown with `fail2ban-client status`
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

## SSH settings (manual)

The script does not change SSH. Do this by hand after it finishes.

> **Before you start:** after this change root can only log in over SSH with a key. Make sure root has a key in `/root/.ssh/authorized_keys`, or that you have another user with `sudo` and a password. Keep your current SSH session open until you have tested a new login.

1. Open `/etc/ssh/sshd_config`:

   ```bash
   sudo vim /etc/ssh/sshd_config
   ```

   Set these three lines (remove the `#` in front if they are commented out):

   ```
   PermitRootLogin prohibit-password
   PasswordAuthentication yes
   PubkeyAuthentication yes
   ```

   | Setting | Meaning |
   | --- | --- |
   | `PermitRootLogin prohibit-password` | root may log in with an SSH key only, not with a password |
   | `PasswordAuthentication yes` | normal users may log in with a password |
   | `PubkeyAuthentication yes` | SSH key login is enabled |

## Close Dokploy port 3000 (manual)

Only for servers that run [Dokploy](https://dokploy.com). The script does not install it.

By default the Dokploy dashboard is open on `http://<server-ip>:3000`. Once you have set a domain for the dashboard in Dokploy and it opens over that domain, remove the public port 3000:

```bash
docker service update --publish-rm "published=3000,target=3000,mode=host" dokploy
```

> **Before you run it:** check that the dashboard opens on your domain. After this command, `http://<server-ip>:3000` stops working, and the domain is the only way into the dashboard.

Use `sudo` in front if your user is not in the `docker` group.
