# Linux Hardening

## Introduction

### Qu'est-ce que le Hardening ?

Le Hardening consiste en une multitude de bonnes pratiques et de configurations pour sécuriser son système que ce soit un serveur ou une
machine personnelle.

### Pourquoi faire ?

Le hardening est devenu quasiment obligatoire quand on doit gérer un serveur connecter à internet pour la simple et bonne
raison qu'une fois exposé votre serveur sera la cible de nombreux **bots** qui vont scanner votre serveur à la recherche
de failles 😨.

### Comment faire ?

Heureusement nous sommes là pour vous, et voici donc notre **Guide Hardenning** !

## Sommaire

### Système
1. Gérer les packages (dpkg --list dpkg --info packageName apt-get remove packageName)
2. fail2ban
3. Sécuriser le service SSH 
4. Désactiver IPv6 
5. Gérer les ports réseau en écoute


### Utilisateur
1. Désactiver les dispositifs USB
2. Appliquer une politique de mot de passe fort
3. Installer un mot de passe sur le GRUB



## Guide

Ici nous prendrons en exemple un serveur sous ***ubuntu serveur 20.04***

### Système

**Avant tout et en priorité mettre à jour son OS et ses logiciels déjà installés**

```shell
sudo apt-get update && sudo apt-get upgrade
```
### 1. **Gérer les packages** ###

Le hardening consiste à avoir le moins de logiciels et package possible afin de limiter le nombre de failles potentielles et d'avoir un système plus propre. On va donc commencer par supprimer tout package dont nous n'avons pas l'usage.
Pour cela la commande ```dpkg``` va servir a gérer les packages, par exemple on peut utiliser:
- ```--list``` pour lister les packages
- ```--remove``` pour supprimer un package

On peut maintenant supprimer tout package inutilisé
<br>

### 2. **fail2ban** ###

Le premier outil que nous allons installer est **fail2ban**, son fonctionnement est simple, mais relativement efficace.
Il va simplement mettre dans une jail (en prison) les ips qui essayent de se connecter à de mainte reprise par **ssh**.
On peut bien sûr gérer les ips qui se sont retrouvé en jail si on sait par exemple trompé nous même trop de fois
ou utilisé une **whitelist** pour être sûr de ne pas se faire ban.<br>

**Installation:**

```shell
sudo apt install fail2ban
```

**Configuration**:

Le fichier à modifier pour gérer ses **jails** et la **whitelist** se trouve dans

```shell
/etc/fail2ban/jail.d/defaults-debian.conf
```

Par exemple si on rajoute les lignes suivantes à la section ```[sshd]``` on peut ban pendant 1h toute ip qui essaye de
se connecter plus de 10 fois en 1min

```shell
maxretry = 10
findtime = 60
bantime = 3600
```

Pour éviter de se ban soit même on peut rajouter la ligne suivante à la section ```[DEFAULT]```, c'est la whitelist.

```shell
ignoreip = {votre ip}
```
<br>

### 3. **Sécuriser le service SSH** ###

Le service ssh est l'une voir la plus grande faille de votre système s'il est mal sécurisé alors cette partie est la plus
**importante**.

Voici une liste de paramètres à modifier dans le fichier ```/etc/ssh/sshd_config```

- ```#PermitRootLogin without-password``` -> ```PermitRootLogin no```<br>
  Empêche de pouvoir se connecter en root depuis une connexion ssh afin d'éviter qu'un attaquant puisse avoir les **clés du royaume**.
- ```#Port22``` -> ```Port XXXX``` Remplacer par n'importe quel port non utilisé à partir de **1024**.<br>
  Cela permet de complexifier la connexion aux attaquants, ils doivent d'abord trouver le port avant l'utilisateur et le
  mot de passe.

Forcer la connexion par clé ssh pour que la connexion soit plus sécuriser et plus facile, pour cela il faut d'abord
[générer une paire de clés ssh](https://docs.ovh.com/fr/dedicated/creer-cle-ssh-serveur-dediees/) sur son client ou ordi perso.

Il suffit ensuite d'ajouter sa **clé publique** sur le serveur dans ```~/.ssh/authorized_keys```.

Dans le fichier ```/etc/ssh/sshd_config``` il faut modifier les lignes suivantes :

- ```#PubkeyAuthentication yes -> PubkeyAuthentication yes```
- ```PasswordAuthentication yes -> PasswordAuthentication no```

> Attention à bien ajouter la clé shh avant pour éviter de rester bloqué à l'extérieur de votre serveur
<br>![Alt Text](https://media.tenor.com/bHGUqVIKzhoAAAAS/let-me-in-eric-andre.gif)

Vous pouvez maintenant redémarrer le système ssh

```shell
sudo systemctl restart ssh
```
<br>

### 4. **Désactiver l'ipv6** ###

Aujourd'hui l'ipv6 n'est pas beaucoup utilisé, si vous n'en avez pas l'usage nous vous conseillons de le désactiver.<br>
Pour le faire il faut ajouter les lignes suivantes dans le fichier ```/etc/sysctl.conf```.
```shell
# désactivation de ipv6 pour toutes les interfaces
net.ipv6.conf.all.disable_ipv6 = 1
# désactivation de l’auto configuration pour toutes les interfaces
net.ipv6.conf.all.autoconf = 0
# désactivavtion de l'ipv6 par default (Par exemple sur les nouvelles cartes réseaux)
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.default.autoconf = 0
```
Puis on peut éxecuter la commande suivante pour appliquer les nouveaux paramètres.
```shell
sysctl -p
```
<br>

### 5. **Gérer les ports réseau en écoute** ###

Savoir quels ports sont écoutés sur votre machine/serveur est important ! Cela permet de savoir s'il y à
une intrusion ou bien savoir quel software utilise quel port.
Pour cela vous pouvez utiliser la commande (qui ne montre que les ports sous écoute) :
```shell
    sudo ss -tulwn | grep LISTEN
``` 
- ```t``` est pour montrer uniquement les sockets TCP
- ```u``` est pour montrer uniquement les sockets UDP
- ```l``` est pour afficher les sockets qui écoutent. Pour exemple, TCP port 22 is opened by SSHD server.
- ```w``` list les processus qui ouvrent des sockets
- ```n``` ne résout pas les noms de services qui n'utilisent pas de DNS

Bien sûr, vous pouvez adapter la commande afin de trouver ce que vous souhaitez.

Pour fermer un port et couper l'application il vous suffit de taper :
```kill "le numero correspondant (ex: 4096)"```
<br>

### Utilisateur
<br>

### 1. **Désactiver les ports physiques** ###

Ce point est valable sur toutes les machines de votre infrastructure car les ports physiques sont une porte d'entrée
sur tout votre réseau. Un utilisateur lambda peut brancher **une clé personnelle infectée** par un malware ou bien un attaquant
lors d'une intrusion à l'aide d'une **bad USB**.

![Alt Text](https://cdn.shopify.com/s/files/1/0068/2142/files/mrduck7_2000x_a63241a5-04a4-4c93-9148-d35f26163e39_800x.gif)

Pour désactiver les ports usb on va utiliser une méthode connue sous le nom de *fake install*.
Il faut d'abord créer un fichier ```block_usb.conf``` dans le dossier ```/etc/modprobe.d``` et y ajouter la ligne suivante.<br>

```shell
install usb-storage /bin/true
```
Pour réactiver les ports usb il vous suffit de supprimer la ligne du fichier.

Cependant, la méthode la plus sûre dans ce cas de figure reste quand même d'obstruer les ports ou de les démonter.
<br>


### 2. **Appliquer une politique de mot de passe fort** ### 

Un mot de passe fort est un "must have" sur son installation, que se soit un serveur ou bien un ordinateur personnel. Il permet de complexifier la tentative
d'intusion par l'utilisisation du mot de passe.
Sur son ordinateur, c'est très simple, il vous suffit de respecter quelques règles tel que :
    - Longueur minimale de 12 caractères
    - Contenir au moins 2 majuscules
    - Contenir au moins 2 chiffres
    - Contenir au moins 2 caractères speciaux

Sur un serveur il existe des commandes qui permettent de forcer l'utilisation de ces bonnes pratiques, pour cela il vous faudra Installer "libpam-cracklib" :
```shell
apt-get install libpam-cracklib
```
Ouvrez le fichier de configuration avec nano ou vim
```shell
sudo vim /etc/pam.d/common-password
```
Et ajoutez-y cette ligne à l'emplacement de la ligne "**password**"
```shell
password required pam_cracklib.so try_first_pass retry=5 minlen=12 difok=2 ucredit=-2 lcredit=-2 dcredit=-2 ocredit=-2 reject_username
```

Avec cette configuration vous avez :
- Retry=5 (Autorise 5 essais pour la saisie du mot de passe)
    - Minlen=12 (Longueur minimale de 12 caractères obligatoire)
    - Difok=3 (Nombre de caractères qui doivent être différents entre l'ancien et nouveau mot de passe)
    - ucredit=-2 (Doit contenir au moins 2 minuscules)
    - lcredit=-2 (Doit contenir au moins 2 majuscules)
    - dcredit=-2 (Doit contenir au moins 2 chiffres)
    - ocredit=-2 (Doit contenir au moins 2 symboles/caractères spéciaux)
<br> 
  
### 3. **Installer un mot de passe sur le GRUB** ###

Nous allons vous montrez comment installer un mot de passe sur le **GRUB** sur une installation **UBUNTU 20.04 LTS server**,
pour cela il faudra vous connecter avec un utilisateur possédant les access privilèges (**sudo** ou **root**)
La première étape est de vérifier votre version du GRUB, pour cela il vous faudra taper la commande suivante dans un terminal :
```shell
grub-install -V
```

Ce tutoriel sera utilisable pour le GRUB , autrement dit le GRUB 2, pour cela assurerez-vous d'avoir **une version 1.99** ou plus ancienne.

> **Warning**
> À partir d'ici il faut faire **tres** attention lors des manipulations

Maintenant vous allez générer votre mot de passe son hash, pour cela vous allez utiliser la commande suivante
```shell
grub-mkpasswd-pbkdf2 
```
> **Surtout n'oubliez pas de copier le hash et de vous rappeler du mot de passe** 


À présent utilisez vim ou nano sur le fichier ```/etc/grub.d/00_header```
Suite à cela vous allez insérer ces lignes à la fin du fichier en remplaçant le hash par **le vôtre**
```
cat << EOF
set superusers="cyberithub"/home/{user}
password_pbkdf2 cyberithub grub.pbkdf2.sha512.10000.C413GH4D904408270DC6B624A2EB947B677E29D63AB2C4B8987AD166FA55729907D5B5ED59AD52CC5842D33ECBC4B77BF04CAED637364B73EF6D50A10AA963CD.220D34777C4556A5B353DFB0C960CDF25DA00AC60C429A88476AF0A69ED0868EE769A7B2A93292426564D6729DB53CB35A2E9A125B793C85969A30C304A7058F
EOF
```

Vous devez à présent mettre à jour votre GRUB :
```shell
 update-grub
```

Et pour finir vous devez redémarrer votre machine soit avec la commande ```init 6``` ou alors par le moyen 
de votre choix.

### 4. **Les bonnes pratiques** ###

Maintenir son système et ses logiciels à jour pour récupérer tout patch de failles:

```sudo apt update && apt upgrade```

Faites des backups régulièrement en cas de malware ou de réinstallation forcée

Ne pas hésiter à réinstaller l'OS en cas de mise à jour majeure (Vous avez une sauvegarde normalement)

Eviter d'installer des packages inutiles afin d'évitter de devoir "faire le ménage"

Limiter les droits des utilisateurs, n'importe qui ne doit pas pouvoir faire n'importe quoi
