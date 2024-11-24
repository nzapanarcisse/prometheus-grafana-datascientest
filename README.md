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
```
3. **Accéder à Grafana** :
   Connectez-vous à votre instance Grafana (tableau de bord Grafana) en utilisant le port du service de type NodePort du fichier values (30007) avec l'URL : **http://185.185.83.44:30007/** dans notre cas.

4. **configuration de datasource** :

   Dans Grafana, **une source de données** est une connexion à un système qui stocke des données que vous souhaitez visualiser et analyser. Grafana prend en charge plusieurs types de sources de données, dont Prometheus, qui est un système de monitoring et d'alerte largement utilisé pour collecter des métriques.

- **Ajouter la source de données :**
   - Dans le tableau de bord Grafana, allez dans le menu latéral et sélectionnez Configuration (l'icône de l'engrenage).
   - Cliquez sur Sources de données.
   - Cliquez sur Ajouter une source de données.
   - Dans la liste des sources de données disponibles, choisissez Prometheus
   - Dans le champ URL, entrez l'URL de votre instance Prometheus. Par exemple : **http://prometheus-service.monitoring.svc:8090.** dans notre cas
   - Cliquez sur le bouton Tester et enregistrer pour vérifier que Grafana peut se connecter à Prometheus avec l'URL fournie.
   - Si la connexion est réussie, vous pouvez enregistrer la configuration.
   - 
   
# I. Monitoring du matériel et du système d'exploitation
