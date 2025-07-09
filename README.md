# procedure_deploiement
Guide pour effectuer des déploiement sur un serveur vps


### Se connecter à un serveur distant
```zsh
 ssh user@ip_du_serveur
```


#### Générer une paire de clé
```zsh
ssh-keygen -t rsa -b 2048
```

#### Trasnférer la clé publique vers serveur distant

```zsh
ssh-copy-id user@ip_du_serveur
```

```bash
# GitHub Actions (CI/CD)
↓
Build de l’image Docker
↓
Push vers GHCR
↓
Connexion SSH vers VPS
↓
docker pull ghcr.io/user/app:tag
↓
docker compose up -d
```

