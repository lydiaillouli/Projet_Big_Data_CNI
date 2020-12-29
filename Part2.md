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
2. Si vous êtes sous Ubuntu comme nous voici les commandes à saisir : 
Télécharger le package Vagrant:
```
curl -O https://releases.hashicorp.com/vagrant/2.2.6/vagrant_2.2.6_i686.deb
```
*Remarque : Au moment de la rédaction de cet article, la dernière version de Vagrant est la version 2.2.13. Rendez vous sur le site de Vagrant pour voir si une nouvelle version est disponible.*
Une fois le fichier téléchargé, installez-le
```
sudo apt install ./vagrant_2.2.6_i686.deb
```
Une fois l'installation terminer vous pouvez testez si tout est bien installer , en vérifiant la version de vagrant :
```
vagrant --version
```

**Résultat**

mettre la capture vagrant_version

### Installation de Ansible
Pour installer Ansible, rendez-vous sur <a href ="https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html">le site d'Ansible</a> . 

Comme nous utilisons Ubuntu nous allons l'installer de la manière suivante: 
```
sudo apt install ansible
```
Ensuite, pour savoir si l'installation c'est bien déroulée il faudra verifié la version de Ansible 
```
ansible --version
```
**Résultat**

mettre la capture ansible_version

### Éxécuter le projet 

Maintenant que nous avons installer tous les outils dont nous avons besoin, il est temps d'excuter le projet. Dans une premier temps télécharger puis le dézipper dans votre dossier. Ensuite ouvrez votre terminal de commande puis dirigez vous vers le dossier où ce trouve notre projet. Pour notre part nous avons placé le projet dans le dossier *Documents*
mettre la vagarnt_up
Une fois dessus tapez la commande suivante : 
```
vagrant up
```
Une fois l'exécution terminée, il vous suffit d'ouvrir VirtualBox et vous visualiserez les deux machines.

### Erreur possible
Au moment ou vous excecutez votre projet avec la commande *vagrant up* il est possible que vous ayez des erreurs.
Voici les erreurs auquels nous avons fait face et comment nous y avons remedier. 


**Erreur 1 : Stderr: VBoxManage: error: VT-x is not available (VERR_VMX_NO_VMX)**

Comme on souhaite ouvrir une machine virtuelle dans une machine virtuelle il est nécéssaire d'activer la case VTx/AMD imbriqué dans les paramètre de votre sytème. 

Si la case est grisé vous pouvez tapez la command suivante dans votre terminale de commande Windows : 
```
VBoxManage modifyvm LeNomDeVotreMAchine --nested-hw-virt on

```
La case sera cette fois-ci coché. Ainsi, lorsque vous allez exécuter la commande *vagrant up* l'erreur aura disparu.


