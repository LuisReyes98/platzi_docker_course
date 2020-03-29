# Notas de docker

Docker esta hecho en GO

## Run

para correr contenedores con docker run

```shell
docker run
```

el comando hace que mi cliente  de docker le hable al servidor y pida las imagenes que necesita

## Contenedores

es una agrupacion de procesos

es una entidad logica

ejecuta sus procesos de forma nativa como si fuera cualquier otro proceso

el universo de los procesos de los contenedores es como el contenedor los define no pueden ver mas alla
no tienen forma de consumir mas recursos que los que el contenedor permite

el unico software que se comparte entre el contenedor y la maquina host es el kernel del sistema operativo

la unica forma de correr docker 100% nativo es en *linux* ya que su kernel es de *linux*

son similares a las maquinas virtuales en lo que logran pero distintas en como funcionan

se puede limitar cuanto disco puede utilizar

## Estado de docker

para conocer el estado de los procesos en docker

Lista los contenedores activos

```shell
docker ps
```

Lista todos los contenedores

```shell
docker ps -a
```

inspeccionar un contenedor

```shell
docker inspect $ID_DEL_CONTENEDOR
```

```shell
docker inspect -f 'filtro de go' $ID_DEL_CONTENEDOR
```

ejemplo:

```shell
sudo docker inspect -f "{{json .Config.Env}}" youthful_mclaren
```

al usar ```docker run``` se crea un nuevo contendor pero de ser iguales los contenedores comparten la imagen entre si

retorna solo el id de los containers

``` shell
docker ps -aq
```

eliminar multiples contenedores

```shell
docker rm $(docker ps -aq)
```

### logs

```shell
docker logs $NOMBRE_DEL_CONTENEDOR_O_ID
```

muestra los logs de ese contenedor

### otras notas

Explorar el estado de docker:

Crear un contenedor
`docker run <imagen>`

Lista los contenedores activos
`docker ps`

Lista todos los contenedores
`docker ps -a`

Lista solo los IDs de los contenedores
`docker ps -aq`

Inspeccionar un contenedor en específico. Muestra un JSON con toda la metadata del estado del contenedor. También se puede hacer con el nombre del contenedor, ejemplo:

`docker inspect <nombre contenedor>`

`docker inspect <ID contenedor>`

Filtro para encontrar un dato en especifico.
`docker inspect -f ‘{{ json .Config.Env }}’ <nombre contenedor>`

Para hacer el filtro
`-f`
Anotación para pedir algo en particular.
`‘{{ json .Config.Env }}’`
En este caso en particular me va dar el PATH del contenedor, NO el de mi maquina.

renombrar un contenedor que ya existe.

`docker rename <nombre contenedor> <nuevo nombre>`

Asignar nombre a un contenedor
``docker run --name <nombre contenedor> <imagen a ejecutar>``

ejemplo:

```shell
docker run --name db mongo
```

Me muestra el output del contenedor, incluso si esta apagado. No se ejecuta, muestra el output que quedó registrado
`docker logs <nombre contenedor>`.

Borra contenedor
`docker rm <nombre-contenedor>`

Borra todos los contenedores
`docker rm $(docker ps -aq)`

Detener contenedor
`docker stop <nombre-contenedor>`

listar archivos en ubuntu todos `-a` los archivos,`-l` en lista , ordenados por modificacion `-c`

```shell
ls -lac
```

Informacion del sistema operativo que esta corriendo en linux

```shell
uname -a
```

leer el final de un archivo en ubuntu `tail archivo` y que haga follow conforme el archivo cambia `tail -f archivo`

listado de procesos en ubuntu `ps`

listado de procesos de todas las sesiones `ps -fea`

### Terminal interactiva

correr un ubuntu es tan facil como

```shell
docker run ubuntu
```

para correr un ubuntu de forma interactiva , es decir hacer mi terminal actual la que ejecute los comandos del contenedor se hace

```shell
docker run -it ubuntu
```

salir del shell creado es con `exit`

### Correr un comando en un contenedor

los contenedores al inicializarce corren un comando por defecto si queremos que corra otro comando solo debemos escribirlo

```shell
docker run $container $comando
```

ejemplo:

```shell
docker run ubuntu tail -f /dev/null
```

ejecutar un comando en contenedor existente

```shell
docker exec $opciones $nombre_del_contenedor $comando
```

ejemplo:

```shell
docker exec -it name bash
```

`-i` es interactiva la terminal

*Notas* docker SIEMPRE le asigna PID 1 al comando que inicializa el contenedor, cuando ese comando haga `exit` el contenedor se apagara

Opciones para Apagar el contenedor:

matando el contenedor:

```shell
docker kill nombre_del_contenedor
```

Eliminando el contenedor

```shell
docker rm -f nombre_del_contenedor
```

deteniendo el contenedor

```shell
docker stop nombre_del_contenedor
```

### APP Armours y docker bug en ubuntu

docker tiene un error en ubuntu el cual impide eliminar o detener contenedores por falta de perimisos

```shell
Error response from daemon: Cannot kill container: blissful_ritchie: Cannot kill container ea830bfa7010afdefa5e1bf6d5e1e71379d718cbeed9464469409d4e1ec3792a: unknown error after kill: runc did not terminate sucessfully: container_linux.go:388: signaling init process caused "permission denied"
: unknown
```

solucionado con

```shell
sudo aa-remove-unknown
```

este comando remueve los perfiles desconocidos de app armour, y docker al instalarse instala docker por defecto el perfil de app armour 'docker-default' que causa este problema de falta de permisos.

### Exponiendo contenedores

detach

```shell
docker run -d
```

para que el proceso no acapare la consola al ejecutarse

corriendo un server ngnix
con nombre server

```shell
docker run -d --name server nginx
```

hacerle publish al puerto que usa nginx,
mi puerto 8080 sera usado como el 80 del contenedor

```shell
docker run -d --name server -p 8080:80 nginx
```

de esta forma en el puerto 8080 de mi computadora se expone el puerto 80 del contenedor

en mi maquina solo se puede asignar un puerto que no tenga ya un proceso asignado.

### Datos en docker

dandole acceso al contenedor a mi sistema de archivos (una carpeta) para que este alamcene informacion alli

asignandole un volumen a docker declaro la ruta de una carpeta en mi computadora y a que carpeta del contenedor quiero que pertenezca.

usando la flag `-v` declaro este volumen

```shell
docker run --name db -d -v /home/user/docker/platzi_docker_course/mongodata:/data/db mongo
```

de esta forma podemos mantener datos como la tablas de una base de datos de forma constante aunque el contenedor se elimine.

de esta forma

```shell
docker exec -it db bash
```

y `mongo`, la data de la base de datos es constante

**ATENCION** es peligroso usar los volumenes de esta forma ya que todos los cambios en la carpeta hechos por otro proceso, se reflejan en la carpeta desde el punto de vista del contenedor ya que son la misma carpeta

### Volumenes de docker

al ejecutar un contenedor docker le crea un volumen para su ejecucion

ver el listado de los volumenes

```shell
docker volume ls
```

eliminar todos los volumenes de docker

```shell
docker volume prune
```

crear un volumen para mis datos

```shell
docker volume create $nombre
```

montar un volumen en el contenedor

```shell
docker run --mount src=$nombre_volumen,dst=$directorio_dentro_del_contenedor $imagen_a_correr
```

ejemplo:

donde se corre un contenedor de mongo y se especifica que los datos de la base de datos se guardaran en el contenedor

```shell
docker run -d --nd --name db --mount src=dbdata,dst=/data/db mongo
```

al usar volumenes se logra persistir data entre contenedores de manera mucho mas segura que usando un directorio del sistema, ademas de que docker permite usar multiples herramientas como almacenamiento en la nube.

### Imagenes

son plantillas de lo que se quiere correr

descargar solo la imagen sin ejecutarla

```sh
docker pull $nombre_de_la_imagen
```

una imagen no es un archivo ni una cosa, es un conjunto de capas layers

cada capa de una imagen representa una diferencia con la anterior, similar a como en git cada commit representa es una diferencia con el estado anterior de los archivos

listar las imagenes

```sh
docker image ls
```

el nombre de la imagen es el **REPOSITORY** de la imagen

las imagenes existentes para acceso publico se pueden ver en el [Docker Hub](https://hub.docker.com/)

al descargar una imagen se le puede especificar una version o por defecto descarga la ultima

para descargar una version explicita de una imagen solo hay que definirla con dos puntos

```sh
docker pull $repositorio:$version
```

ejemplo

```sh
docker pull ubuntu:18.04
```

las imagenes son eficientes es decir si una imagen usa el mismo contenido que otra, ese contenido solo existe una vez.

permitiendo que al descargar dos veces la misma imagen esta no existe dos veces en el disco y que imagenes similares solo descarguen las diferencias.

### Creando imagenes

las imagenes se crean a partir de archivos `Dockerfile`

al crear la imagen se le da el nombre de la imagen y el path del archivo que especifica el build, el build debe hacerse en una carpeta vacia para empezar donde docker tendra acceso a todos los contenidos que alla en esa carpeta

```sh
docker build -t $repositiorio_base:$version_que_crearemos $path_al_archivo_Dockerfile
```

cambiarle el tag a una imagen

```sh
docker tag $tag_vieja $tag_nueva
```

cambiarle el nombre a una version de un repostiorio

```sh
docker tag $repostiorio:$version $nuevo_repositorio:$nueva_version
```

hacer push de una imagen a docker hub

```sh
docker push $nombre_del_repository:$version
```

ejemplo

```sh
docker tag ubuntu:platzi luisreyes64/ubuntu:platzi
```

al hacer push docker no reenvia los layers que obtuvieste de docker hub

Construir imagenes custom con **Dockerfile**

```sh
docker build -t $tag_de_la_imagen $path_al_contexto
```

hacer build con un dockerfile que no es en la misma carpeta

```sh
docker build -t $tag_imagen -f $path_dockerfile $path_contexto
```

ejemplo:

```sh
docker build -t platziapp -f build/development.Dockerfile .
```

### Analisando imagenes

ver los comando ejecutados en una imagen cuando se hizo build

```sh
docker history $nombre_de_la_imagen
```

ver el output completo

```sh
docker history --no-trunc $nombre_de_la_imagen
```

#### Dive

herramienta de tercero para realizar analisis de imagenes [Dive](https://github.com/wagoodman/dive)

```sh
dive $nombre_imagen
```

### Comandos de dockerfile

`COPY` en el Dockerfile copia las cosas de un directorio a un directorio del contenedor en el contexto de build

ejemplo

copia todo de la carpeta actual a la carpeta de src, que es un path comun donde poner datos de aplicacion de usuario

```Dockerfile
COPY [".", "/usr/src/"]
```

`WORKDIR` hace que los siguientes comandos se ejecuten en el directorio especificado

### Ejecutar el contenedor para desarrollo

para lograr ejecutar la app sin necesidad de reconstruir los contenedores cada vez que haya un cambio se debe pasar un servicio de monitoreo de archivos que reinicie el servidor cada vez que haya los cambios un ejemplo de esto es nodemon, ademas de eso se debe montar la carpeta que contiene el proyecto en desarrollo como un volumen en el contenedor

```sh
docker run --rm --name nodeapp -p 3000:3000 -v /home/luis/Documents/courses/docker/docker:/usr/src platziapp
```

### Realizar conexiones entre contenedores

Ya que los contenedores se comportan como maquinas virtuales para realizar una conexion entre ellos se debe hacer a traves de una red.

ver las redes disponibles de docker

```sh
docker network ls
```

creando una network a la cual otros contenedores se podran conectar despues

```sh
docker network create --attachable $nombre_de_la_red
```

una vez un contenedor este creado y corriendo este puede ser conectado a una red de docker

conectando un contenedor a una network de docker

```sh
docker network connect $network_name $container_name
```

inspeccionar una red de docker

```sh
docker network inspect $network_name
```

cuando diversos contenedores estan conectados a una misma docker network esto pueden acceder a la direccion de red de los otros contendores usando el nombre del contenedor

Ejemplo de correr el contendor con un puerto y varibales de entorno

```sh
docker run -d --name app -p 3000:3000 --env MONGO_URL=mongodb://db:27017/test platziapp
```

### Docker compose

Construccion de entornos de aplicacion de forma estable

#### Versiones

es importante declarar la version de docker compose, ya que entre las versiones 2 y 3 la sintaxis es la misma pero las features son muy diferentes.

#### Servicios

en docker compose se habla de servicios y no de contenedores, ademas cada servicio puede tener mas de un contenedor.

Nombre de los contenedores

por defecto si no se le define el nombre de los contendores de la siguiente forma

```javascript
`${nombre_carpeta_del_contenedor}_${nombre_del_servicio}`
```

#### Comandos

levantar todos los contenedores

```sh
docker-compose up
```

ejecutar docker compose sin output

```sh
docker-compose up -d
```

dar de baja los contenedores

```sh
docker-compose down
```

listar contenedores que se estan ejecutando con docker compose

```sh
docker-compose ps
```

##### Logs

ver los logs de un servicio

```sh
docker-compose logs $nombre_servicio
```

hacer follow de los logs

```sh
docker-compose logs -f $nombre_servicio
```

##### Ejecutar comando en contenedor de un servicio

```sh
docker-compose exec $nombre_servicio $comando
```

ejemplo abriendo el sistema de consola:

```sh
docker-compose exec app bash
```

##### Escalar el servicio

escalar un servicio para que use multiples contenedores

```sh
docker-compose scale $nombre_servicio=$numero_de_contenedores
```

de esta forma es facil simular un servicio de multiples servidores, escalable y de gran escala

### Conceptos de imagenes productivas

Permitir que archivos y carpetas no formen parte de la imagen final

El use de  `.dockerignore` permite ignorar archivos y carpetas en el contexto de build

Realizar tests en la app en el build
los docker file pueden utilizarse para generar multiples contextos y ejecutar acciones en estos contextos y que el output de files de estos contextos no quede en la imagen productiva.

Ejemplo de Dockerfile donde se ejecutan tests y no se guardan los archivos generados por ellos ni los archivos innecesarios

```Dockerfile
FROM node:10 as builder

COPY ["package.json", "package-lock.json", "/usr/src/"]

WORKDIR /usr/src

RUN npm install --only=production

COPY [".", "/usr/src/"]

RUN npm install --only=development

RUN npm run test


# Productive image
FROM node:10

COPY ["package.json", "package-lock.json", "/usr/src/"]

WORKDIR /usr/src

RUN npm install --only=production

# aqui solo trae los archivos de produccion
# ademas el path es dentro del contenedor que llamamos builder
COPY --from=builder ["/usr/src/index.js", "/usr/src/"]

EXPOSE 3000

CMD ["node", "index.js"]
```

### Manejar docker desde un contenedor

Por los problemas de codigo nativo que soluciona docker sigue habiendo un programa que se debe ejecutar en la maquina este es el mismisimo **Docker**.

Y como solucionar este ultimo problema de natividad, esto se puede solventar al utilizar Docker para correr una imagen de Docker.

#### Docker en Docker

Ejecutando docker en contenedor que a su vez se comunica con el docker daemon de la computadora en que se ejecuta

```sh
docker run -it --rm -v /var/run/docker.sock:/var/run/docker.sock docker:19.03.8
```

de esta forma puedes hacer build de las imagenes de build dentro del contenedor de docker y probar las imagenes desde alli y luego subir la imagenes a un servidor que corra docker

este metodo se suele usar mucho en entornos de integracion continua como Jenkins

#### Ejecutando

ejecutando dive con docker dentro de docker para analizar la imagen platziapp

```sh
docker run --rm -it \
-v /var/run/docker.sock:/var/run/docker.sock \
-v $(which docker):/bin/docker \
wagoodman/dive:latest platziapp
```
