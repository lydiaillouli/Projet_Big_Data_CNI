# Projet_Big_Data_CNI
## Introduction 
Un système distribué, est un système comportant de multiples composants situés sur différentes machines qui communiquent et coordonnent des actions afin d'apparaître comme un seul système cohérent pour l'utilisateur final. Dans le cadre de notre projet nous avons décidé de mettre en application ce principe en travaillant sur un des **plugins CNI**.

**CNI (Container Network Interface)** est un projet de la **Cloud Native Computing Foundation (CNCF)** regroupant des plugins qui ont pour but de configurer des interfaces réseau dans des conteneurs Linux. L'objectif de ce projet est de créer une norme conçue pour faciliter la configuration réseau des conteneurs que ça soit pendant leurs créations ou lors de leurs destructions, le tout basée sur un plugin.

Ces plugins permettent de s'assurer que les exigences réseau de Kubernetes sont satisfaites et fournissent les fonctionnalités réseau requises pour assurer le bon fonctionnement du cluster. Ainsi il existe plusieurs plugins dont le plugin **flannel**, qui est celui que nous avons utilisé dans ce projet.

Les outils utilisés dans ce projet sont : 
- Virtualbox v6.1.16
- Vagrant v2.2.6
- Ansible v2.9.6

## Structure du projet 

### Vagrant 
Vagrant est un logiciel libre et open-source pour la création et la configuration des environnements de développement virtuel. Il peut être considéré comme un wrapper autour de logiciels de virtualisation comme VirtualBox. Il permet de simplifier et d'automatiser la gestion de machines vituels.

C'est le fichier **Vagrantfile** qui permet de décrire comment nos VM seront configurées et déployées : 

```ruby 

IMAGE_NAME = "bento/ubuntu-18.04"   # Image Ubuntu 
MEM = 2048                          # Quantité de RAM
CPU = 2                             # Nombre de processeur ( Au minimum 2 pour que le déploiement fonctionne) 
MASTER_NAME="master"                # Nom du noeud Master 
WORKER_NBR = 1                      # Nombre de worker
NODE_NETWORK_BASE = "192.168.50"    # Les 3 premiers octets de l'IP qui sera assigné à tous les types  de noeud
POD_NETWORK = "192.168.100.0/16"    # Réseau privé pour la communication inter-pod 



Vagrant.configure("2") do |config|
    config.ssh.insert_key = false

    # config RAM / CPU
    config.vm.provider "virtualbox" do |v|
        v.memory = MEM
        v.cpus = CPU
    end

    # config du Master 
    config.vm.define MASTER_NAME do |master|
        
        # config du Nom d'hôte et réseau 
        master.vm.box = IMAGE_NAME
        master.vm.network "private_network", ip: "#{NODE_NETWORK_BASE}.10"
        master.vm.hostname = MASTER_NAME

        # Config des Roles Ansible 
        master.vm.provision "ansible" do |ansible|
            
            # Le role Ansible qui sera lancé 
            ansible.playbook = "roles/main.yml"

            # Groupes de l'inventaire Ansible
            ansible.groups = {
                "masters" => ["#{MASTER_NAME}"],
                "workers" => ["worker-[1:#{WORKER_NBR}]"]
            }

            ansible.extra_vars = {
                node_ip: "#{NODE_NETWORK_BASE}.10",
                node_name: "master",
                pod_network: "#{POD_NETWORK}"
            }
        end
    end

    #Config du Worker 
    (1..WORKER_NBR).each do |i|
        config.vm.define "worker-#{i}" do |worker|

            # config du Nom d'hôte et réseau
            worker.vm.box = IMAGE_NAME
            worker.vm.network "private_network", ip: "#{NODE_NETWORK_BASE}.#{i + 10}"
            worker.vm.hostname = "worker-#{i}"

            #Config des Roles Ansible
            worker.vm.provision "ansible" do |ansible|

                # Le role Ansible qui sera lancé 
                ansible.playbook = "roles/main.yml"

                # Groupes de l'inventaire Ansible
                ansible.groups = {
                    "masters" => ["#{MASTER_NAME}"],
                    "workers" => ["worker-[1:#{WORKER_NBR}]"]
                }

                ansible.extra_vars = {
                    node_ip: "#{NODE_NETWORK_BASE}.#{i + 10}"
                }
            end
        end
    end
end

```

#### Description du Vagrantfile 
Pour augmenter le nombre de noeuds par exemple, vous pouvez modifier la variable **WORKER_NBR**. Les variables **CPU** et **RAM** permettent d'allouer plus ou moins de ressources à nos noeuds. 
Le fichier ci-dessus tel qu'il est configuré permet de déployer 2 noeuds (un master et un worker) avec **2 CPU** et **2048 de RAM** pour chaque noeud avec un **réseau privée hôte Virtualbox**. Ce réseau va permettre l'accès aux **noeuds Kubernetes** depuis notre machine hôte.

### Ansible 

Pour ce projet nous utilisons l'outil Ansible pour automatiser l'installation et la configuration des packages communs de nos noeuds.
Pour cela nous avons 3 rôles ansibles : 

- **Common** : Installation et configuration des packages communs de nos noeuds.
- **Master** : Configuration spécifique au master Kubernetes.
- **Worker** : Configuration spécifique au worker Kubernetes.

De plus, nous avons configuré un playbook ansible **main.yml** qui est appelé par le fichier vagrant vu précedemment : 

```ruby
- hosts: masters
  become: yes
  roles:
    - { role: master}   

- hosts: workers
  become: yes
  roles:
    - { role: worker}
```

Si la machine est de type master, on lance le rôle roles/master. En revanche si la machine est de type worker, on lance le rôle roles/worker.

### Le plugin flannel 

Comme évoqué précedemment, les plugins CNI ont pour objectif de simplifier la configuration réseau des counteneurs Linux. 
Kubernetes ne fournit pas de mise en œuvre de réseau par défaut, mais définit seulement le modèle et laisse à d'autres outils le soin de le mettre en œuvre. Il existe de nombreuses implémentations de nos jours, Flannel est l'une d'entre elles.

#### Fonctionnement 
Flannel créé un **bridge cni0** sur chaque nœud et y attache des interfaces **veth**. Les **pods** ne sont **créés** que sur les minions, le master ne contient aucune instance. Chaque nœud a donc son sous-réseau du **pool flannel**. 

![MicrosoftTeams-image](https://user-images.githubusercontent.com/44178364/103305926-a8fabe00-4a0c-11eb-988f-93637b07608f.png)

#### Déploiement  
Flannel peut être ajouté à n'importe quel cluster Kubernetes existant, dans notre cas il nous a semblé plus simple de l'ajouter avant l'excution de notre projet gâce à Ansible.
Nous l'avons donc défini en tant que tâche sur notre **role master** : 

```ruby
- name: Install flannel pod network
  become: false
  command: kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

```

## Lancer le projet sur votre machine 
### Prérequis : 

Pour implémenter notre projet il faudra utiliser un système d'exploitation Linux (Ubuntu, Fedora, Debian...). 
- Si vous êtes sous **Windows** (ce qui était notre cas), le projet pourra être exécuté sur une machine virtuelle Unbuntu (par exemple) via VirtualBox.
- Si vous êtes sous **Mac** il suffit juste de télécharger VirtualBox ou bien Vmware. Attention ! Pour ceux qui sont sous le nouveau Mackbook M1, VirtualBox et WMware ne sont pas encore compatible avec cette nouvelle version. Vous pouvez néanmoins utiliser une machine virtuelle Fedora pour lancer le projet. Retrouvez en cliquant <a href="https://www.youtube.com/watch?v=WQNj6WEh6pc&ab_channel=MartinNobel">ici</a> une vidéo (en anglais) qui vous montres comment installer une VM Fedora sur Mac M1. 

Dans notre cas nous avons utilisé une VM **Ubuntu 20.04** sur VirtualBox

### Installation de VirtualBox  
*Remarque: Si vous l'aviez déjà installé vous pouvez passer cette étape*

Nous allons installer VirtualBox directement via notre terminal de commande Linux.

**Étape 1: Mettre à jour votre liste de paquets disponibles.**
```
sudo apt-get update
```
**Étape 2: Installer le paquet virtualbox.**
```
sudo apt-get install virtualbox
```
Bravo vous venez d'installer VirtualBox sur votre machine. 

### Installation de Vagrant
Pour installer Vagrant vous avez deux méthodes : 
1.  Vous pouvez aller directement sur le site officel de Vagrant en cliquant <a href="https://www.vagrantup.com/downloads.html">ici</a>  et télécharger le package correspondant à votre système d'exploitation.
2. Si vous êtes sous Ubuntu comme nous voici les commandes à saisir :

Télécharger le package Vagrant:
```
curl -O https://releases.hashicorp.com/vagrant/2.2.6/vagrant_2.2.6_i686.deb
```
Une fois le fichier téléchargé, installez-le
```
sudo apt install ./vagrant_2.2.6_i686.deb
```
Une fois l'installation terminer vous pouvez testez si tout est bien installer , en vérifiant la version de vagrant :
```
vagrant --version
```

**Résultat**

![vagrant_version](https://user-images.githubusercontent.com/44178364/103303903-75696500-4a07-11eb-957a-563028dda033.PNG)

### Installation de Ansible
Pour installer Ansible, rendez-vous sur <a href ="https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html">le site d'Ansible</a> . 

Comme nous utilisons Ubuntu nous allons l'installer de la manière suivante: 
```
sudo apt install ansible
```
Ensuite, pour savoir si l'installation s'est bien déroulée il faudra verifier la version de Ansible 
```
ansible --version
```
**Résultat**

![ansible_version](https://user-images.githubusercontent.com/44178364/103303783-291e2500-4a07-11eb-9375-02b7b7aa1db3.PNG)

### Éxécuter le projet 

Maintenant que nous avons installé tous les outils dont nous avons besoin, il est temps d'éxécuter le projet. Dans un premier temps télécharger le projet puis le dézipper dans votre dossier. Ensuite ouvrez votre terminale de commande puis dirigez-vous vers le dossier où se trouve notre projet. Pour notre part nous avons placé le projet dans le dossier *Documents*

![vagrant_up](https://user-images.githubusercontent.com/44178364/103303952-a053b900-4a07-11eb-9338-695881337d51.PNG)

Une fois dessus tapez la commande suivante 
```
vagrant up
```
![vagrant2](https://user-images.githubusercontent.com/44178364/103304169-2bcd4a00-4a08-11eb-9963-537978ef31e2.PNG)

Une fois l'exécution terminée, il vous suffit d'ouvrir VirtualBox et vous visualiserez les deux machines.

### Erreur possible
Au moment ou vous excecutez votre projet avec la commande *vagrant up* il est possible que vous ayez des erreurs.
Voici les erreurs auquels nous avons fait face et comment nous y avons remedier. 



**Erreur 1 : Stderr: VBoxManage: error: VT-x is not available (VERR_VMX_NO_VMX)**



<img width="827" alt="Erreur1" src="https://user-images.githubusercontent.com/44178364/103304416-df363e80-4a08-11eb-9b73-3d5d3b0789ab.png">

Si vous utilisez une VM virtualbox comme machine hôte (comme nous) il est nécéssaire d'activer la case VTx/AMD imbriqué dans les paramètres de votre sytème. 

<img width="675" alt="VTx" src="https://user-images.githubusercontent.com/44178364/103304303-8d8db400-4a08-11eb-8731-f0d58551cf94.png">

Si la case est grisée vous pouvez taper la commande suivante dans votre terminale de commande Windows : 
```
VBoxManage modifyvm LeNomDeVotreMAchine --nested-hw-virt on
```
La case sera cette fois-ci coché. Ainsi, lorsque vous allez exécuter la commande *vagrant up* l'erreur aura disparu.

### Conclusion

Nous vous avons présenté notre projet ainsi que les étapes qui vous permetterons de l'implémenter sur votre machine. Si vous avez des questions n'hésitez pas à nous contacter sur nos adresses emails : 

mohamed.benbouzid@edu.ece.fr
pierre.dacostafaro@edu.ece.fr
lydia.illouli@edu.ece.fr

