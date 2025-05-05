# Taller Docker: Flask + Redis + Docker Hub

Este documento contiene el desarrollo completo del taller de Docker paso a paso. Abarca desde la instalación de Docker, creación de imágenes personalizadas, ejecución de contenedores, integración con Redis usando Docker Compose y publicación en Docker Hub.

---

## Paso 0: Instalación y Preparación

### Requisitos:

* Tener Docker Desktop instalado.
* Acceder al sitio: [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
* Descargar la versión correspondiente a tu sistema operativo.
* Instalar y abrir Docker Desktop hasta que indique: Docker is running.

### Verificar que Docker está funcionando:

```bash
docker --version
```

Debería mostrar una versión activa como: Docker version 24.0.5

---

## Paso 1: Crear una imagen básica (Hello App)

### 1.1 Crear carpeta y archivo base:

```bash
mkdir -p ~/Sites/hello-world
cd ~/Sites/hello-world
echo "hello" > hello
```

### 1.2 Crear Dockerfile:

Archivo `Dockerfile`:

```dockerfile
FROM busybox
COPY /hello /
RUN cat /hello
```

### 1.3 Construir imagen:

```bash
docker build -t helloapp:v1 .
```

### 1.4 Verificar que la imagen fue creada:

```bash
docker images
```

---

## Paso 2: Crear aplicación Flask con Redis

### 2.1 Crear estructura del proyecto:

```bash
mkdir -p ~/Sites/friendlyhello
cd ~/Sites/friendlyhello
```

### 2.2 Crear archivo `app.py`:

```python
from flask import Flask
from redis import Redis, RedisError
import os, socket

redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)
app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(),  visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

### 2.3 Crear archivo `requirements.txt`:

```text
Flask
Redis
```

### 2.4 Crear `Dockerfile` para Flask:

```dockerfile
FROM python:3-slim
WORKDIR /app
COPY . /app
RUN pip install --trusted-host pypi.python.org -r requirements.txt
EXPOSE 80
ENV NAME World
CMD ["python", "app.py"]
```

### 2.5 Construir imagen:

```bash
docker build -t friendlyhello .
```

### 2.6 Ejecutar app localmente sin Redis:

```bash
docker run --rm -p 4000:80 friendlyhello
```

Acceder en el navegador: [http://localhost:4000](http://localhost:4000)

---

## Paso 3: Integrar Redis con Docker Compose

### 3.1 Crear archivo `docker-compose.yaml`:

```yaml
version: '3'

services:
  web:
    build: .
    ports:
      - "4000:80"
    depends_on:
      - redis

  redis:
    image: redis
    command: redis-server --appendonly yes
    volumes:
      - "./data:/data"
```

### 3.2 Ejecutar los servicios:

```bash
docker compose up
```

### 3.3 Verificar en el navegador:

[http://localhost:4000](http://localhost:4000)

Ahora deberías ver el contador de visitas aumentando al refrescar.

### 3.4 Detener servicios:

```bash
docker compose down
```

---

## Paso 4: Publicar la imagen en Docker Hub

### 4.1 Crear cuenta en [https://hub.docker.com/](https://hub.docker.com/)

### 4.2 Iniciar sesión desde terminal:

```bash
docker login
```

### 4.3 Etiquetar imagen:

```bash
docker tag friendlyhello andresazcona/friendlyhello
```

### 4.4 Subir la imagen:

```bash
docker push andresazcona/friendlyhello
```

---

## Paso 5: Ejecutar imagen publicada

### Usar imagen de Docker Hub con Compose:

```yaml
version: '3'

services:
  web:
    image: andresazcona/friendlyhello
    ports:
      - "4000:80"
    depends_on:
      - redis

  redis:
    image: redis
    command: redis-server --appendonly yes
    volumes:
      - "./data:/data"
```

```bash
docker compose up
```

[http://localhost:4000](http://localhost:4000) → Ver visitas incrementarse correctamente.

---

## Ejercicios realizados

### Ejercicio 1: Ejecutar imagen propia desde Docker Hub

* Se modificó el `docker-compose.yaml` para usar `andresazcona/friendlyhello`
* `docker run --rm -p 4000:80 andresazcona/friendlyhello`
* 

### Ejercicio 2: Ejecutar imagen de un compañero

* Se reemplazó la imagen por `medids0526/friendlyhello` en `docker-compose.yaml`
* `docker run --rm -p 4000:80 medids0526/friendlyhello`

---

## Autor

**Andrés Azcona**
Arquitectura de Software – DYAS – 2025
