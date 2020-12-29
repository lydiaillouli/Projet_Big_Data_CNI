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
# ####################################################################
# ################### CONFIGURATION VARIABLES ########################
# ####################################################################
IMAGE_NAME = "bento/ubuntu-18.04"   # Image to use
MEM = 2048                          # Amount of RAM
CPU = 2                             # Number of processors (Minimum value of 2 otherwise it will not work)
MASTER_NAME="master"                # Master node name
WORKER_NBR = 1                      # Number of workers node
NODE_NETWORK_BASE = "192.168.50"    # First three octets of the IP address that will be assign to all type of nodes
POD_NETWORK = "192.168.100.0/16"    # Private network for inter-pod communication



Vagrant.configure("2") do |config|
    config.ssh.insert_key = false

    # RAM and CPU config
    config.vm.provider "virtualbox" do |v|
        v.memory = MEM
        v.cpus = CPU
    end

    # Master node config
    config.vm.define MASTER_NAME do |master|
        
        # Hostname and network config
        master.vm.box = IMAGE_NAME
        master.vm.network "private_network", ip: "#{NODE_NETWORK_BASE}.10"
        master.vm.hostname = MASTER_NAME

        # Ansible role setting
        master.vm.provision "ansible" do |ansible|
            
            # Ansbile role that will be launched
            ansible.playbook = "roles/main.yml"

            # Groups in Ansible inventory
            ansible.groups = {
                "masters" => ["#{MASTER_NAME}"],
                "workers" => ["worker-[1:#{WORKER_NBR}]"]
            }

            # Overload Anqible variables
            ansible.extra_vars = {
                node_ip: "#{NODE_NETWORK_BASE}.10",
                node_name: "master",
                pod_network: "#{POD_NETWORK}"
            }
        end
    end

    # Worker node config
    (1..WORKER_NBR).each do |i|
        config.vm.define "worker-#{i}" do |worker|

            # Hostname and network config
            worker.vm.box = IMAGE_NAME
            worker.vm.network "private_network", ip: "#{NODE_NETWORK_BASE}.#{i + 10}"
            worker.vm.hostname = "worker-#{i}"

            # Ansible role setting
            worker.vm.provision "ansible" do |ansible|

                # Ansbile role that will be launched
                ansible.playbook = "roles/main.yml"

                # Groups in Ansible inventory
                ansible.groups = {
                    "masters" => ["#{MASTER_NAME}"],
                    "workers" => ["worker-[1:#{WORKER_NBR}]"]
                }

                # Overload Anqible variables
                ansible.extra_vars = {
                    node_ip: "#{NODE_NETWORK_BASE}.#{i + 10}"
                }
            end
        end
    end
end

```

#### Description du Vagrantfile 
Pour augmenter le nombre de noeuds par exemple, nous pouvons modifier la variable **WORKER_NBR**. Les variables **CPU** et **RAM** permettent d'allouer plus ou moins de ressources à nos noeuds. 
Le fichier ci-dessus tel qu'il est configurer permet de déployer 2 noeuds (un master et un worker) avec 2 CPU ET 2048 de RAM pour chaque noeud avec un réseau privée hôte Virtualbox. Ce réseau va permettre l'accès aux noeuds Kubernetes depuis notre machine hôte.

### Ansible 

Nous avons utilisé l'outil Ansible pour automatiser l'installation et la configuration des package communs de nos noeuds.
Pour cela nous avons 3 roles ansibles : 
- **Common** : Installation et configuration des package communs de nos noeuds.
- **master** : Configuration spécifique au master Kubernetes.
- **worker** : Configuration spécifique au worker Kubernetes.

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


## Lancer le projet sur votre machine 
