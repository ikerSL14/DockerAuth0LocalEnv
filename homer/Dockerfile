FROM b4bz/homer:latest

# Homer default port
EXPOSE 8080

# Optional metadata
LABEL org.opencontainers.image.title="Homer" \
      org.opencontainers.image.description="A very simple static homepage for your server." \
      org.opencontainers.image.source="https://github.com/bastienwirtz/homer" \
      org.opencontainers.image.url="https://github.com/bastienwirtz/homer"

# Embed minimal Homer configuration to avoid demo bootstrap
ENV INIT_ASSETS=0
RUN mkdir -p /www/assets \
    && cat > /www/assets/config.yml <<'EOF'
title: "Demo dashboard"
subtitle: "Local services"
logo: "assets/icons/logo.svg"
theme: default

links: []

services:
  - name: Monitoring
    items:
      - name: Grafana
        icon: fas fa-chart-line
        url: http://localhost/grafana/
      - name: Prometheus
        icon: fas fa-database
        url: http://localhost:9090/
      - name: Dozzle
        icon: fas fa-terminal
        url: http://localhost:8888/
      - name: Traefik
        icon: fas fa-network-wired
        url: http://localhost:8080/
      - name: FileBrowser
        icon: fas fa-folder-open
        url: http://localhost/filebrowser/
  - name: Databases
    items:
      - name: phpMyAdmin
        icon: fas fa-database
        url: http://localhost/phpmyadmin/
      - name: pgAdmin
        icon: fas fa-database
        url: http://localhost:5050/
  - name: Web
    items:
      - name: App (Nginx)
        icon: fas fa-globe
        url: http://localhost/web/
EOF

