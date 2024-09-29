# oracle19ForMacM1M2M3
Manual para desplegar una base de datos Oracle 19c en un pc Mac con procesador M1, M2, M3  
Puedes ver en video las indicaciones https://youtu.be/5X4AhY1zOoU

# Instrucciones para desplegar la base de datos de Oracle 19c con SQL\*Plus (sqlcl)

## Necesitarás

1. Procesador Apple M1, M2, M3
2. Docker
3. Docker compose
4. DBeaver

## Instalaciones

1. Instalar home brew https://brew.sh/
2. Instalar Docker Desktop (incluye docker compose) https://docs.docker.com/desktop/install/mac-install/
3. Instalar DBeaver https://dbeaver.io/download/

## Obtener las imagenes para docker

1. Abrir una ventana de comandos (terminal)
2. Descargar la magen de bd Oracle arm  
   `docker pull dvtoever/oracle:19.3.0-ee-slim-faststart`
3. Descargar la imagen oficial de sqlcl (SQL\*Plus mejorado)  
   `docker pull container-registry.oracle.com/database/sqlcl:latest`

## Preparando el entorno

1. Abrir una ventana de comandos (terminal)
2. Crear el directorio donde se va a ejecutar el proyecto (reemplazar miUsuario por tu usuario Mac)  
   `mkdir /Users/miUsuario/Documents/oracle19`
3. Obtener el id de la imagen (IMAGE ID) de dvtoever/oracle  
   `docker images`
4. Crear un archivo docker-compose.yaml con los siguientes parámetros (reemplazar el id de la imagen por el obtenido en paso 3):
   ```
   services:
      oracle19-arm:
         image: idDeMiImagenPaso3
         container_name: oracle19-arm
         environment:
            - ORACLE_PASSWORD=Oracle123
            - APP_USER=oca
            - APP_USER_PASSWORD=oca
            - ORACLE_DATABASE=XEPDB1
            - INIT_SGA_SIZE=1536
            - INIT_PGA_SIZE=512
         ports:
            - "1531:1521"
         volumes:
            - oracle-data:/opt/oracle/oradata
            - oracle-backup:/opt/oracle/backup
         networks:
            - my-network
         restart: always
   volumes:
      oracle-data:
      oracle-backup:
   networks:
      my-network:
         driver: bridge
   ```
5. Guardarlo en la ruta del paso 2.

## Desplegar la Base de datos

1. Abrir Docker Desktop
2. Abrir la terminal de docker
3. Ir a la ruta donde se colocó el archivo yaml (reemplazar miUsuario por tu usuario Mac)  
   `cd /Users/miUsuario/Documents/oracle19`
4. Ejecutar docker compose  
   `docker-compose up`
5. Verificar que esté corriendo bien (opción "Containers" menú del lado izquierdo)
6. Crear una conexión básica en DBeaver  
   **Host**: localhost  
   **Port:** 1531  
   **Database:** XEPDB1  
   **Username:** sys  
   **Role:** SYSDBA  
   **Password:** Oracle123  
   Test Connection ...
7. Abrir una hoja de trabajo SQL (click derecho, Open SQL Script)
8. Dar permisos al usuario oca (creado con el contenedor)
   ```
   GRANT CREATE SESSION, CREATE USER, SELECT ANY TABLE, CREATE TABLE, CREATE TABLESPACE TO oca;
   ```

## Desplegar el cliente sqlcl (SQL\*Plus)

1. Abrir una ventana de comandos (terminal)
2. Obtener el nombre completo (NAME) o id (NETWORK ID) de la red creada en el contenedor de base de datos  
   `docker network ls`
3. Obtener la dirección IP de la base de datos usando el nombre anterior:  
   _docker network inspect [NAME]_  
    `docker network inspect oracle19_my-network`
4. Copiar la dirección IP del resultado anterior, donde se encuentre el nombre del contenedor de base de datos, el campo IPv4Address, por ejemplo:  
   `"IPv4Address": "172.18.0.2/16",`
5. Desplegar el cliente sqlcl (SQL*Plus) apuntando a la red y dirección IP del contenedor donde está corriendo la base de datos, aquí mismo se define el usuario con el que se quiere conectar  
   **Usuario sys:** *docker run --name [containerName] --network [nombreRed] --rm -it [idImagenSqlcl] [usuarioSys]/[PasswUsuarioSys]@[IPv4Address]:[ContainerPort]/[BD] as sysdba*. Por ejemplo:  
    `docker run --name sqlpluscl --network oracle19_my-network --rm -it 6f58bdb2c2fa sys/Oracle123@172.18.0.2:1521/XEPDB1 as sysdba`  
   **Otro usuario:** *docker run --name [containerName] --network [nombreRed] --rm -it [idImagenSqlcl] [usuario]/[PasswUsuario]@[IPv4Address]:[ContainerPort]/[BD]\*. Por ejemplo:  
    `docker run --name sqlpluscl --network oracle19_my-network --rm -it 6f58bdb2c2fa oca/oca@172.18.0.2:1521/XEPDB1`
