# Journal des commandes par étape

## 1. Système et OpenSSH
```bash
hostnamectl
sudo apt update && sudo apt upgrade
sudo apt install openssh-server
sudo systemctl enable --now ssh
sudo ss -lntp | grep ':22'
```

## 2. Réseau
```bash
ip -br addr
ping -c 4 192.168.56.101          # depuis l'hôte
ssh genius_24@192.168.56.101
```

## 3. Utilisateurs, groupes, permissions
```bash
sudo groupadd sshusers
sudo groupadd projet
sudo adduser analyste
sudo usermod -aG sshusers,projet analyste
sudo usermod -aG sshusers,projet genius_24
sudo mkdir /srv/projet
sudo chown root:projet /srv/projet
sudo chmod 2770 /srv/projet
```

## 4. Pare-feu
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from 192.168.56.0/24 to any port 22 proto tcp
sudo ufw enable
sudo ss -lntup
nmap -Pn -p 1-1024 192.168.56.101  # depuis l'hôte
```

## 5. Clés SSH
```bash
ssh-keygen -t ed25519 -C "genius_24@lab-ubuntu"
ssh-copy-id genius_24@192.168.56.101
ssh-copy-id analyste@192.168.56.101
```

## 6. Durcissement SSH
Voir `configs/sshd_config.d/99-hardening.conf`
```bash
sudo sshd -t
sudo sshd -T | grep -E '^(permitrootlogin|maxauthtries|passwordauthentication)'
sudo systemctl reload ssh
ssh -o PubkeyAuthentication=no genius_24@192.168.56.101   # doit être refusé
```

## 7. Fail2ban
Voir `configs/fail2ban/jail.d/sshd-local.conf`
```bash
sudo apt install fail2ban
sudo systemctl enable --now fail2ban
sudo fail2ban-client status sshd
sudo fail2ban-client set sshd unbanip 192.168.56.1
```

## 8. Journaux et ressources
```bash
sudo journalctl -u ssh --no-pager | grep -E "Invalid user|Failed|Accepted" | tail -15
sudo grep -E "Found|Ban |Unban " /var/log/fail2ban.log | tail -10
uptime; free -h; df -h /
```

## 9. Sauvegarde et restauration
```bash
sudo mkdir -p /var/backups/projet
sudo tar -czf /var/backups/projet/sauvegarde-$(date +%F).tar.gz /srv/projet /etc/ssh/sshd_config.d /etc/fail2ban/jail.d /etc/ufw
sudo chmod 600 /var/backups/projet/sauvegarde-*.tar.gz
```

## 10. Audit
```bash
sudo apt install lynis
sudo lynis audit system --quick
sudo grep hardening_index /var/log/lynis-report.dat
sudo sed -i 's/^UMASK.*/UMASK           027/' /etc/login.defs
sudo chage -M 90 -m 1 -W 7 analyste
```
