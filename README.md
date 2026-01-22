# Roadmap.sh-SSH-Remote-Server-Setup
<https://roadmap.sh/projects/ssh-remote-server-setup>

# Etapes pour configurer la connexion ssh avec clé privée

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

<br>

##
<br>

# En bonus on installe le banisseur des attaques Brute force

### 1. Installation des packages correspondants
````
$ sudo apt update && sudo apt install fail2ban -y
````

### 2. Création d'une configuration locale
````
$ sudo nano /etc/fail2ban/jail.local
````

Ensuite on colle le contenu ci dessous à l'intérieur du fichier
````
[DEFAULT]
bantime = 1h
findtime = 10m
maxretry = 3
banaction = iptables-multiport

[sshd]
enabled = true
port = 2222
filter = sshd-aggressive
logpath = /var/log/auth.log
maxretry = 3
````

> Où :
>
> Section [DEFAULT]
> - **bantime = 1h** représente le durée pendant laquelle une IP sera banie après avoir dépassé le nombre maximal de tentatives.
> - **findtime = 10m** indique la fenêtre de temps durant laquelle fail2ban compte les tentatives échouées. Si le nombre de tentatives dépasse maxretry dans ces 10 minutes, l'IP est bannie.
> - **maxretry = 3** Nombre maximum de tentatives échouées autorisées pendant la période findtime avant de bannir l'IP.
> - **banaction = iptables-multiport**  représente l'action utilisée pour bannir l'IP. Ici, fail2ban utilisera iptables pour bloquer l'IP sur plusieurs ports simultanément.
>
> Section [sshd]
> - **enabled = true** represente le statut de la surveillance SSH
> - **port = 2222** représente le port SSH surveillé.
> - **filter = sshd-aggressive** représente le fichier de filtre utilisé pour détecter les tentatives suspectes dans les logs. Le filtre "sshd-aggressive" est plus strict que le filtre standard "sshd" et bannit plus rapidement.
> - **logpath = /var/log/auth.log** représente le fichier de log surveillé pour détecter les tentatives de connexion SSH échouées.
> - **maxretry = 3** représente la surcharge le paramètre par défaut pour cette jail spécifique.

### 3. Activation du service au démarrage et redémarrage
````
$ sudo systemctl enable fail2ban
$ sudo systemctl start fail2ban
````

### 4. On vérifie le status
````
$ sudo fail2ban-client status sshd
````

A ce stade, le service devrait être démarré et vous pouvez tester.<br>
Si tout est bien installé la commande ci dessus devrait vous retourner ceci

````
Status for the jail: sshd
|- Filter
|  |- Currently failed: 0
|  |- Total failed: 0
|  `- File list: /var/log/auth.log
`- Actions
   |- Currently banned: 0
   |- Total banned: 0
   `- Banned IP list:
````

### 5. Quelques commandes utiles
> #### 5.1 Débannir une IP
> ````
> $ sudo fail2ban-client set sshd unbanip <IP>
> ````

> #### 5.2 Voir les logs
> ````
> $ sudo tail -f /var/log/fail2ban.log
> ````

> #### 5.3 Voir les logs de connexion ssh sur une période
> ````
> $ sudo journalctl -u ssh --since "5 minutes ago"
>