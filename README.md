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
`docker run <contenedor>`

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

También puedo asignar un nombre cuando ejecuto run.
``docker run --name <nombre contenedor> <contenedor>``

Me muestra el output del contenedor, incluso si esta apagado. No se ejecuta, muestra el output que quedó registrado
`docker logs <nombre contenedor>`.

Borra contenedor
`docker rm <nombre-contenedor>`

Borra todos los contenedores
`docker rm $(docker ps -aq)`

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
