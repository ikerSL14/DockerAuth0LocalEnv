FROM traefik:v3.6.0-rc1

# Write static and dynamic configuration into the image
RUN mkdir -p /etc/traefik/dynamic \
    && cat > /etc/traefik/traefik.yml <<'EOF'
entryPoints:
  web:
    address: ":80"
  websecure:
    address: ":443"
  traefik:
    address: ":8080"

log:
  level: DEBUG

accessLog:
  format: json

api:
  dashboard: true
  insecure: true

providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
  file:
    directory: "/etc/traefik/dynamic"
    watch: true

EOF

RUN cat > /etc/traefik/dynamic/01-middlewares.yml <<'EOF'
http:
  middlewares:
    gzip-compress:
      compress: {}

    # OAuth2-Proxy ForwardAuth middleware for centralized authentication
    # Uses /oauth2/auth endpoint to verify authentication
    oauth2-proxy-auth:
      forwardAuth:
        address: "http://oauth2-proxy:4180/oauth2/auth"
        authResponseHeaders:
          - "X-Auth-Request-User"
          - "X-Auth-Request-Email"
          - "X-Auth-Request-Access-Token"
        trustForwardHeader: true
    
    # Error handler for authentication failures
    # When forwardAuth returns 401, redirect to OAuth2-Proxy login
    oauth2-error-handler:
      errors:
        status:
          - "401-499"
        service: oauth2-service
        query: "/oauth2/start"
    
  services:
    oauth2-service:
      loadBalancer:
        servers:
          - url: "http://oauth2-proxy:4180"
EOF

# Expose common Traefik ports
EXPOSE 80 443 8080

# Optional metadata
LABEL org.opencontainers.image.vendor="Traefik Labs" \
      org.opencontainers.image.url="https://traefik.io" \
      org.opencontainers.image.source="https://github.com/traefik/traefik" \
      org.opencontainers.image.title="Traefik" \
      org.opencontainers.image.description="A modern reverse-proxy" \
      org.opencontainers.image.version="v3.6.0-rc1" \
      org.opencontainers.image.documentation="https://docs.traefik.io"