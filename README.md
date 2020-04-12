# Notes taken from LinkedIn Learning
Les notes ont été prises depuis [le cours "L'essentiel de Kubernetes".](https://www.linkedin.com/learning/l-essentiel-de-kubernetes/configurer-le-cluster-on-premise?originalSubdomain=fr)

## Introduction aux conteneurs

Arrivés avec l'ascenscion du dévelopement de micro-services, avec des cycles de mise en production court et  
avec une tendance scale out plutôt que scale up, les conteneurs deviennent incontournables.  
Il permet de lancer plusieurs services isolés les uns des autres sur un même host.  
  
**Les conteneurs sont arrivés avec :**  
- **FreeBSD jails**  
- **Solaris Zones**  
- **AIX WPARs**  
- **LXC** (linuxcontainer)  
- **Docker** (s'est imposé grâce à sa solution enterprise et le support qu'il a donc apporté aux entreprises)
- **OCI** (open container initiative, fait partie de la LinuxFoundation et porté par Docker et d'autres leaders de   
la conteneurisation.) 

On retrouve dans l'OCI des specificationss comme le runtime spec, qui spécifie le runtime qui permet d'exectuer  
le conteneur. Ou encore l'image spec, qui spécifie comment doit être composé une image de type conteneur.

## Docker ? c'est quoi ?

Docker c'est le leader mondial en terme de conteneurisation. Il est aussi le runtime par défaut utilisé par  
Kubernetes, lui aussi qui s'impose comme un leader mais dans l'orchestration de conteneur. Docker s'execute  
sous la forme d'un service/daemon appelé le docker engine. Et donc la machine qui execute du docker s'appelle  
un docker host. Cet host va executer des conteneurs qui sont des instances d'execution d'images qu'on récuperera  
depuis des registry. Pour accéder à l'engine on utilise la docker cli (sous windows interface web).  
L'image est un ensemble de layers, fourni par un éditeur ou vous-même.  
Le conteneur est l'instance en cours d'execution d'une image. L'image contient les libs/binaires.  
Les applications deviennent portables (multi hosts), facile à déployer/versionner avec git par exemple.  
  
## Intro historique & commerciale de kubernetes

Bon, maintenant y a énormément de conteneurs, mais du coup il nous faut un outil pour automatiser leur   
déploiement et toutes les actions requises à leur bon fonctionnement. Kube est donc un orechestrateur  
qui va amaner une fonction de cluster permettant de manager plusieurs docker host depuis une ligne de commande. 
Il va venir avec des fonctionnalités de **scalabilité, de load balancing & d'HA facilitées**.  
Kubernetes est un projet issue de google (Borg) et va être fourni à la CNCF (cloud native computing foundation).  
Cela sera le Premier projet de la fondation. (Première version en 2015.) 
  
Il va donc nous gérer les apps conteneurisées, le déploiement massif, le monitoring de lui-même, la montée   
en charge et le rolling update. (mise à jour à chaud sans down time). 
  
Kube va gérer la partie execution ainsi que la partie configuration de applications via des fichiers yaml. 
Cela facilite la gestion d'application sous différent environnements (test, dev, staging, pprod, prod)
Il va gérer aussi des secrets et des RBAC (role based access control). Les secrets permettant de chiffrer  
certaines configurations pour les applications qu'on ne voudrait pas divulguer à tous les utilisateurs du  
cluster.  

## Intro fonctionnelle de kubernetes 

Un **cluster kubernetes c'est quoi ?** Un ensemble de noeuds (linux ou windows) fournissant des services   
définies. Soit on a un cluster Linux, soit on a un cluster Windows.  
Un **cluster kubernetes ça execute quoi ?** Des pods, et non pas des conteneurs. Un pod contient 1 ou un  
ensemble de conteneur. Les conteneurs dans le même pod possèdent la même stack réseau & stockage.  
Un **déploiement Kubernetes ? qésaco ?** Un objet Kubernetes qui gère un pod ou des pods, de l'instancier à   
moulte reprise et même de le mettre à jour.  
C'est quoi un **service Kubernetes ?** C'est différent d'un service Docker, ne confondez pas. Sur Kubernetes, un  
service sert à exposer les ports d'un pod soit inter cluster, soit extra cluster (pour des clients par exemple).  

**Pour administrer** et/ou utiliser **kubernetes** on peut utiliser :  
- La CLI : [`kubectl`](https://github.com/kubernetes/kubectl)
- Le BUI (Browser User Interface) : [`kubernetes dashboard`](https://github.com/kubernetes/dashboard)

Au sein d'un cluster Kubernetes, il y a **2 différents type de Noeuds** :  
- les **masters** = aussi appelés Kubernetes Control Plane, se charge de la gestion du cluster, ils orechestrent et execute les pods. Ils exposent l'apiserver qui est le point d'entrée du cluster. Toutes commandes kubectl intérogent l'apiserver.  
- les **workers** = il execute des pods. Il fournit des ressources pour le cluster et reçoit ses infos par le master. Ce sont les workers qui exposent les applications aux users.  


## Architecture générale de kubernetes

![kubernetes architecture](assets/architecture_k8s.png)

## Les différents composants de Kubernetes  

Les composants des masters :  
- [kube-apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) (point d'entrée du cluster, moyen de communication avec les [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/))
- [kube-scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/) (orchestrateur, répartition de charges, choix du worker adapté au pod à exécuter celon les specs...)
- [kube-controller-manager](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/) (contient un ensemble de controller, pour controler le nombre de réplicas, les accès...) 
- [etcd](https://kubernetes.io/fr/docs/concepts/overview/components/#etcd) (base de donnée (clé/valeur) contenant les états du cluster, les infos des objets déployés, et toutes les activités au sein du cluster.)
  
Les composants des workers :
- [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/) (Composant qui permet de communiquer avec les masters via l'apiserver. Il doit s'assurer que les pods s'exec comme il le devrait suivant les specs.)
- [kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/) (Gère la partie réseau et l'exposition des ports, c'est la barrière entre les utilisateurs et les pods.)
  
## Les ressources Kubernetes  
  
Il existe plusieurs catégories de ressources. Celles qui vont gérer les applications, celles qui vont gérer  
la partie Load Balancing, celles qui vont configurer les applications, celles qui vont gérer la partie  
stockage et enfin les ressources qui gèrent la configuration même du cluster.  
  
* Ressources de **gestion d'applications** :
  - **Pod** : exécute l'application dans un conteneur. Si ils crash, alors le replicaSet va s'en rendre compte et provisionner un nouveau pod. Il ne va surtout pas restart le pod ayant crash. Une des Best practice niveau application est de set 1 container par pod, pour ne pas se perdre et pouvoir découper les services plus rapidement et facilement. Ensuite une application doit générélament être composée de 1 ou plusieurs pod avec 1 pod par service pour pouvoir effectuer un scaling horizontal des services indépendemment des autres.
  - **Deployment** : défini comment on va déployer le pod et comment on va controler celui-ci
  - **ReplicaSet** : défini l'ensemble de réplicas d'un pod. C'est avant tout un controller qui doit maintenir l'état d'un pod suivant ses specs.  
  - *Example* > Au final on va avoir un déploiement qui va lancer un ReplicaSet et qui va surveiller un   
ensemble de replicas d'un pod.  
* Ressources de **gestion du LoadBalancing et du réseau** :
  - **Service** : Gère l'accès au pod soit à l'intérieur du cluster soit à l'extérieur de celui-ci.
  - **Add-on réseau** : Ce qu'on va installer pour gérer le cluster niveau réseau. 
* Ressources de **gestion de la configuration des applications** :
  - **Configmap** : Gère les configurations des applications  
  - **Secret** : Gère les credentials nécessaires pour les applications
* Ressources de **gestion du stockage** :  
  - **Persistent Volume (PV)** : Permet de créer des volumes qui sont extérieurs au container donc au pod, et qui ont un cycle de vie indépendant.  
  - **Volume Claim** : Opération de demande de PV suivant des pré requis, taille, utilisation etc...
* Ressources de **gestion de configuration du cluster** :  
  - **Metadata** : ressource utilisée par le cluster sur les différents objets
  - **Namespace** : permet de séparer les applications en vues logiques au sein du cluster
  - **Roles** : permet de gérer les roles et les droits d'accès
  
## Manipuler les ressources Kubernetes  
  
Pour manipuler ces ressources, il faut les définir et donc passer par des fichiers YAML ou JSON. Ces fichiers  
vont décrire les ressources sous forme de clés et de valeurs.  

Par exemple au format YAML (le plus répandu) :  
![nginx-pod-definition](assets/nginx-pod.png)  
  
* On peut voir différentes clés : 
  - apiVersion : spécifie la version de l'api 
  - kind :  définit le type de ressource
  - metadata : permet d'ajouter des informations au niveau du cluster sur l'objet
    - name : permet d'indiquer le nom que portera la ressource
    - label : peut permettre par la suite de faire des sélections
  - spec : spécifie comment créer le pod, comment le gérer etc..
    - containers : liste les différents containers présent dans le Pod

![nginx-pod-explaination](assets/nginx-pod-explaination.png)
  
## Déployer et installer un cluster Kubernetes  
  
* Il y a plusieurs types d'installation :  
  - Installations orientées **développement et locales** :  
    - [Minikube](https://github.com/kubernetes/minikube) > It implements local k8s cluster on Linux, Mac, Windows.  
    - [Docker Destop](https://docs.docker.com/docker-for-windows/kubernetes/) > Allow us to provision a k8s single-node cluster.  
    - [Microk8s](https://github.com/ubuntu/microk8s) > Run local, small & fast k8s cluster from a single-package.  
    - [Kind](https://github.com/kubernetes-sigs/kind) > Run local k8s cluster using docker container as nodes.  
    - [K3d](https://github.com/rancher/k3d) > Run local single-node k3s cluster using docker container as nodes.  
  - Installations orientées **production et multi environnement** :  
    - **Installations hébergées** de cluster multi-noeuds : (cluster managé par le service provider)
      - EKS > Amazon Elastic Kubernetes Service  
      - GKE > Google Kubernetes Engine
      - AKS > Azure Kubernetes Service
    - **Installations on-premise** de cluster multi-noeuds : (cluster managé par la team OPS)
      - [Kubeadm](https://github.com/kubernetes/kubeadm) > Ligne de commande facilitant la création de noeuds Kubernetes et donc de cluster Kubernetes.  
      - [Kubespray](https://github.com/kubernetes-sigs/kubespray) > Ensemble de configurations Ansible permettant de provisionner un cluster Kubernetes.  
      - [RKE](https://rancher.com/docs/rke/latest/en/) > Rancher Kubernetes Engine. Permet à l'aide d'un installer de créer un cluster Kubernetes puis de le manager depuis la Rancher UI.  
      - Docker enterprise Edition  
 
## La Pratique sur un cluster locale  

Pour pratiquer et manipuler un cluster Kubernetes localement, contrairement au cours sur LinkedIn Learning, j'ai choisi  
d'utiliser micro.k8s pour générer le cluster et le manager. Tout simplement pour des raisons d'affinité et de pratique.  
(J'ai déjà utilisé auparavant micro.k8s pour du développement autour de Kubernetes.)  
En plus de ça, microk8s est utilisable plus rapidement que minikube. Avec microk8s, la commande cliente ne sera pas  
`kubectl` **mais** `microk8s.kubectl`.  

Microk8s met à disposition un cluster à 1 seul Noeud sous Linux très rapidement et en 1 commande :  
- `sudo snap install microk8s --classic` > Installe microk8s et le cluster directement.  
- `sudo microk8s status --wait-ready` > Affiche l'avancée du lancement du cluster.   

Pour jouer et manipuler microk8s **vous allez vite avoir besoin d'alias**, sinon vous allez devenir fou.  
Voici 2 alias dont que je me sers tout le temps et que j'ai posé dans mon `~/.zshrc`.  
![microk8s-alias](assets/microk8s-alias.png)

Vous pouvez **lancer, stopper et afficher l'état de votre cluster** très rapidement.  
![manage-microk8s-cluster](assets/manage-microk8s-cluster.png)  

Ensuite vous pouvez évidemment **afficher les ressources kubernetes** présentes sur votre cluster.  
![display-mk8s-rsc](assets/display-microk8s-rsc.png)

Microk8s permet aussi de **lancer plusieurs services** avec 1 simple ligne de commande.  
Ici le kube-dashboard & le kube-dns.  
![enable-dns-dash](assets/microk8s-enable-dns-dashboard.png)



## L'essentiel de Kubernetes

On install kubeadm, on `kubeadm config images pull` pour récupérer les images
docker servant à provisionner dans pods les composants faisant fonctionner kubernetes.

- l'apiserver
- le scheduler
- le controller manager
- le kube proxy
- etcd
- et coredns

Puis on check quel `CNI` (solution de gestion du réseau dans kube) on utilise avant d'initialiser  
le cluster via la commande :  
`kubeadm init --apiserver-advertise-address=X.X.X.X --pod-network-cidr=X.X.X.X/16` (pour le CNI calico)

Ensuite on va devoir sur le master adapter le fichier de configuration de kubernetes pour utiliser la  
commande `kubectl`. 
- mkdir -p $HOME/.kube
- sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
- sudo chown $(id -u):$(id -g) $HOME/.kube/config

Maintenant on peut ajouter des pods dans le cluster kubernetes. Sachant qu'un pod est une entité managé  
par kubernetes qui représente 1 ou plusieurs conteneurs. C'est la plus petites ressources kubernetes.  

Donc on va pouvoir lancer le pod qui sera notre CNI et qui représentera la gestion du réseau au sein de  
notre cluster : `kubectl apply -f https://docs.projectcalico.org/v3.8/manifests/calico.yaml`
Pour vérifier que tout est ok : 
- `kubectl get nodes`
- `kubectl get pods | grep calico`

Ensuite si on veut rajouter dans le cluster un worker on aura juste à taper la commande `kubeadm join` qui  
nous a été donné en standard output lors de la création du master. Cela devra ressembler à quelque chose du  
style : (sur le worker évidemment)    
`kubeadm join X.X.X.X(master):6443 --token XHHEGFJHZKDBCZDJH --discovery-token-ca-cert-hash sha256:rjkergç9EJEhffje9`

On verifie avec `kubectl get no`  

## Context

Le Context défini le cluster kubernetes cible, l'utilisateur pour se connecter au cluster avec  
ses credentials et son namespace par défaut. 

Lister les context disponible dans notre kubeconfig :  
`kubectl config get-contexts`  

Par défaut pour que la commande sache quel context elle utilise un fichier à l'emplacement  
~/.kube/config (sous Linux) mais on peut changer l'emplacement en manipulant la variable d'environnement  
$KUBECONFIG. On peut donc avoir plusieurs configurations et jongler avec elles en changeant l'emplacement.  
On peut aussi spécifier à la ligne de commande `kubectl` l'argument `--kubeconfig`.  

Pour changer de context :  
`kubectl config use-context foo`
  
Pour visualiser la configuration du context actuel :  
`kubectl config view`  
  
Pour supprimer un context :  
`kubectl config delete-context foo`  

Ajouter un nouveau context depuis un autre fichier :  
`scp root@ip_master:/etc/kubernetes/admin.conf ~/.kube/`  
`export KUBECONFIG=~/.kube/config:~/.kube/admin.conf`

## Pour les motivés et ceux qui veulent aller plus loin dans Kubernetes 

Voici un repository github listant tous les projets autour de Kubernetes qui peuvent nous intéresser. Ils sont  
rangés par types de solution et par domaines. Vous y trouverez des liens pour des soltutions de **monitoring**,  
de **Loging**, de **Networking** mais aussi des solutions pour déployer des **clusters Kubernetes** et **tutos**.  
Vous trouverez aussi le CNCF landscape affichant toutes les solutions portées par la CNCF et donc ces solutions sont  
compatibles avec l'éco-système Kubernetes pour la plupart.  
  
- [List of projects around Kubernetes](https://github.com/ramitsurana/awesome-kubernetes)
- [CNCF landscape](https://landscape.cncf.io/)
