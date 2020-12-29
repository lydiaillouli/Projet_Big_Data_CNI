## Lancer le projet sur votre machine 

### Prérequis : 

Pour implémenter notre projet il faudra utiliser un systeme d'exploitation Linux (Ubuntu, Fedora, Debian...).
- Si vous êtes sous **Windows** (ce qui était notre cas), le projet pourra être exécuter sur une machine virtuel Unbuntu (par exemple) via VirtualBox.
- Si vous êtes sous **Mac** il suffit juste de télécharger VirtualBox ou bien Vmware. Attention ! Pour ceux qui sont sous le nouveau Mackbook M1, VirtualBox et WMware ne sont pas encore compatible avec cette nouvelle version. Vous pouvez néanmoins utiliser une machine virtuel Fedora pour lancer le projet. Retrouvez en cliquant <a href="https://www.youtube.com/watch?v=WQNj6WEh6pc&ab_channel=MartinNobel">ici</a> une vidéo (en anglais) qui vous montres comment installer une VM Fedora sur Mac M1. 

Dans notre cas nous avons utiliser une VM **Ubuntu 20.04** sur VirtualBox

### Installation de VirtualBox  
*Remarque: Si vous l'aviez déjà installé vous pouvez passer cette étape*

Nous allons installer Virtual box directement via notre terminal de commande Linux.

###**Étape 1: Mettre à jour votre liste de paquets disponibles.**
```
sudo apt-get update
```
###**Étape 2: Installer le paquet virtualbox.**
```
sudo apt-get install virtualbox
```
Bravo vous venez d'installaer VirtualBox sur votre machine. 

### Installation de Vagrant
Pour installer Vagrant vous avez deux méthodes : 
1.  Vous pouvez aller directement sur le site officel de Vagrant en cliquant <a href="https://www.vagrantup.com/downloads.html">ici</a>  et télécharger le package correspondant à votre système d'exploitation.
2. Si cous 

