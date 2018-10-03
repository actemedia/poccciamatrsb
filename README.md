# POC Docker CCI

## Compte Docker hub

* Login : `poccciamatrsb`
* Email : `nicolas.reynis@actemedia.com`
* Pass : `nolynoly`

## Images Docker

### Construire les images

Pour l'image nginx:

```bash
docker build -f Dockerfile-nginxama --tag nginxama:v0.0.1 .
```

Pour l'image wordpress:

```bash
docker build -f Dockerfile-wpama --tag wpama:v0.0.1 .
```

Pour vérifier que les images se sont bien construites:
```bash
# Liste des images
docker image ls | grep ama
# Détail des layers des images construites
docker image history nginxama:v0.0.1
docker image history wpama:v0.0.1
```

### Pousser les images sur le docker hub

Pour rendre les images accessibles il faut les pousser sur un registry.

```
docker login -u poccciamatrsb -p nolynoly
# On tag les images construites précedemment
docker tag nginxama:v0.0.1 poccciamatrsb/nginxama:v0.0.1
docker tag wpama:v0.0.1 poccciamatrsb/wpama:v0.0.1
# On a maintenant deux images dans le namespace de notre compte
docker image ls | grep poccci
# On les upload
docker push poccciamatrsb/nginxama:v0.0.1
docker push poccciamatrsb/wpama:v0.0.1
```

On peut voir que nos images sont maintenant sur le docker hub:  
https://hub.docker.com/r/poccciamatrsb/

## Lancer la stack Docker

```bash
docker stack deploy --compose-file docker-compose.yml stackcci_1
```

Pour voir l'état des differents conteneurs de la stack déployée:

```bash
docker stack ps stackcci_1
docker service ls | grep stackcci_1
```

Pour arreter la stack:

```bash
docker stack rm stackcci_1
```

Pour accéder au wordpress déployé, il faut tout d'abord obtenir l'IP du swarm master qui orchestre la stack.  
Si vous êtes sur linux ce sera probablement votre localhost.  
Si vous êtes sur un autre système qui passe par une machine virtuelle intermédiaire, vous pouvez lister les noeuds avec cette commande:

```bash
$ docker-machine ls
NAME      ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
default   *        virtualbox   Running   tcp://192.168.99.100:2376           v18.06.1-ce
```

Dans ce cas l'url d'accès sera donc `http://192.168.99.100`
**Attention, au premier lancement la stack peut prendre plusieurs minutes à s'initialiser.**
Si vous rencontrez des erreurs 502, vérifiez l'état de vos services avec `docker stack ps stackcci_1 | grep Running`.
Quand tout les processus tourneront de manière stable depuis quelques minutes wordpress sera accessible.

# Aller plus loin

## Rentrer dans un conteneur

Il faut dabord obtenir l'ID du conteneur:

```bash
docker container ls
```

Puis executer une commande dans celui-ci via l'orchestrateur:

```bash
docker exec -t -i <CONTAINER_ID> /bin/sh
```

## Voir les logs d'un service

```bash
docker service ls
docker service logs <SERVICE_NAME>
```

## Nettoyer son environnement

Pour remettre à plat votre machine de développement voici quelques commandes utiles:

```bash
# Supprimer toutes les images
docker rmi $(docker images -q) -f
# Supprimer tout les conteneurs
docker rm $(docker ps -a -q)
# Supprimer tout les volumes
docker volume prune
```

## Modifier l'image wordpress pour intégrer vos développement

Il faut éditer le fichier `Dockerfile-wpama` :  
https://docs.docker.com/develop/develop-images/dockerfile_best-practices/

## Limitations du POC

Actuellement le POC utilise un volume pour l'ensemble du wordpress, cela permet notamment de partager les sources entre les conteneurs `php-fpm` et `nginx` ainsi que de gérer la persistence des données.
Cette approche ne sera pas viable en production car le code applicatif est copié dans le volume. Ce qui veut dire que lorsque vous deploierez une nouvelle version de l'application (avec une nouvelle version de l'image), Le nouveau code ne sera pas visible car il sera "obscurcit" par le volume qui sera sur la couche au-dessus.  
Il faudra donc veiller à ce que le volume contienne uniquement les données métier et non toute l'application comme c'est le cas actuellement. Cela implique aussi que le code applicatif devra être inclut dans les conteneurs `php-fpm` et dans le conteneur `nginx` car le montage du volume ne suffira pas à partager toutes les données.

## Lancer plusieurs instances

Pour lancer une seconde stack, il faut éditer le `docker-compose.yml` pour attribuer un autre port à cette instance:

```yml
services:
  front:
    ports:
      - "80:80"
```

Devient:

```yml
services:
  front:
    ports:
      - "81:80"
```

```bash
docker stack deploy --compose-file docker-compose.yml stackcci_2
```
Il suffit ensuite d'accéder à la deuxième stack sur le port 81 : `http://<IP_SWARM_MASTER>:81`
