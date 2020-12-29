# Projet_Big_Data_CNI
## Introduction 
Un système distribué, est un système comportant de multiples composants situés sur différentes machines qui communiquent et coordonnent des actions afin d'apparaître comme un seul système cohérent pour l'utilisateur final. Dans le cadre de notre projet nous avons décidé de mettre en application ce principe en travaillant sur un des **plugins CNI**.

**CNI (Container Network Interface)** est un projet de la **Cloud Native Computing Foundation (CNCF)** regroupant des plugins qui ont pour but de configurer des interfaces réseau dans des conteneurs Linux. L'objectif de ce projet est de créer une norme conçue pour faciliter la configuration réseau des conteneurs que ça soit pendant leurs créations ou lors de leurs destructions, le tout basée sur un plugin.

Ces plugins permettent de s'assurer que les exigences réseau de Kubernetes sont satisfaites et fournissent les fonctionnalités réseau requises pour assurer le bon fonctionnement du cluster. Ainsi il existe plusieurs plugins dont le plugin **flannel**, qui est celui que nous avons utilisé dans ce projet.

Les outils utilisés dans ce projet sont : 
```
-Virtualbox v6.1.16
-Vagrant v2.2.13
-Ansible v2.10.3
```
## Structure du projet 

### Vagrant 
Vagrant est un logiciel libre et open-source pour la création et la configuration des environnements de développement virtuel. Il peut être considéré comme un wrapper autour de logiciels de virtualisation comme VirtualBox.Il permet de simplifier et d'automatiser la gestion de machines vituels.

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
Le fichier ci-dessus tel qu'il est configurer permet de déployer 2 noeuds (un master et un worker) avec **2 CPU** et **2048 de RAM** pour chaque noeud avec un **réseau privée hôte Virtualbox**. Ce réseau va permettre l'accès aux **noeuds Kubernetes** depuis notre machine hôte.

### Ansible 

Vous pouvez l'outil Ansible pour automatiser l'installation et la configuration des package communs de nos noeuds.
Pour cela nous avons 3 roles ansibles : 

- **Common** : Installation et configuration des package communs de nos noeuds.
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

### Le plugin flannel 

Comme évoqué précedemment, les plugins CNI ont pour objectif de simplifier la configuration réseau des counteneurs Linux. 
Kubernetes ne fournit pas de mise en œuvre de réseau par défaut, mais définit seulement le modèle et laisse à d'autres outils le soin de le mettre en œuvre. Il existe de nombreuses implémentations de nos jours, Flannel est l'une d'entre elles.

#### Fonctionnement 
Flannel créé un **bridge cni0** sur chaque nœud et y attache des interfaces **veth**. Les **pods** ne sont **créés** que sur les minions, le master ne contient aucune instance. Chaque nœud a donc son sous-réseau du **pool flannel**. 


#### Déploiement  
Flannel peut être ajouté à n'importe quel cluster Kubernetes existant, dans notre cas il nous a semblé plus simple de l'ajouter avant l'excution de notre projet gâce à ansible.
Nous l'avons donc défini en tant que tâche sur notre **role master** : 

```ruby
- name: Install flannel pod network
  become: false
  command: kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

```

## Lancer le projet sur votre machine 
