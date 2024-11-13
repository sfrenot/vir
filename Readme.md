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

Vous pouvez suivre le tutorial [suivant](https://medium.com/@talhakhalid101/creating-a-kubernetes-cluster-for-development-with-kind-189df2cb0792)
Ou celui-ci qui va plus loin [autre](https://medium.com/@martin.hodges/using-kind-to-develop-and-test-your-kubernetes-deployments-54093692c9fa)


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
- role: worker
- role: worker
```

La commande pour créer le cluster : 
`kind create cluster --config ./configuration.yaml`

Vérifiez que vous avez vos nodes et vos pods. 

`kubctl get nodes`   
`kubectl get pods`   
`kubectl get pods -A`   

- Déclarer une configuration de déployement de noeuds nginx. 
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: web-nginx
  name: web-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-nginx
  template:
    metadata:
      labels:
        app: web-nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
```

kubectl create -f ~/cours/VIR/vir/kindDeployment.yaml 

Testez que votre déployement s'est bien déroulé. 
`podman top 7e29d612b8e4`

Connectez vous sur un des noeuds 
`podman exec -it 7e29d612b8e4 bash ou docker exec -it 7e29d612b8e4 -- bash`

- Déclarer une configuration de service permettant de rendre accessible le service à l'extérieur. 
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-nginx
spec:
  selector:
    app: web-nginx
  type: NodePort
  ports:
    - port: 80
      nodePort: 30080
```

Essayez de modifier la page web affichée et vérifier qu'il y a un roulement sur les instances déployées. 

Si vous faites un `curl http://localhost:30070` votre accès fait :   
30070 -> 30080 -> 80   
Web -> kind -> conteneur    

```bash
get pods   
ps docker   
docker exec -it <xxxx> bash

crictl ps
crictl exec -it <xxxx> bash

cd /usr/share/nginx/html
echo "coucou" > index.html
```


# Test deploiement kind avec image docker user defined
[ici](https://medium.com/@martin.hodges/using-kind-to-develop-and-test-your-kubernetes-deployments-54093692c9fa)
[github](https://github.com/MartinHodges/basic-kind-cluster)

1- Créer un cluster à partir de la description suivante
```yaml
apiVersion: kind.x-k8s.io/v1alpha4
kind: Cluster
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 30000
    hostPort: 30000
    listenAddress: "0.0.0.0" # Optional, defaults to "0.0.0.0"
    protocol: tcp # Optional, defaults to tcp
  - containerPort: 31321
    hostPort: 31321
  - containerPort: 31300
    hostPort: 31300
- role: worker
- role: worker
```
`kind create cluster --name my-k8s --config ./kind-config.yaml`

Vérifier si le cluster est actif : `kubectl get pods -n kind-my-k8s -A`
Vérifier les nodes : `kubectl get nods`

2- Créer votre propre image docker
`mkdir -p sample-app/files`

`vi sample-app/files/index.html`
````html
<html>
  <head>
    <title>Dockerfile</title>
  </head>
  <body>
    <div class="container">
      <h1>My App</h1>
      <h2>This is my first app</h2>
      <p>Bonjour tout le monde</p>
    </div>
  </body>
</html>
````

`vi sample-app/files/default`
```
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /usr/share/nginx/html;
    index index.html index.htm;

    server_name _;
    location / {
        try_files $uri $uri/ =404;
    }
}
```

`vi sample-app/Dockerfile`

````dockerfile
FROM ubuntu:24.04
RUN  apt -y update && apt -y install nginx
COPY files/default /etc/nginx/sites-available/default
COPY files/index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["/usr/sbin/nginx", "-g", "daemon off;"]
````


3- Construire l'image docker et la charger dans le registre du cluster
```bash
cd sample-app
docker build -t sample-app:1.0 .
kind load docker-image sample-app:1.0 --name my-k8s
```

`kubectl create namespace sample-app  
`vi config.yml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-app
  namespace: sample-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
      - name: sample-app
        image: sample-app:1.0
        imagePullPolicy: Never
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: sample-app-svc
  namespace: sample-app
spec:
  selector:
    app: sample-app
  type: NodePort
  ports:
  - name: http
    port: 80
    targetPort: 80
    nodePort: 30000
```

Déployer et déclarer le service.
`kubectl apply -f ./KindDeploymentService.yaml`

Vérifiez le déploiement et le service
`kubectl get services`
`kubectl get pods`

Tester que cela fonctionne.


### Reinstall en cas de besoin de k3s
```shell
k3s-killall.sh
k3s-uninstall.sh
curl https://get.k3s.io -o install-k3s.sh
INSTALL_K3S_SKIP_ENABLE=true INSTALL_K3S_EXEC=--snapshotter=fuse-overlayfs ./install-k3s.sh
```

### Installation plusieurs noeuds
Booter sur la clé usb : https://tc-net.insa-lyon.fr/iso/k3s/k3s-tc.iso  

Sur une machine qui sera le serveur / controleur :   
!! ATTENTION L'ORDRE DES PARAMETRES PEUT JOUER

- Vérifier la date : `date` --> Changer la date si besoin avec la commande `date <memeordrequeaffichage>`  
- Monter le réseau : `./startnet.sh`
- Récuperer l'IP : `ip a`
- Lancer le master kube : `k3s server --snapshotter fuse-overlayfs --token <toto>`
- Lancer un agent : `k3s agent --server https://<x.x.x.x>:6443 --token <toto> --with-node-id <agentX> --snapshotter fuse-overlayfs`

