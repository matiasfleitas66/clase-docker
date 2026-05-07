# TP Docker — MySQL + Java App Server

Datos del alumno
- Nombre: Matias Fleitas

### 1. ¿Qué es Docker?

### 2. Volúmenes en Docker

### 3. Redes en Docker

### 4. ¿Por qué Payara Server?

### 5. Explicación del docker-compose.yml

### 6. Explicación del init.sql

### 7. Dificultades y soluciones

# Capturas de Pantalla Obligatorias

## Parte 1 — Infraestructura Docker (5 capturas)

### 1. Salida de docker --version y docker info en la terminal

### 2. Salida de docker network ls mostrando la red java-net

### 3. Salida de docker volume inspect mysql-data

### 4. Salida de docker ps con ambos contenedores activos

### 5. docker network inspect java-net con ambos contenedores en la red

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

