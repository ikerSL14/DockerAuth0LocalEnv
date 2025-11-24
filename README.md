**Cómputo en la Nube – Parcial 2**

Iker Ignacio Salazar Liévano — 212H17013

**🚀 Plataforma Multiservicio con Docker, Traefik, OAuth2 y Monitoring**

Este proyecto implementa una arquitectura completa basada en Docker Compose, que integra servicios web, autenticación, monitoreo, bases de datos y herramientas administrativas.
Todo el tráfico es gestionado mediante Traefik como reverse proxy, y la autenticación se implementa mediante OAuth2-Proxy.

**🧩 Arquitectura del Proyecto**

El sistema está organizado en tres capas principales:

1️⃣ Data Layer — Bases de Datos

Servicios dedicados al almacenamiento persistente:

- PostgreSQL
- MariaDB
- pgAdmin 4
- phpMyAdmin

Cada base de datos utiliza volúmenes administrados por Docker.

2️⃣ Backend Layer — Aplicaciones Internas

Incluye:

- Nginx + PHP-FPM para servir aplicaciones ubicadas en /web
- Filebrowser (protegido por OAuth2-Proxy)
- Servicios inicializadores (init scripts, configuración de Filebrowser)

3️⃣ Edge Layer — Gateway, Autenticación y Monitoreo

Servicios expuestos al usuario final:

- Traefik 3
Reverse proxy con enrutamiento dinámico y middlewares.

- OAuth2-Proxy
Provee autenticación centralizada para todos los servicios.

- Prometheus
Recolección de métricas.

- Grafana
Visualización bajo subpath /grafana.

- Dozzle
Visualización de logs en tiempo real.

- Homer Dashboard
Página inicial (landing page) accesible en /.

**🔐 Autenticación: OAuth2-Proxy + Traefik**

Toda la autenticación del sistema se realiza mediante:

- OAuth2-Proxy (con soporte para OIDC / OAuth2)
- Traefik para redirección, protección de servicios y envío de cabeceras.
- OAuth2-Proxy actúa como capa de seguridad para servicios internos, incluyendo:
   - Filebrowser
   - Grafana
   - phpMyAdmin
   - Prometheus
   - Web PHP
   - Dozzle
   - Homer

**📦 Servicios y Rutas Disponibles**

Todos accesibles desde: http://localhost

**🛠 Tecnologías Implementadas**

- Docker & Docker Compose (contenedores)
- Traefik 3 (reverse proxy)
- OAuth2-Proxy
- Grafana + Prometheus (monitoreo)
- Filebrowser
- MariaDB + PostgreSQL
- Nginx + PHP-FPM

**▶ Cómo iniciar el proyecto**

- Iniciar todos los servicios
docker compose up --build -d

- Detener contenedores
docker compose down

- Borrar volúmenes
docker compose down -v

**📁 Estructura General del Proyecto**
.
├── docker-compose.yml
├── grafana/
├── prometheus/
├── traefik/
├── oauth2-proxy/
├── php-fpm/
├── nginx/
├── homer/
├── filebrowser/
└── README.md

**🧪 Requisitos**

- Docker Desktop actualizado

- Configuración válida de OAuth2-Proxy (según proveedor elegido)

- Traefik con acceso al socket de Docker

**📝 Notas**

- Toda la infraestructura funciona completamente en localhost.

- No se requiere configuración externa.

- Los volúmenes persisten automáticamente.