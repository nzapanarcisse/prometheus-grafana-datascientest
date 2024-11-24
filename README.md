# Prometheus & Grafana training
Aujourd'hui, les systèmes d'information doivent gérer un nombre croissant de composants à surveiller et à administrer dans des environnements Cloud-Native (stacks, outils, etc.), où les pratiques DevOps sont largement promues. Avec l'émergence des microservices (Docker, Kubernetes), les besoins en matière de monitoring ont évolué et se répartissent en quatre grandes catégories :

1. Monitoring du matériel et du système d'exploitation
2. Monitoring du cluster (nœud de contrôle et nœud de travail)
3. Monitoring des conteneurs/pods
4. Monitoring des applications/services

Le **monitoring** est indispensable au sein du SI afin de faire la maintenance Pro-active et réaliser des audits techniques sur l’infrastructure. Ainsi les DSI doivent au plus vite arrimer leur méthode de supervision des infrastructures actuelles qui se veulent dynamiques, éphémères et élastiques.

Dans cette **formation** nous apportons une solution concrète à cette problématique en partant d’un projet **(une situation concrète d’un client)** pour déboucher sur la supervision de son infrastructure faisant essentiellement du micro-service à l’aide de solution on-premise **(Prometheus + grafana)**

**Prérequis:**

1. avoir de bonnes bases sur Docker 
2. avoir de bonnes bases sur kubernetes


## Lab-1 (installation odoo sur le cluster via helm3)
**lien chart odoo** https://github.com/helm/charts/tree/master/stable/odoo
**installation chart odoo**
```bash
 cd lab-1
 helm repo add bitnami https://charts.bitnami.com/bitnami
 helm repo update
 helm search repo odoo
```
installation odoo
```bash
 kubectl create namespace monitoring
 helm install pg-postgres-odoo-datascientest bitnami/postgresql -f values_postgres.yaml -n monitoring
 helm install odoo-datascientest bitnami/odoo -f values.yaml -n monitoring
 kubectl get all -n monitoring

 helm uninstall odoo-datascientest -n monitoring
 helm uninstall pg-postgres-odoo-datascientest -n monitoring
```
1. **Obtenez l'URL d'Odoo en exécutant les commandes suivantes :**

```bash
   export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services odoo-datascientest)
   export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
   echo "Odoo URL: http://$NODE_IP:$NODE_PORT/"
```
2. **Obtenez les identifiants de connexion:**
```bash
   export ODOO_EMAIL=user@example.com
export ODOO_PASSWORD=$(kubectl get secret --namespace "default" odoo-datascientest -o jsonpath="{.data.odoo-password}" | base64 -d)

echo Email   : $ODOO_EMAIL
echo Password: $ODOO_PASSWORD
```

## INSTALLATION PROMETHEUS ET GRAFANA

**installation Prometheus***

 1- **Accédez au répertoire du projet  et exécuté les commandes:**
  
  **config-map.yaml**
  - Le ficher **config-map.yaml** définit la configuration de Prometheus.
  
  - **scrape_interval** et *evaluation_interval* : Définissent la fréquence à laquelle Prometheus collecte des métriques et évalue les règles.

  - **scrape_configs** : Définit les cibles à surveiller. Ici, Prometheus est configuré pour surveiller lui-même.

  - Le **scrape_interval** de 20 secondes permet une collecte fréquente des données.
  - La cible **localhost:9090** indique que Prometheus surveille son propre endpoint.
  **prometheus-deployment.yaml**
    - **replicas** : Indique le nombre d'instances de Prometheus à exécuter. Ici, une seule instance est déployée.
    - **containers** : Définit le conteneur Prometheus, y compris l'image et les arguments pour spécifier le fichier de configuration et le chemin de stockage.
    - Le **volumeMount** pour prometheus-config-volume permet à Prometheus d'accéder à sa configuration via le ConfigMap

 **clusterRole.yaml**
 - Le fichier clusterRole.yaml définit les permissions nécessaires pour que Prometheus puisse surveiller le cluster Kubernetes. Il spécifie les ressources auxquelles Prometheus peut accéder et les actions qu'il peut effectuer sur ces ressources.

- Ressources et permissions : Le rôle accorde à Prometheus l'accès à des ressources critiques comme les nœuds, les services, les pods et les points d'ingress. Cela lui permet de collecter des métriques sur l'état du cluster et des applications déployées.
  
- Accès non basé sur les ressources : En plus des ressources Kubernetes, le rôle permet également l'accès à des URLs spécifiques, comme celles fournissant des métriques, ce qui est essentiel pour la collecte de données par Prometheus.
- Liaison de rôle : Le fichier inclut également une liaison de rôle qui associe ce rôle à un compte de service, garantissant ainsi que Prometheus peut utiliser ces permissions lorsqu'il s'exécute dans le namespace approprié.
  
- **ClusterRole** : Définit les permissions nécessaires pour que Prometheus puisse interroger les ressources Kubernetes.
- **rules** : Spécifie les ressources (telles que les pods et les services) que Prometheus peut lire.
- Le **ClusterRoleBinding** lie le ClusterRole au compte de service par défaut, permettant à Prometheus d'accéder aux métriques et aux ressources nécessaires pour la surveillance.
  
```bash
  cd source/prometheus
  kubectl create namespace monitoring
  kubectl apply -f config-map.yaml
  kubectl apply -f prometheus-deployment.yaml -n monitoring
  kubectl apply -f prometheus-service.yaml -n monitoring
  kubectl get all -n monitoring
  kubectl apply -f clusterRole.yaml
```
entrez l'url **http://185.185.83.44:30000/** ou **185.185.83.44** est l'adresse ip de votre serveur. ceci bous permettra d'accéder à prometheus.
![image](https://github.com/user-attachments/assets/5010e7b1-0e90-4088-a436-fec22680c2ab)


Vous pouvez aller dans **status/targets** pour voir qu'effectivement Prometheus récupère ses propres métriques comme configuré.

![image](https://github.com/user-attachments/assets/231aec6a-b75d-4ea3-95eb-414b2999668d)

**installation Grafana avec Helm***

1. **Ajouter le dépôt de Helm pour Grafana** :
```bash
   helm repo add grafana https://grafana.github.io/helm-charts
   helm repo update
```
2. **ce placer dans le dossier sources/grafana du projet** :
```bash
   helm install grafana-dashboard -f values2.yaml grafana/grafana --version 8.0.0
   #pour avoir le mot de passe admin
   kubectl get secret --namespace default grafana-dashboard -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```
3. **Accéder à Grafana** :
   Connectez-vous à votre instance Grafana (tableau de bord Grafana) en utilisant le port du service de type NodePort du fichier values (30007) avec l'URL : **http://185.185.83.44:30007/** dans notre cas.
![image](https://github.com/user-attachments/assets/d78b778e-465b-4a22-9fea-2764bb481c5c)

4. **configuration de datasource** :

   Dans Grafana, **une source de données** est une connexion à un système qui stocke des données que vous souhaitez visualiser et analyser. Grafana prend en charge plusieurs types de sources de données, dont Prometheus, qui est un système de monitoring et d'alerte largement utilisé pour collecter des métriques.

5. **Ajouter la source de données :**
   - Dans le tableau de bord Grafana, allez dans le menu latéral et sélectionnez Configuration (l'icône de l'engrenage).
   - Cliquez sur Sources de données.
   - Cliquez sur Ajouter une source de données.
   - Dans la liste des sources de données disponibles, choisissez Prometheus
   - Dans le champ URL, entrez l'URL de votre instance Prometheus. Par exemple : **http://prometheus-service.monitoring.svc:8090.** dans notre cas
   - Cliquez sur le bouton Tester et enregistrer pour vérifier que Grafana peut se connecter à Prometheus avec l'URL fournie.
   - Si la connexion est réussie, vous pouvez enregistrer la configuration.

     ![image](https://github.com/user-attachments/assets/f3b0112e-0903-40ba-ac82-2c232098d957)


     ![image](https://github.com/user-attachments/assets/9925ac86-2a9b-4b8a-b95c-7336994848d8)

Ajoutons un tableau de bord remarquable proposé par la communauté, qui vous permettra de visualiser les données de Prometheus. Ce tableau de bord est accessible via le lien suivant :**https://grafana.com/grafana/dashboards/3662**

cd dasbord est conçu pour fournir une vue d'ensemble des métriques de votre serveur Prometheus. Il permet de suivre des indicateurs clés de performance, tels que :

- **Santé du serveur :** Surveillez l'état général de votre instance Prometheus.
- **Statistiques de requêtes :** Visualisez le nombre de requêtes traitées, les temps de réponse, et d'autres métriques pertinentes.
- **Erreurs et alertes :** Identifiez rapidement les problèmes potentiels grâce aux graphiques d'erreurs et d'alerte.
- **installation du dashbord**
Copiez **l'ID(3662)** du tableau de bord pour l'importer dans Grafana. Ensuite, dans votre instance Grafana, créez un nouveau tableau de bord et ajoutez l'ID que vous avez copié.
   ![image](https://github.com/user-attachments/assets/4540c2a5-0db7-4307-8942-7d790a15b709)


# Bravo ! 🎉

# Bravo ! 🎉
# I. Monitoring du matériel et du système d'exploitation

**definition d'un exporteur**

**Un exporteur dans Prometheus** est un composant qui collecte des métriques d'une application, d'un service ou d'un système, puis les expose dans un format que Prometheus peut comprendre et récupérer. Les exporteurs sont essentiels pour surveiller des systèmes qui n'ont pas d'intégration native avec Prometheus.

**Prometheus Node Exporter**
Le Prometheus Node Exporter est un des exporteurs les plus couramment utilisés. Il est spécifiquement conçu pour collecter des métriques sur les ressources système d'un serveur, telles que :

- CPU : Utilisation du processeur, fréquence, etc.
- Mémoire : Utilisation de la mémoire, mémoire libre et utilisée, etc.
- Disque : Utilisation des disques, performance des entrées/sorties, etc.
- Réseau : Statistiques sur le trafic réseau, erreurs, etc.
  
1.**installation Node Exporter**
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install node-exporter prometheus-community/prometheus-node-exporter
helm search repo prometheus-community
kubectl get all
```

2.**configuration Node Exporter**

```bash
cd ../lab-4
#pour que prometheus puisse prendre en compte la nouvelle configuration
kubectl delete -f ../sources/prometheus/config-map.yaml -n monitoring
kubectl delete -f ../sources/prometheus/prometheus-deployment.yaml -n monitoring
kubectl apply -f config-map.yaml
kubectl apply -f prometheus-deployment.yaml -n monitoring
```

Accédez à prometheus/targets pour voir le Node Exporter. Cliquez sur Prometheus et effectuez une recherche pour découvrir les métriques disponibles grâce à notre exporteur.
![image](https://github.com/user-attachments/assets/1e7383c3-42cf-49f6-b5fb-7d0b24edb56b)


3.**ajout Node Exporter à Grafana**
La communauté a déjà mis à notre disposition plusieurs tableaux de bord pour Node Exporter. Ces tableaux de bord fournissent de nombreuses informations sur les nœuds de notre cluster. Vous pouvez les consulter ici :

**https://grafana.com/grafana/dashboards/1860**
Pour importer un tableau de bord, copiez l'ID et renseignez-le dans Grafana via le menu Tableau de bord > Nouveau tableau de bord > Importer un tableau de bord.


![image](https://github.com/user-attachments/assets/a995b554-0c1b-4cec-95ca-df8d46bb76c2)


# Bravo ! 🎉

# Bravo ! 🎉



