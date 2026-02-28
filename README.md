# Keycloak with PostgreSQL

Replace the dev `docker-compose.yml` with a production setup:

```yaml
services:
  postgres:
    image: postgres:17
    environment:
      POSTGRES_DB: keycloak
      POSTGRES_USER: keycloak
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U keycloak"]
      interval: 5s
      timeout: 5s
      retries: 5

  keycloak:
    image: quay.io/keycloak/keycloak:26.2
    command: start --import-realm
    environment:
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://postgres:5432/keycloak
      KC_DB_USERNAME: keycloak
      KC_DB_PASSWORD: ${POSTGRES_PASSWORD}
      KC_BOOTSTRAP_ADMIN_USERNAME: admin
      KC_BOOTSTRAP_ADMIN_PASSWORD: ${KC_ADMIN_PASSWORD}
      KC_PROXY_HEADERS: xforwarded
      KC_HOSTNAME: ${KC_HOSTNAME}
      KC_HTTP_ENABLED: "true"
    ports:
      - "8180:8080"
    volumes:
      - ./keycloak/realm-export.json:/opt/keycloak/data/import/realm-export.json:ro
    depends_on:
      postgres:
        condition: service_healthy

volumes:
  pgdata:
```

```env
# .env for production Keycloak
POSTGRES_PASSWORD=your-secret-here
KC_ADMIN_PASSWORD=your-admin-password
KC_ADMIN_USER=your-admin-user
KC_HOSTNAME=subdomain.domain
```

Key settings:
- **`KC_HOSTNAME`** — tells Keycloak its public URL so it generates correct redirect URIs
- **`KC_PROXY_HEADERS: xforwarded`** — trusts headers from Cloudflare Tunnel
- **`command: start`** (not `start-dev`) — enables production mode
- Port mapped to **8180** on host (I probably have something running on 8080)



