# Roadmap.sh-SSH-Remote-Server-Setup
https://roadmap.sh/projects/ssh-remote-server-setup

## Etapes pour configurer la connexion ssh avec clé privée

> #### NB : Pour ce TP nous utiliserons
>
> - Ubuntu server 24.04 LTS comme serveur (machine distante)
> - Windows 11 comme hôte (Celle qui se connectera par ssh au serveur)
>
> La machine serveur est une machine virtuelle créé en local sous VistualBox


Pour cela nous allons suivre minitieusement ces étapes.

### 1. S'assurer que OpenSSH Server est installé sur la VM

> #### 1.1. On met à jour les pacquets ubuntu
> ````
> $ sudo apt update
> ````

> #### 1.2. On installe le package Open SSH Server
> ````
> $ sudo apt install openssh-server
> ````

Maintenant que le serveur Open SSH est installé, on passe à la génération des clés SSH.<br>
Le but ici est de générer la clé privé sur la machine hôte et l'ajouter sur le serveur comme clé authorisée.

### 2. Configuration des clés privées
> #### 2.1. Génération de la clé privée sur la machine hôte
> *Sur Powershell ou CMD à l'emplacement où son souhaite stocker la clé (Dans mon exemple : C:\Users\jb.onomo\\.ssh\)*
> ````
> $ ssh-keygen -t ed25519 -C "BecomeDevopsUbServer" -f C:\Users\jb.onomo\.ssh\devops_vm_key
> ````
Où : 
- -t ed25519 représente le type d'encodage de de la clé
- -C "BecomeDevopsUbServer" représente le commmentaire ou la description de la clé
- -f C:\Users\jb.onomo\.ssh\devops_vm_key l'emplacement des fichiers contenants les clés

Ça va créer :
- devops_vm_key (clé privée)
- devops_vm_key.pub (clé publique)

> #### 2.2. Affichage de la clé publique
> ````
> $ cat C:\Users\jb.onomo\.ssh\devops_vm_key.pub
> ````

Cela affichera une clé du type : <br>

````
ssh-ed25519 AAAAC3NzaXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXhBz+w1sha+ BecomeDevopsUbServer
````

> #### 2.3. Ajout de la clé privée de notre machine hôte dans la liste des clés authorisées de notre Serveur.
>
> #### 2.3.1. On cré le dossier ssh si il n'existe pas sur notre serveur
> ````
> $ mkdir -p ~/.ssh
> ````

> #### 2.3.2. On donne tout les droits au dossier ssh  à l'utiliseaur actuel uniquement
> ````
> $ chmod 700 ~/.ssh
> ````

> #### 2.3.3. On ouvre le fichier contenant les clés privées authorisées à se connecter au serveur
> ````
> $ nano ~/.ssh/authorized_keys
> ````
>
> Ensuite on y va à la ligne on colle la clé privée généré sur la machine hôte

> #### 2.3.4. Ensuite on donne les droits de lecture et ecriture sur ce fichier uniquement à l'utilisateur actuel
> ````
> $ chmod 600 ~/.ssh/authorized_keys
> ````

### 3. Configuration des paramètres SSH sur le serveur
Pour cela nous avons besoin dde modifier le fichier de configuration SSH

````
$ sudo nano /etc/ssh/sshd_config
````

Dans ce fichier, nous allons décommenter et modifier certaines lignes

````
# Port 2222
# PermitRootLogin no
# PasswordAuthentication no
# PubkeyAuthentication yes
# AuthorizedKeysFile .ssh/authorized_keys
````

- **Port 2222 :** c'est pour définir un port de connexion personnalisé
- **PermitRootLogin no :** Pour interdire la connexion ssh par l'utilisateur root
- **PasswordAuthentication no :** Pour interdire la connexion ssh par mot de passe
- **PubkeyAuthentication yes :** Pour autoriser la connexion ssh par clé publique
- **AuthorizedKeysFile .ssh/authorized_keys :** Pour spécifier le fichier des clés autorisées

On enregistre et on ferme le fichier.<br>

### 4. On applique les configuration SSH en redémarant le service
````
$ sudo systemctl restart sshd
````

### 5. Et Enfin on teste la connexion depuis la machine hôte
````
$ ssh -i C:\Users\jb.onomo\.ssh\devops_vm_key -p 2222 username@192.168.X.X
````
> Où :
>
> - **-i C:\Users\jb.onomo\.ssh\devops_vm_key** représente le chemin vers le fichier contenant la clé privée
> - **-p 2222** représente le port de connxion ssh
> - **username@192.168.X.X** représente la combinaison username et IP Address