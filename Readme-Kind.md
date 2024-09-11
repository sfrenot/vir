# Virtualisation avec kind
Kind est un outil de test pour monter une simulation d'un cluster sur une machine unique. C'est un des deux outils standards proposés par Kubernetes pour en comprendre le principe de fonctionnement.   

Le site de référence de kind est [ici](https://kind.sigs.k8s.io/).
Les outils pour tester ou utiliser kubernetes sont [ici](https://kubernetes.io/docs/tasks/tools/). 

## Installation de kind
L'environnement de tp est testé sur une machine linux debian bootable sur une clé. Vous avez la possibilité de travailler sous windows ou macos.   

Dans ce tp nous allons travailler sur une clé USB bootable, dont l'image est déposée [ici](https://tc-net.insa-lyon.fr/iso/).

Lorsque vous avez fini de démarrer sur une clé usb, comment est structuré votre environnement. La clé bootable est une clé débian fabriquée à partir de l'outil liveb-build. La documentation du projet se trouve [ici](https://live-team.pages.debian.net/live-manual/html/live-manual/index.en.html). 

Si vous souhaitez fabriquer une clé similaire et tester le processus de fabrication. Je vous suggère de regarder rapidement le script fourni en annexe. 

Démarrer votre machine sur la clé usb. 
Comment cela fonctionne t'il ?   

--  
Une fois votre machine démarrée sur la clé usb. Il faut tester le montage réseau. La clé contient un script de démarrage du réseau eduroam que vous pouvez tester immédiatement. Un `ping www.google.fr` vous permet de tester votre configuration réseau. Par curiosité, pouvez-vous identifier votre adresse ip ainsi que votre configuration réseau. 


# Démarrage de kind
La documentation de kind se trouve [ici](https://kind.sigs.k8s.io/)  

Vous pouvez suivre le tutorial (suivant)[https://medium.com/@talhakhalid101/creating-a-kubernetes-cluster-for-development-with-kind-189df2cb0792]

Un résumé du tutorial est le suivant : 
- Installez kind sur votre environnement : (installation)[https://kind.sigs.k8s.io/docs/user/quick-start/]
- Lancez un cluster avec un fichier de configuration minimal  suivant : 
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 30080
    hostPort: 30070
```











