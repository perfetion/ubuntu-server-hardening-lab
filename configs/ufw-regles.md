# Règles UFW appliquées

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from 192.168.56.0/24 to any port 22 proto tcp
sudo ufw enable
sudo ufw status verbose
```

Résultat : politique entrante en refus, sortante en autorisation, SSH autorisé uniquement depuis le réseau Host-Only (192.168.56.0/24).
