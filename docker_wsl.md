# Docker + WSL + LDAP — Documentación

## 1. Arquitectura

Windows
└── Docker Desktop
    └── WSL Ubuntu
        ├── Contenedor: ldap_server (osixia/openldap)
        └── Contenedor: web_app (php:8.2-apache)

Los contenedores se comunican mediante la red interna
de Docker Compose llamada `empresa-docker_default`.

---

## 2. docker-compose.yml

```yaml
version: '3.9'

services:
  ldap:
    image: osixia/openldap:latest
    container_name: ldap_server
    environment:
      LDAP_ORGANISATION: "Empresa"
      LDAP_DOMAIN: "empresa.local"
      LDAP_ADMIN_PASSWORD: "admin123"
    ports:
      - "389:389"

  web:
    build: ./web
    container_name: web_app
    ports:
      - "8080:80"
    depends_on:
      - ldap
```

---

## 3. Dockerfile

```dockerfile
FROM php:8.2-apache

# Instalar dependencias del sistema para LDAP
RUN apt-get update && apt-get install -y \
    libldap2-dev \
    && rm -rf /var/lib/apt/lists/*

# Configurar e instalar extensión LDAP
RUN docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu \
    && docker-php-ext-install ldap

COPY . /var/www/html/
```

---

## 4. Evidencias

### Contenedores activos
![docker ps](evidencias/docker_ps.png)

### Formulario de login
![login](evidencias/login.png)

### Login exitoso
![login ok](evidencias/login_ok.png)

---

## 5. Redes Docker

Docker Compose crea automáticamente una red interna llamada
`empresa-docker_default`. Esto permite que los contenedores
se comuniquen entre sí usando el **nombre del servicio** como hostname.

| Dentro de Docker | Resultado |
|---|---|
| `ldap_connect("localhost")` | ❌ Falla — apunta al mismo contenedor |
| `ldap_connect("ldap")` | ✅ Correcto — apunta al servicio LDAP |

Para verificar la red:
```bash
docker network ls
docker network inspect empresa-docker_default
```

---

## 6. Ventajas frente al entorno local

| Problema local | Solución con Docker |
|---|---|
| Conflictos de puertos | Cada contenedor tiene su propio puerto |
| Versiones diferentes en cada PC | Imagen fija, mismo resultado siempre |
| Apache y PHP mezclados | Cada servicio en su propio contenedor |
| LDAP difícil de instalar | Se levanta con una sola imagen |
| "En mi PC sí funciona" | Ambiente replicable en cualquier equipo |
