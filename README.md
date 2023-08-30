# Reverse proxy for Orders project

Other parts:

1. [Revers-proxy](https://github.com/baikov/orders-traefik)
2. [Front](https://github.com/baikov/orders-frontend)
3. [Back](https://github.com/baikov/orders-backend)

## Global reverse-proxy for local development

> Creates an external network `dev-proxy` to which frontend and backend containers are connected.

### Preparation

1. Install Docker Desktop
1. Copy `.env.example` and rename it to `.env`


### Mode 1: local development without https

1. Set project name (the same for Traefik repo, Nuxt repo and Django repo if use all stack)
    ```env
    COMPOSE_PROJECT_NAME=orders
    ```
1. Uncomment `Mode 1` block:
    ```env
    # 1. As reverse-proxy for development on localhost with http
    COMPOSE_FILE=local.yml
    DOMAIN=localhost  # or another aliace for 127.0.0.1 declared in etc/hosts
    ```
1. Run `docker compose build` and `docker compose up -d`

Traefik dashboard is available at: http://localhost:8080/dashboard/#/

### Mode 2: local development with custom domain and https

1. Set project name (the same for Traefik repo, Nuxt repo and Django repo if use all stack)
    ```env
    COMPOSE_PROJECT_NAME=orders
    ```
1. Uncomment `Mode 2` block:
    ```env
    # Mode 2: SSL-mode (with custom domain)
    COMPOSE_FILE=local.ssl.yml
    DOMAIN=orders.local
    ```
1. Add local domain + subdomain for dashboard in `/etc/hosts`:
    ```vim
    127.0.0.1 localhost orders.local tr.orders.local
    ```
1. Apply changes:
    ```bash
    sudo killall -HUP mDNSResponder
    ```
1. Install `mkcert` (or another tolls for local certificates)
1. Go to `./compose/local-ssl/cert/`
1. Generate a wildcard certificate for a local domain:
    ```bash
    mkcert -cert-file orders.local.pem -key-file orders.local-key.pem orders.local "*.orders.local"  # * for subdomains
    ```
1. Install local CA certificate with the command:
    ```bash
    mkcert -install
    ```
1. If you use not `orders.local` - go to `./compose/local-ssl/dynamic/` and edit `tls.yml`. Change the file names to your own
1. Run `docker compose build` and `docker compose up -d`

Traefik dashboard is available at: https://tr.orders.local/dashboard/#/

## Production reverse-proxy in Docker (Mode 3)

> Creates an external network `global` to which frontend and backend containers are connected.

1. Copy `.env.example` and rename it to `.env` on production server
1. Set project name (the same for Traefik repo, Nuxt repo and Django repo if use all stack)
    ```env
    COMPOSE_PROJECT_NAME=orders
    ```
1. Uncomment `Mode 3` block and change domain and email:
    ```env
    # Mode 3: As reverse-proxy for production
    COMPOSE_FILE=production.yml
    DOMAIN=orders.baikov.dev
    EMAIL=emailfor@letsencrypt.com  # for Let's Encrypt resolver
    ```
1. Run `docker compose build` and `docker compose up -d`
1. If the domain is correctly bound to the server, the certificate will be issued automatically with Let's Encrypt

Traefik dashboard is available at: https://tr.baikov.dev/dashboard/#/
