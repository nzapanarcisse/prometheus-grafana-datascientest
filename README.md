# prometheus-training
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

**installation prometheus***



# I. Monitoring du matériel et du système d'exploitation
