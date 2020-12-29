#Configuration variable#

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
