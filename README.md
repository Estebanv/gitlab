# Gitlab

Personalizacion del yml del repositorio [sameersbn/docker-gitlab](https://github.com/sameersbn/docker-gitlab), para instalar el gilab en al unlu.

Obviamente se requiere el docker-compose.

Solo hace falta ejecutar `docker-compose -p gitlab up -d`.

La la URL esta configurada en gitlab como `gitlab.unlu.edu.ar`. Editar el archivo hosts con los valores que corresponda.

## Administración

Para el upgrade hay que tener en cuenta las 2 fases básicas (siempre y cuando se este usando la personalización de la imgen base).

### Fase 1

Estos pasos se realizan en la maquina que tenga la personalizacion (Con este repo clonado, obviamente).

Se pullea la nueva imagen y se edita el dockerfile:

```
docker pull sameersbn/gitlab:X.X.X # Donde X.X.X es la ultima version de la imagen
mcedit Dockerfile # Cambiar la imagen a la nueva version X.X.X
```

Se buildea con las opciones que sean necesarias (se incluyeron las mas importantes para usar dentro de la UNLu):

```
docker build --build-arg http_proxy=http://proxy.unlu.edu.ar:8080 -t estebanv/gitlab:v1 .
docker push estebanv/gitlab:v1
```

Se commitea y pushea contra github:

```
git commit -a -m 'Actualizada la imagen de gitlab de X.X.X a Y.Y.Y'
git push origin master
```

### Fase 2

En el server:

Primero hacer el backup:

Lo mejor es loguearse al contenedor de gitlab (docker exec -it gitlab bash). Los comandos son (back y restore):

```
sudo -HEu git bundle exec rake gitlab:backup:create RAILS_ENV=production
sudo -HEu git bundle exec rake gitlab:backup:restore RAILS_ENV=production
```

Actualizar la imagen personalizada:

```
docker pull estebanv/gitlab:v1
```

Detener y eliminar los containers del stack (podria ser solo el de gitlab, pero no se podria usar docker-compose):

```
docker-compose -p gitlab stop
docker-compose -p gitlab rm --all
```

Editar el yml para poner la nueva imagen si fuera necesario (la nuestra se llama igual por el momento) y levantar el contenedor/stack segun corresponda:

```
mcedit docker-compose.yml
docker-compose -p gitlab up -d
```
