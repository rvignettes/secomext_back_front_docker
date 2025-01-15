
# Instrucciones para Correr el Backend y Frontend con Docker Compose

Aquí se explica cómo utilizar `docker-compose` para configurar y ejecutar el proyecto usando los contenedor para el **backend** y otro para el **frontend** en un entorno Docker.

---

## 1. Configuración del Proyecto

### **Estructura del Proyecto**
El proyecto tiene la siguiente estructura básica:

```
project-root/
├── backend/
│   ├── Dockerfile
│   ├── pruebapractica.war
├── frontend-pruebapractica/
│   ├── Dockerfile
│   ├── dist/
│   │   └── frontend-pruebapractica/
└── docker-compose.yml
```

---

## 2. Configuración de Backend

1. Se creo un archivo `Dockerfile` en el directorio del backend:

   ```dockerfile
   # Se usa la iumagen siguiente que tiene el tomcat10 y el jkd 17 requerido
   FROM tomcat:10.1-jdk17

   #se posiciona en el directoro de trabajo en Tomcat
   WORKDIR /usr/local/tomcat/webapps

   # Copia el archivo WAR al directorio de despliegue de Tomcat
   COPY pruebapractica.war .

   # Exponer el puerto 8080
   EXPOSE 8080
   ```

2. Coloca el archivo `pruebapractica.war` en el mismo directorio que el `Dockerfile`.

---

## 3. Configuración de Frontend

1. Crea un archivo `Dockerfile` en el directorio del frontend:

   ```dockerfile
   #imagen base de NGINX
   FROM nginx:stable-alpine

   #se copia los archivos generados por Angular al directorio de NGINX
   COPY dist/frontend-pruebapractica/browser /usr/share/nginx/html

   # Exponer el puerto 80
   EXPOSE 80
   ```

2. Ajusta la URL del backend en tu código Angular para que apunte al contenedor del backend(ejem):
   ```typescript
   private baseUrl = 'http://localhost:8080/pruebapractica/api'; // URL del backend
   ```
3. Checar de que la carpeta `dist/frontend-pruebapractica` contiene el build de producción generado con Angular:
   ```bash
   ng build --configuration=production
   ```
---

## 4. Configuración del archivo `docker-compose.yml`

Crea un archivo `docker-compose.yml` en la raíz del proyecto con el siguiente contenido:

```yaml

services:
  backend:
    build:
      context: ./pruebapractica
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    container_name: backend-container
    networks:
      - app-network
    volumes:
      - ./pruebapractica/target:/app

  frontend:
    build:
      context: ./frontend-pruebapractica
      dockerfile: Dockerfile
    ports:
      - "80:80"
    container_name: frontend-container
    networks:
      - app-network
    volumes:
      # Mapeo del dist de Angular
      - ./frontend-pruebapractica/dist/frontend-pruebapractica/browser:/usr/share/nginx/html

networks:
  app-network:
    driver: bridge
```

---

## 5. Ejecutar los Contenedores

1. En terminal navega al directorio raíz del proyecto:
   ```bash
   cd /ruta al project-root
   ```

2. Ejecutar el compose:
   ```bash
   docker-compose up --build
   ```

3. Verificar que los servicios estén corriendo:
   - **Backend**: [http://localhost:8080](http://localhost:8080)
   - **Frontend**: [http://localhost](http://localhost)

---

## 6. Detener y Limpiar los Contenedores

1. Para detener los contenedores sin eliminarlos:
   ```bash
   docker-compose stop
   ```

2. Para detener y eliminar los contenedores:
   ```bash
   docker-compose down
   ```

---


