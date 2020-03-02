

GALLAND Cyprien
VATIN Clement

# TP 3 : Gestion des paquets

## Exercice 1 : Commandes de base

Mise à jour du système : on utilise les commandes `sudo apt update` (met à jour l'index des paquets) puis `sudo apt upgrade` (mise à jour des paquets qui le peuvent)..

**1)** La commande `cat /var/log/apt/history.log` permet de voir, entre autres, les 5 derniers paquets mis à jours. Pour nous il s'agit de libwrap0, openssh-sftp-server, ncurses-term, openssh-server et ssh-import-id.

**2)** Pour compter le nombre de paquets installés grâge à dpkg, on utilise la commande `sudo dpkg -l`, qui liste les paquets installés, une ligne par paquet : On compte alors le nombre de lignes (en otant les 5 premières, qui ne sont pas des paquets), dans notre cas 548. On peut aussi passer par `dpkg -l | wc --lines`, qui affiche le nombre de lignes du retour de la commande `dpkg -l` : En sachant le nombre de lignes à enlever (5), on trouve le nombre de paquets. Pour effectuer la même opération avec apt, on utilise `apt list --installed | wc -l`, qui nous renvoie 548 lignes, dont une ligne d'en tête à enlever, soit 548 paquets : le nombe de paquets est donc le même quel que soit la commande, ce qui est rassurant.

**3)** Pour connaître le nombre de paquets disponibles en téléchargement, on utilise `apt list | wc -l`. Grâce à cette commande, on peut savoir qu'il y a 61599 paquets disponibles au téléchargement.

**4)** Pour créer l'alias permettant la mise à jour du système, on utilise la commande `alias maj='sudo apt update && sudo apt upgrade'`.

**5)** Le paquet fortune à pour utilité l'affichage de petits messages, proverbes ou citation à chaque fois que la commande *fortune* est utilisé dans le terminal. Pour l'installer, la commande est `sudo apt install fortune-mod`.

**6)** On veut connaitre les paquets permettant de jouer au sudoku : `apt list | grep "sudoku"` nous renvoie, dans la liste de tout les paquets possibles, ceux contenant le mot sudoku : Ici il y en a 3, à savoir gnome-sudoku, ksudoku et sudoku.

**7)** Pour lister les derniers paquets installés explicitement avec `apt install`, on utilise `grep "install" /var/log/dpkg.log`.

## Exercice 2
Pour trouver le paquet d'origine de la commande `ls`, on tape la commande `dpkg -S '/bin/ls'`, qui nous renvoie le paquet coreutils.

On peut désormais écrire un script appelé *origine-commande*, prenant en argument le nom d’une commande, et indiquant quel paquet l’a installée :

`a=$(which -a $1 | tail -n 1)`
`
b =$(dpkg -S $a)`

`echo $b`

## Exercice 3 
On souhaite écrire une commande qui aﬀiche “INSTALLÉ” ou “NON INSTALLÉ” selon le nom et le statut du package spécifié dans cette commande :

On passe par un script contenant le code suivant :

`a= $(apt list --installed | grep $1 wc --lines)`

`if [$a == 0] ; then`

`echo "Non installé"`

`else`

`echo "Installé"`

`fi`

il suffit ensuite d’appeler ce script en lui fournissant en paramètre le nom du package voulu.
## Exercice 4
Pour lister les programmes livrés avec coreutils, on utilise `dpkg -L coreutils`, qui nous affiche l'emplacement de chacun desdits programmes. En lisant le résultat on peut donc obtenir la liste des programmes.

La commande [, contenue dans coreutils, est en fait un synonyme de la commande test, qui permet de tester une expression : au lieu de faire `test EXPRESSION`, il suffit de taper `[EXPRESSION]`, et le résultat retourné sera le même. 
## Exercice 5
On veut télécharger emacs a partir de la version graphique d'aptitude. Pour ce faire, on commence par installer aptitude : `sudo apt install aptitude`, puis par le lancer avec la commande `aptitude`. On se retrouve alors dans une interface graphique : avec /, on exécute une recherche pour chercher emacs parmi les multiples paquets installables, puis on sélectionne emacs. Il suffit ensuite de suivre les instructions, en téléchargeant ce qu'il faut avec g.
## Exercice 6 : installation d'un paquet par PPA
**1)** On commence par installer le paquet java par ppa :

`sudo add-apt-repository ppa:linuxuprising/java`

`sudo apt update`

`sudo apt install oracle-java11-installer-local`

**2** On peut alors, avec la commande `ls /etc/apt/sources.list.d`, constater la création d'un nouveau fichier : linuxuprisin-ubuntu-java-eoan.list. 

En ouvrant celui ci on observe qu'il contient 2 lignes : la premiere contient *deb* suivi d'une adresse http, la deuxieme, commentée, contient *deb-src* suivi de la même adresse.
Cette adresse correspond au lien de mise a jour du nouveau paquet.

## Exercice 7 : Création de dépôt personnalisé.
### Création d'un paquet Debian avec dpkg-deb
**1)** Dans le dossier script (créé lors d'un précédent TP), on crée un sous-dossier *origine-commande* où on place un sous-dossier *DEBIAN*, ainsi que l’arborescence suivante : usr/local/bin

`cd scripts`

`mkdir origine-commande`

`cd origine-commande`

`mkdir Debian`

`mkdir usr`

`mkdir usr/local`

`mkdir usr/local/bin`

`cd Debian`

`vim control`

**2)** Cette dernière ligne à simultanément créé et ouvert le fichier control; On y place le contenu suivant :

*Package: origine-commande #nom du paquet*

*Version: 0.1 #numéro de version*

*Maintainer: Galland-Vatin #notre nom*

*Architecture: all #les architectures cibles de notre paquet (i386, amd64...)*

*Description: Cherche l'origine d'une commande*

*Section: utils #notre programme est un utilitaire*

*Priority: optional #ce n'est pas un paquet indispensable*

**3)** On retourne alors dans le dossier parent de origine-commande `cd ~` et on exécute la commande suivante `dpkg-deb --build origine-commande`. On vient de créer un paquet.
### Création du dépôt personnel avec reprepro
**1)** Dans notre dossier personnel, on crée un dossier qui sera la racine de notre dépôt : `mkdir repo-cpe`

**2)** On y ajoute deux sous-dossiers, conf (qui contiendra la configuration du dépôt) et packages (qui contiendra nos paquets) :

`mkdir repo-cpe/conf`

`mkdir repo-cpe/packages`

**3)** Dans conf, on crée un fichier distribution :

`vim repo-cpe/conf`

dans lequel on place le texte suivant :

*Origin: Galland-Vatin CPE*

*Label: repo-cpe*

*// Suite: stable*

*Codename: cosmic # le nom de la distribution cible*

*Architectures: i386 amd64 # (architectures cibles)*

*Components: universe # (correspond à notre cas)*

*Description*

**4)** On se place dans le dossier repo-cpe :`cd repo-cpe`.

Nous avons changé entre temps de logiciel de vm, il faut réactiver la carte réseau qui s'est désactivée lors du changement : `ip link set ens33 up`.

on exécute ensuite `sudo apt install reprepro`, puis `reprepro -b . export` pour installer reprepro

**5)** On va désormais copier le paquet *origine-commande.deb* dans le dossier packages du dépôt :

`cp ../scripts/origine-commande.deb packages/  `

`cp ../scripts/origine-commande.deb`

On se place ensuite à la racine du dépôt (`cd ~/repo-cpe`), pour executer la commande suivante :

`reprepro -b . includedeb cosmic origine-commande.deb`

Suite à cette commande, notre paquet est désormais inscrit dans le dépôt.

**6)** On veut désormais indiquer à apt qu’il existe un nouveau dépôt dans lequel il peut trouver des logiciels :

`sudo touch /etc/apt/sources.list.d/repo-cpe.list `

`sudo vim /etc/apt/sources.list.d/repo-cpe.list`

Cela crée (puis ouvre dans vim) le fichier repo-cpe.list dans le  dossier /etc/apt/sources.list.d. On va remplir ce fichier avec la ligne 

*deb file:/home/Galland_Vatin/repo-cpe cosmic multiverse*

qui reprend la configuration du dépôt.

**7)** On conclut en lançant `sudo apt update`. Pour une prise en charge complète de notre dépôt, il faut maintenant signer le dépôt.
### Signature du dépôt avec GPG
**1)** On commence par créer une nouvelle paire de clés :

`gpg --gen-key  `

`Real name: Galland-Vatin  `

`Email address: clement.vatin@cpe.fr  `

`okay  `

`passphrase : a  `

On ne prends pas en compte le warning (clef insecure) et on valide.

**2)** On ajoute à la configuration du dépôt la ligne suivante : *SignWith: yes* (on passe pour cela par la commande `vim /home/repo-cpe/conf/distributions`, puis on ajoute la ligne voulue).

**3)** On ajoute maintenant la clé à notre dépôt : `reprepro--ask-passphrase -b .export`, puis on confirme avec le passphrase (a).

**4)** On ajoute maintenant la clé publique :

`gpg --export --armor clement.vatin@cpe.fr > public.key`

**5)** Enfin, On ajoute cette clé à la liste des clés fiables connues de apt : 

`sudo apt-key add public.key`

Théoriquement, le paquet est configuré, et téléchargeable. Cependant, bien que refaisant plusieurs fois l'exercice, nous ne sommes pas parvenus à télécharger le paquet. Il s'agit probablement d'une erreur entre le nom du paquet et celui de la commande.

## Exercice 8 : installation d'un logiciel à partir du code source
**1)** On commence par cloner le dépôt git : 

`git clone https://github.com/jubalh/nudoku`

Ceci permet de récupérer en local le code source du logiciel nudoku.

**2) 3) 4)** On se place dans le dossier nudoku, qui vient d’être créé (`cd nudoku`) et on lance la commande `autoreconf -i` .

On installe les divers paquets manquants, en relançant à chaque fois `autoreconf -i` .

Cependant, le problème de la macro AM_GNU_GETTEXT manquante n'est pas résolu par l'installation du paquet gettext, car les dépôts ne sont pas sur la bonne version de gettext.

En lisant le readme, on constate qu'en exécutant simplement `sudo apt install nudoku`, çà marche. 
