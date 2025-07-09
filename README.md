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

# La premiere fois à installer sur le serveur
```zsh
curl -fsSL https://get.docker.com | sudo bash
```
```zsh
sudo apt install docker-compose -y
```
```zsh
adduser deployer
usermod -aG docker deployer
```

# 🚀 Procédure de déploiement d'application sur un VPS avec Docker, GHCR, GitHub Actions et Caddy

## ✅ Stack technique validée

| Élément               | Choix retenu                            |
|-----------------------|-----------------------------------------|
| CI/CD                 | GitHub Actions                         |
| Build d’image Docker  | Dockerfile                             |
| Registry Docker       | GitHub Container Registry (GHCR)       |
| Déploiement VPS       | `docker compose pull && up -d`         |
| Source sur le VPS     | ❌ Aucun (uniquement artefacts Docker) |
| Reverse Proxy + HTTPS | Caddy avec certificats auto Let's Encrypt |

---

## 📦 1. Préparation du VPS

### 🔐 Connexion au serveur
```bash
ssh root@<IP_DU_SERVEUR>
```

#### Installation des dépendances /  Mise à jour du système
```zsh
apt update && apt upgrade -y
apt install -y curl git ufw
```
#### Installer docker + docker compose
```zsh
curl -fsSL https://get.docker.com | bash
apt install -y docker-compose
```

#### Créer un utilisateur non root (optionnel mais recommandé)
```zsh
adduser deployer
usermod -aG docker deployer
```

##### Activer le parefeu
```zsh
ufw allow OpenSSH
ufw allow 80
ufw allow 443
ufw enable
```

#### structure du projet serveur
```zsh
~/apps/ton-app/
├── docker-compose.yml
├── Caddyfile
```

#### configuration du caddyfile
```zsh
app.mondomaine.com {
    reverse_proxy frontend:${FRONTEND_PORT}
}

api.mondomaine.com {
    reverse_proxy backend:${BACKEND_PORT}
}
```

#### Fichier .env
```.env
# .env

# PostgreSQL
POSTGRES_DB=mydb
POSTGRES_USER=postgres
POSTGRES_PASSWORD=mysecretpassword

# Backend
DATABASE_URL=postgres://postgres:mysecretpassword@db:5432/mydb

# Ports
FRONTEND_PORT=3000
BACKEND_PORT=4000
```

#### contenu du docker compose
```yaml
services:
  frontend:
    image: ghcr.io/tonuser/frontend:latest
    container_name: frontend
    expose:
      - "${FRONTEND_PORT}"
    restart: always
    networks:
      - app_net
    depends_on:
      backend:
        condition: service_healthy
    env_file:
      - .env

  backend:
    image: ghcr.io/tonuser/backend:latest
    container_name: backend
    expose:
      - "${BACKEND_PORT}"
    restart: always
    networks:
      - app_net
    depends_on:
      - db
    env_file:
      - .env
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:${BACKEND_PORT}/health || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 3

  db:
    image: postgres:15
    container_name: postgres
    restart: always
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - app_net
    env_file:
      - .env

  caddy:
    image: caddy:latest
    container_name: caddy
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
      - caddy_config:/config
    networks:
      - app_net

volumes:
  pgdata:
  caddy_data:
  caddy_config:

networks:
  app_net:
```

#### Fichier GitHub Actions .github/workflows/deploy.yml
```yaml
name: Deploy to VPS

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Log in to GHCR
        run: echo "${{ secrets.GHCR_PAT }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin

      - name: Build Docker image
        run: docker build -t ghcr.io/${{ github.repository }}:latest .

      - name: Push Docker image
        run: docker push ghcr.io/${{ github.repository }}:latest

      - name: Deploy to VPS
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            docker login ghcr.io -u ${{ github.actor }} --password ${{ secrets.GHCR_PAT }}
            cd ~/apps/ton-app
            docker compose pull
            docker compose up -d
```

#### Commandes utiles à exécuter manuellement sur le serveur
```zsh
docker compose pull         # Met à jour l'image depuis GHCR
docker compose up -d        # Relance l’application
docker compose down         # Stoppe et supprime les conteneurs
docker image prune -a       # Supprime les anciennes images inutilisées
```










