## What it does

1. Updates the system
2. Installs `curl`, `wget`, `git`, `htop`, `neofetch`, `ufw`, `vim`, `ca-certificates`, `fail2ban` (failures are skipped), then enables and starts fail2ban
3. Adds `c` (clear) and `update` (full system update) aliases to `~/.bashrc`
4. Installs Docker from Docker's official repository
5. Prints a summary

## Usage

```bash
curl -fsSL -o setup-vps https://raw.githubusercontent.com/kalgo4444/load-script/main/setup-vps
chmod +x setup-vps
./setup-vps
source ~/.bashrc
```

## Manual steps

The script does not touch SSH or the firewall. Keep your current SSH session open until you have tested a new login.

**SSH:** in `/etc/ssh/sshd_config`, set:

```
PermitRootLogin prohibit-password   # root: key only
PasswordAuthentication yes          # other users: password allowed
PubkeyAuthentication yes            # key login on
```

Make sure root has a key in `/root/.ssh/authorized_keys`, or that another `sudo` user has a password.

**Firewall:** allow SSH before enabling, or you can lock yourself out.

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow OpenSSH  
sudo ufw allow 80/tpc  
sudo ufw allow 443/tcp
sudo ufw --force enable
sudo ufw status verbose
```

**Dokploy port 3000**

```bash
docker service update --publish-rm "published=3000,target=3000,mode=host" dokploy
```
