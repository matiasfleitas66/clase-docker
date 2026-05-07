# TP Docker — MySQL + Java App Server

Datos del alumno
- Nombre: Matias Fleitas

### 1. ¿Qué es Docker?

docker es una plataforma para aislar/"empaquetar" aplicaciones con sus dependencias de una manera mas ligera que una maquina virtual entera, compartiendo el kernel del anfitrion.

### 2. Volúmenes en Docker

un volumen en docker es la manera que tienen los contenedores para guardar datos que persistan despues de su ciclo de vida.

| tipo              | descripcion                                                   |
| -                 | -                                                             |
| `named volumes`   | gestionados por docker, recomendados para bases de datos.     |
| `bind mounts`     | mapeo directo de un directorio del host.                      |
| `tmpfs mounts`    | almacenamiento en memoria ram (no persistente).               |

### 3. Redes en Docker

los contenedores pueden comunicarse por redes gestionadas por docker, usando sus hostnames por el DNS interno de docker.

| tipo      | descripcion                                                   | uso tipico                    |
| -         | -                                                             | -                             |
| `bridge`  | red privada aislada. contenedores se comunican por nombre.    | desarrollo, microservicios    |
| `host`    | el contenedor comparte la red del host directamente.          | alto rendimiento (linux)      |
| `none`    | sin acceso a red.                                             | procesos batch aislados       |
| `overlay` | red multi-host para docker swarm.                             | produccion distribuida        |

### 4. ¿Por qué Payara Server?

para un entorno docker java empresarial con gui, payara server es la opcion mas
recomendable por las siguientes razones:

- es una distribucion de glassfish mantenida activamente con soporte de produccion.
- soporta jakarta ee completo: jpa, ejb, jax-rs, cdi, jms, y mas.
- incluye payara admin console, una consola de administracion web (gui) accesible en el puerto `4848`.
- tiene imagenes docker oficiales optimizadas y actualizadas.
- escala bien de desarrollo a produccion sin cambiar el stack tecnologico.

alternativas como wildfly (jboss), tomee o ibm liberty tambien tienen soporte docker,
pero payara ofrece la mejor combinacion de gui integrada, jakarta ee completo y mantenimiento
activo en contenedores.

### 5. Explicación del docker-compose.yml

docker compose permite definir y gestionar entornos multi-contenedor mediante un único archivo
`yaml`. en lugar de ejecutar múltiples comandos docker run con decenas de flags, definís toda
la infraestructura en un archivo versionable que va junto al código fuente del proyecto.
- un solo comando para levantar todo el entorno: `docker compose up -d`
- un solo comando para bajar todo: `docker compose down`
- el archivo `docker-compose.yml` se versiona con git junto al proyecto.
- las dependencias entre servicios (`depends_on`) se gestionan automáticamente.
- los healthchecks garantizan que mysql esté listo antes de arrancar payara.

```yaml
# docker-compose.yml
version: '3.9'

# ──────────────────────────────────────
# REDES
# ──────────────────────────────────────
networks:
  java-net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16

# ──────────────────────────────────────
# VOLÚMENES
# ──────────────────────────────────────
volumes:
  mysql-data:
    driver: local

# ──────────────────────────────────────
# SERVICIOS
# ──────────────────────────────────────
services:

  # ── MySQL ──────────────────────────
  mysql:
    image: mysql:8.0
    container_name: mysql-container
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: root1234
      MYSQL_DATABASE: appdb
      MYSQL_USER: appuser
      MYSQL_PASSWORD: apppass
    volumes:
      - mysql-data:/var/lib/mysql
      - ./mysql/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    ports:
      - '3306:3306'
    networks:
      - java-net
    healthcheck:
      test: ['CMD', 'mysqladmin', 'ping', '-h', 'localhost', '-proot1234']
      interval: 10s
      timeout: 5s
      retries: 5

  # ── Payara Server ───────────────────
  payara:
    image: payara/server-full:6.2024.6
    container_name: payara-container
    restart: unless-stopped
    environment:
      PAYARA_PASSWORD: admin1234
    ports:
      - '8080:8080'   # HTTP aplicaciones Java
      - '4848:4848'   # Admin Console (GUI)
      - '9009:9009'   # Debug remoto Java (JDWP)
    networks:
      - java-net
    depends_on:
      mysql:
        condition: service_healthy
    volumes:
      - ./payara/deployments:/opt/payara/deployments
```

### 6. Explicación del init.sql

este script se ejecuta automaticamente la primera vez que el contenedor mysql
inicializa la base de datos

```sql
-- mysql/init.sql
USE appdb;

CREATE TABLE IF NOT EXISTS usuarios (
  id         INT AUTO_INCREMENT PRIMARY KEY,
  nombre     VARCHAR(100) NOT NULL,
  email      VARCHAR(150) UNIQUE NOT NULL,
  creado_en  TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO usuarios (nombre, email) VALUES
  ('Admin', 'admin@example.com'),
  ('Test User', 'test@example.com');
```

### 7. Dificultades y soluciones

nada aun

# Capturas de Pantalla Obligatorias

## Parte 1 — Infraestructura Docker (5 capturas)

### 1. Salida de `docker --version` y `docker info` en la terminal

### 2. Salida de `docker network ls` mostrando la red java-net

### 3. Salida de `docker volume inspect mysql-data`

### 4. Salida de `docker ps` con ambos contenedores activos

### 5. `docker network inspect java-net` con ambos contenedores en la red

## Parte 2 — MySQL (3 capturas)

### 6. Logs de MySQL mostrando: ready for connections

### 7. Salida de SHOW DATABASES; mostrando la base appdb

### 8. Salida de SELECT * FROM usuarios; con los datos del init.sql

## Parte 3 — Payara Admin Console / GUI (5 capturas)

### 9. Pantalla de login de Admin Console en http://localhost:4848

### 10. Dashboard principal de Payara tras iniciar sesión

### 11. Pantalla del Connection Pool MySQLPool creado

### 12. Resultado del botón Ping mostrando conexión exitosa a MySQL

### 13. JDBC Resource jdbc/MySQLDS visible en la consola

## Parte 4 — Conectividad entre contenedores (1 captura)

### 14. Salida del ping de Payara hacia mysql-container desde la terminal

