# Projet_Big_Data_CNI
## Introduction 
Un système distribué, est un système comportant de multiples composants situés sur différentes machines qui communiquent et coordonnent des actions afin d'apparaître comme un seul système cohérent pour l'utilisateur final. Dans le cadre de notre projet nous avons dévidé de mettre en application ce principe en travaillant sur les plugins CNI.

CNI (Container Network Interface) est un projet de la Cloud Native Computing Foundation regroupe des plugins qui ont pour but de configurer des interfaces réseau dans des conteneurs Linux. L'objectif principal de CNI est de faciliter la configuration réseau des conteneurs via une norme que ça soit pendant leurs créations ou lors de leurs destructions, le tout basée sur un plugin.

Les plugins mis en place par CNI ont pour but de grantir que les exigences réseau de Kubernetes sont remplies et fournissent les fonctionnalités réseau requises pour assurer le bon fonctionnement du cluster. Différents plugins existent dont le plugin flannel, qui est celui utilisé dans notre rôle.

Ces plugins permettent de s'assurer que les exigences réseau de Kubernetes sont satisfaites et fournissent les fonctionnalités réseau requises pour assurer le bon fonctionnement du cluster. Ainsi il existe plusieurs plugins dont le plugin flannel, qui est celui que nous avons utilisé dans ce projet. 

Les outils utilisés dance ce projet sont : 
- Virtualbox
- Vagrant
- Ansible

