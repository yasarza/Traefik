# Traefik v3 Reverse Proxy Setup

A complete Traefik v3 deployment with automatic SSL certificates via Let's Encrypt and Cloudflare DNS challenge.

## 📁 File Structure

```
Traefik/
├── docker-compose.yml    # Main Docker Compose file
├── traefik.yml           # Traefik static configuration
├── config.yml            # Traefik dynamic configuration (middlewares, external services)
├── .env                  # Environment variables (dashboard credentials, Cloudflare email)
├── cf-token              # Cloudflare API token (secret)
├── acme.json             # SSL certificates storage (auto-generated)
└── logs/                 # Traefik logs directory
    ├── traefik.log       # General Traefik logs
    └── access.log        # Access logs
```

## 🚀 Quick Start

```bash
# 1. Clone the repo
git clone https://github.com/yasarza/Traefik.git
cd Traefik

# 2. Create Docker network
docker network create proxy

# 3. Create config files from examples
cp .env.example .env
cp cf-token.example cf-token
touch acme.json
mkdir -p logs

# 4. Set permissions
chmod 600 acme.json
chmod 600 cf-token

# 5. Edit configuration files with your domain and credentials

# 6. Deploy
docker compose up -d
```

## 📋 Setup Steps

### Step 1: Prerequisites

- Docker and Docker Compose installed
- A domain name you own
- Cloudflare account managing your domain's DNS (or another supported provider)

### Step 2: Configure Cloudflare API Token

1. Go to [Cloudflare Dashboard](https://dash.cloudflare.com)
2. Navigate to: **Profile** → **API Tokens** → **Create Token**
3. Use template: **Edit Zone DNS**
4. Configure permissions:
   - **Zone** - **DNS** - **Edit**
   - **Zone Resources** - **Include** - **Specific zone** - *Your domain*
5. Create and copy the token
6. Paste the token into `cf-token` file

### Step 3: Generate Dashboard Password

```bash
# Install htpasswd (Ubuntu/Debian)
sudo apt install apache2-utils

# Generate password hash
echo $(htpasswd -nB admin) | sed -e s/\\$/\\$\\$/g
```

Paste the output into `.env` as `TRAEFIK_DASHBOARD_CREDENTIALS`.

### Step 4: Update Configuration Files

- **`.env`**: Add your password hash and Cloudflare email
- **`traefik.yml`**: Replace `your-email@example.com` with your email
- **`docker-compose.yml`**: Replace `yourdomain.com` with your actual domain

### Step 5: Create DNS Records

| Type | Name    | Content        |
|------|---------|----------------|
| A    | traefik | Your Server IP |
| A    | *       | Your Server IP |

### Step 6: Deploy

```bash
docker compose up -d
```

Access dashboard at: `https://traefik.yourdomain.com`

---

## 🐳 Adding Docker Services

```yaml
services:
  myapp:
    image: myapp:latest
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.myapp.entrypoints=https"
      - "traefik.http.routers.myapp.rule=Host(`myapp.yourdomain.com`)"
      - "traefik.http.routers.myapp.tls=true"
      - "traefik.http.routers.myapp.tls.certresolver=cloudflare"
      - "traefik.http.services.myapp.loadbalancer.server.port=8080"
    networks:
      - proxy

networks:
  proxy:
    external: true
```

---

## 🔧 Troubleshooting

| Error | Solution |
|-------|----------|
| `acme.json permissions too open` | Run `chmod 600 acme.json` |
| `401 Unauthorized` | Regenerate password hash with `$$` escaping |
| `Certificate not valid` | Check Cloudflare API token permissions |

---

## 📚 Resources

- [Traefik Documentation](https://doc.traefik.io/traefik/)
- [Video Tutorial - Jim's Garage](https://www.youtube.com/watch?v=CmUzMi5QLzI)
- [DNS Providers](https://doc.traefik.io/traefik/https/acme/#providers)
