# TP#4: Docker 101

**Objectifs**:
- Se familiariser avec les concepts d’images et de conteneurs
- Manipuler les commandes de base de Docker
- Comprendre le fonctionnement et la manipulation des volumes

## Étape 1: vérifier la version de Docker

Vérifiez la bonne installation de docker

```sh
docker --version
> Docker version 24.0.6, build ed223bc
```

Si vous souhaitez obtenir plus ample informations sur l'installation de votre hôte Docker:

```sh
docker info
```

## Étape 2: création d'un premier conteneur Docker

Créer un conteneur via la commande Docker:

```sh
docker container run -i -t ubuntu /bin/bash
```

Quelqus explications:
- On lance un conteneur grâce à la commande `docker container run`
- Le flag `-i` demande à Docker de laisser ouvert le flux d'entrée standard du conteneur pour pouvoir lui passer des commandes par ce flux
- Le flag `-t` ordonne à Docker d'assigner un [`tty`](https://fr.wikipedia.org/wiki/Tty_(Unix)) au conteneur que l'on crée. Cela va nous permettre de passer des commandes à notre conteneur (en lui assignant un shell).
- On veut utiliser une image de base qui correspond à une `ubuntu`. Notre conteneur disposera ainsi d'une structure de dossier identique à celle de la dernière distribution Ubuntu.
- en dernier argument, on passe la commande que l'on veut exécuter sur le conteneur. Ici on choisit d'exécuter `/bin/bash` afin d'avoir accès à un shell et pouvoir taper des commandes.

Si vous avez remarqué, le lancement de cette commande crée un flot d'événements en sortie :

```sh
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
6bbedd9b76a4: Pull complete
fc19d60a83f1: Pull complete
de413bb911fd: Pull complete
2879a7ad3144: Pull complete
668604fde02e: Pull complete
Digest: sha256:2d44ae143feeb36f4c898d32ed2ab2dffeb3a573d2d8928646dfc9cb7deb1315
Status: Downloaded newer image for ubuntu:latest
```

Docker ne trouve pas l'image de base Ubuntu sur le système alors il va la télécharger depuis une registry publique: [Docker Hub](https://hub.docker.com/). Chaque ligne avec un identifiant ainsi que la note "Download complete" correspond à une couche qui permet de construire l'image de base.

- Vous devriez maintenant avoir accès à un shell à l'intérieur de votre conteneur:

```sh
root@a68455defe90:/#
```

> Note: vous êtes désormais dans un environnement totalement isolé de votre système hôte.

```sh
root@a68455defe90:/# apt update && apt install vim
[...]
Fetched 24.5 MB in 2s (8330 kB/s)
Reading package lists... Done
Reading package lists... Done
Building dependency tree
[...]
The following NEW packages will be installed:
  file libexpat1 libgpm2 libmagic1 libmpdec2 libpython3.5 libpython3.5-minimal
  libpython3.5-stdlib libsqlite3-0 libssl1.0.0 mime-support vim vim-common vim-runtime
0 upgraded, 14 newly installed, 0 to remove and 15 not upgraded.
Need to get 12.2 MB of archives.
After this operation, 58.3 MB of additional disk space will be used.
Do you want to continue? [Y/n]
```
- Tapez Y (Yes) pour continuer l'installation

Tout ce que vous venez d'effectuer en termes d'installation restera dans le conteneur.
Si vous le détruisez, il ne restera plus rien.

- Tapez `exit` sur le terminal du conteneur afin d'en sortir.

- Listez les conteneurs:

```sh
docker container ps
```

Vous remarquerez que notre conteneur n'y apparaît pas.

En quittant notre conteneur, vous l'avez stoppé uniqmenet. Tant que le conteneur n'a pas été détruit (via commande `docker container rm`), il restera disponible sur votre hôte.

- Un conteneur ne vit que pour le processus qu'il contient
- Testez par vous même:

```sh
docker container run -i -t ubuntu echo "hello world!"
> hello world!
```
En tapant, `docker container ps`, vous voyez que votre conteneur n'est plus présent. 
Il est arrêté.

- Pour lister les conteneurs stoppés:
```sh
docker container ps -a
```

## Étape 3: démarrer un conteneur stoppé

- Lancez un conteneur d'ubuntu en le nomant `mon_ubuntu` :
```sh
docker container run --name mon_ubuntu -i -t ubuntu /bin/bash
```

- Tapez `exit`

- Vérifiez que le conteneur est stoppé:

```sh
# sans le -a !
docker container ps
```

- Redémarrez le conteneur via:

```sh
docker container start mon_ubuntu
```
- Vérifiez que le conteneur est bien lancé:

```sh
# sans le -a !
docker container ps
```

Vous devriez voir `mon_ubuntu` dans la liste.

## Étape 4: s'attacher à un conteneur

Dans l'étape précédente, le conteneur a redémarré avec les mêmes options que lors de son premier lancement. 

Une session interactive nous attend sur le conteneur.

- Attachez-vous au conteneur via:

```sh
docker container attach mon_ubuntu
```

> Note: Vous devriez voir apparaître un shell comme la première fois. Appuyez sur la touche « Entrée » si vous ne le voyez pas.

- Tapez `exit`

Votre container est alors arrêté de nouveau.

## Étape 5: créer un conteneur démon

- Le mode démon permets de lancer des processus en tâche de fond sans bloquer la session active.

- Tapez la commande suivante :
```sh
docker container run --name mon_ubuntu_demon -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
```

La commande docker container `run` couplée au flag `-d` pour dire à Docker que l'on veut que le conteneur soit lancé en tâche de fond.

Le processus lancé en entré de l'image ubuntu est un script `shell` qui sort `hello world` toutes les secondes sur la sortie standard.

- Tapez la commande:
```sh
docker container ps
```

Vous devriez trouver le conteneur `mon_ubuntu_demon`

## Étape 6: examiner un conteneur de l'intérieur

- `docker container logs` permet de voir la sortie standard (`stdout`) du conteneur en cours d'exécution. 

- Tapez la commande suivante:

```sh
docker container logs mon_ubuntu_demon
```

- Vous devriez voir la sortie attendue :
```plain
hello world
hello world
hello world
hello world
hello world
```

Pour suivre en continue les logs du conteneur sans s'attacher au conteneur (`attach` nous forcerait à quitter le processus du conteneur):

```sh
docker container logs -f mon_ubuntu_demon
```

- Tapez `Ctrl-C` pour sortir de la commande de logs (sans terminer le processus du conteneur)

- Pour uniquement lister les 20 plus récents logs de votre conteneur:

```sh
docker container logs --tail 20 my_daemon
```

## Étape 7: inspecter les processus du conteneur

- Pour connaître les processus qu'un conteneur execute, tapez:

```sh
docker container top mon_ubuntu_demon
```

Cette commande va lister tous les processus qui s'exécutent dans notre conteneur de la même manière que l'on utiliserait la commande top de Linux pour lister les processus.
```sh
UID   PID    PPID   C  STIME  TTY  TIME      CMD
root  16597  16581  0  15:03  ?    00:00:00  /bin/sh -c while true; d[...]
root  18857  16597  0  15:39  ?    00:00:00  sleep 1
```

## Étape 8: inspecter les informations d'un conteneur

Pour inspecter une quantité d'informations pratique sur notre conteneur au format JSON, tapez:

```sh
docker container inspect mon_ubuntu_demon
```

```json
[
  {
    "Id": "613bdd482[...]",
        "Created": "2016-11-16T14:03:34.006557847Z",
        "Path": "/bin/sh",
        "Args": [
            "-c",
            "while true; do echo hello world; sleep 1; done"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 16597,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2016-11-16T14:03:34.247842989Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
[...]
```

La commande `inspect` interroge le conteneur et retourne un grand nombre d’information comme les informations de configurations globales par exemple.

## Étape 9: stopper un conteneur

- la commande `docker container stop` permet de stopper pour de bon un conteneur:
```sh
docker container stop mon_ubuntu_demon
```

## Étape 10: supprimer un conteneur

- la commande `docker container rm` permet de supprimer un conteneur:
```sh
docker container rm mon_ubuntu_demon
```

## Étape 11: attacher un volume à notre conteneur

Par défaut, les conteneurs que nous créons génèrent des modifications dans leur environnement. Si on supprime le conteneur, il ne restera plus rien.

Pour conserver les modifications d'un conteneur, on peut attacher un volume de données.

- Créez un nouveau volume `volume_test` :
```sh
docker volume create volume_test
```

Maintenant que nous avons créé notre volume, nous allons monter ce volume sur un nouveau conteneur.

- Utilisez le flag `-v` (ou `--volume`) de la commande `docker container run`:

```sh
docker container run --name mon_ubuntu_avec_volume -i -t -v volume_test:/volume_test ubuntu /bin/bash
```
Notre conteneur a accès au volume `volume_test` et peut y écrire des données.

Créons maintenant un fichier dans ce dossier `/volume_test` avec la commande suivante :

```bash
touch /volume_test/testfile
```

Sortez du conteneur avec `exit` et supprimez-le avec `docker container rm`:

```sh
docker container rm mon_ubuntu_avec_volume
```

Maintenant, créons un autre container avec `volume_test`:

```sh
docker container run --name mon_autre_ubuntu -i -t -v volume_test:/volume_test ubuntu /bin/bash
``` 

Vous pouvez voir que le fichier existe toujours dans le volume.

## Conclusion: résumé des commandes

Bravo ! Vous avez pris en main les commandes de base de Docker afin de créer et manipuler un `conteneur`.

Voici un résumé des commandes vu pendant ce TP:

`docker info` Liste les informations concernant le démon Docker de l'hôte.

`docker --version` Donne des informations sur la version du démon Docker.

`docker container run` Démarre un conteneur, avec :
- `-i` Laisse ouvert l'entrée standard du conteneur
- `-t` Assigne un pseudo-tty (pour session interactive avec shell)
- `-d` Sert à démarrer un conteneur démon.
- `--name` Sert à modifier le nom par défaut assigné au conteneur.
- `-v / --volume` Monte un dossier de la machine hôte sur le système de fichiers du conteneur.

`docker container ls` Liste les conteneurs démarrés.

`docker container ls -a` Liste tous les conteneurs (même ceux qui sont stoppés).

`docker container start` Démarre ou redémarre un conteneur.

`docker container attach` S'attache à la session interactive du conteneur ou observe ses logs (mais Ctrl-C tue le processus/conteneur).

`docker container logs`	Affiche les logs et la sortie standard du conteneur (tous), avec :
- `-f` Affiche le contenu et ce qui est produit en live.
- `--tail X` Affiche les X dernières entrées de la sortie standard.

`docker container top`	Liste les processus qui s'exécutent dans un conteneur.

`docker container stop` Stoppe le conteneur (il reste redémarrable).

`docker container rm` Supprime le conteneur (et son contenu) définitivement.
