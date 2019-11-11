# Ejemplo de proxy reverso con nginx y docker

En este ejemplo se construira un demo de un proxy reverso entre un backend y un frontend.

El backend es un simplemente un servicio rest que responte la peticion GET por el puerto 8080 y retorna un texto

El Frontend es un html simple que intentara consumir el backend por el endpont /api

## Descripcion del ejercicio

### Backend

En el directorio backend_demo se encuentra una aplicación de nodejs junto con un Dockerfile para correr la aplicación en docker
Mas información ver: https://nodejs.org/de/docs/guides/nodejs-docker-webapp/

### Frontend 

En el directorio frontend_demo se encuentra el html y el dockerfile para correr una aplicación en el puerto 80 que consuma el end-point /api

### configuaracion de proxy reverso

En el directorio opt se encuentra el archivo de configuracion para que el nginx corra como proxy reveso 

## Ejecucion

*Crear imagen backend*
```bash
    cd backend_demo
    docker build -t backend_image .
```

*Crear imagen front*
```bash
    cd frontend_demo
    docker build -t frontend_image .
```

*Crear contenedor backend*
Se mapea el puerto 46160 para que el host pueda hacer pruebas, pero el proxy reverso usa el puerto interno del contenedor el 8080

```bash
    docker run -d -p 49160:8080 --name backend backend_image
```

*Crear contenedor frontend*
Se mapea el puerto 49161 para que el host pueda hacer pruebas, pero el proxy reverso usa el puerto interno del contenedor el 80

```bash
    docker run -d -p 49161:80 --name frontend frontend_image
```
*Crear network*
```bash
    docker network create proxynetwork
```

*Conectar contenedores a la nueva red*
```bash
    docker network connect proxynetwork frontend
    docker stop frontend
    docker start frontend
    
    docker network connect proxynetwork backend
    docker stop backend
    docker start backend
```

*Crear contenedor de nginx para proxy*
```bash
    docker run -d -p 80:80 --network=proxynetwork --name proxy -v "$(pwd)"/opt/proxy/:/etc/nginx/conf.d nginx:latest
```

*Conclusión:* Si al acceder por el navegador a: http://localhost/
Se ve la pagina web mostrando 

My name is: 
Felix E. Orduz G.

Entonces el ejercicio funciono bien

## Posibles Problemas

- Al crear la red y adjuntar los contendores se me presentaron problemas por eso me toco detener el contenedor y volverlo a arrancar.

- Todas las ejecuciones se toma como punto de inicio el directorio del repositorio.

