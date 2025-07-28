# Déploiement PHPBB avec Helm

## 📋 Objectif

Déployer un forum PHPBB sur Kubernetes avec les spécifications suivantes :
- ✅ Forum PHPBB accessible via NodePort
- ✅ Base de données MySQL avec volume persistant de 10Gi
- ✅ Déploiement dans un namespace dédié "forum"
- ✅ Utilisation de Helm pour la gestion
- ✅ Auto-scaling avec HPA (Horizontal Pod Autoscaler)

## 🚀 Prérequis

- Cluster Kubernetes fonctionnel (k3s, minikube, etc.)
- Helm installé
- kubectl configuré

## 📦 Étape 1 : Configuration du repository Helm

### Ajouter le repository DataScientest
```bash
helm repo add datascientest http://dst-hart-museum.datascientest.com:8080
```

### Vérifier l'ajout du repository
```bash
helm repo list
```

### Mettre à jour les repositories
```bash
helm repo update
```

### Vérifier les charts disponibles
```bash
helm search repo datascientest
```

Le chart `datascientest/phpbb` devrait être dans dans la liste.

## 🔍 Étape 2 : Exploration du chart PHPBB

### Examiner les valeurs par défaut
```bash
helm show values datascientest/phpbb
```

### Voir les informations du chart
```bash
helm show chart datascientest/phpbb
```

### Prévisualiser les templates (optionnel)
```bash
helm template datascientest/phpbb
```

## 📁 Étape 3 : Préparation de l'environnement

### Créer le répertoire de travail
```bash
mkdir forum
cd forum
```

### Créer le fichier de configuration personnalisé
```bash
nano values.yaml
```

**Contenu du fichier `values.yaml` :**
```yaml
# Configuration PHPBB
phpbb:
  service:
    type: NodePort  # Pour accès externe
    port: 80

# Configuration MySQL  
mysql:
  pvc:
    storage: 10Gi           # Volume de 10Gi comme demandé
    storageClassName: local-path  # Classe de stockage (ajuster selon votre cluster)
  service:
    type: ClusterIP         # Accès interne uniquement
    port: 3306

# Nombre de replicas initial
replicaCount: 2
```

## 🚀 Étape 4 : Déploiement PHPBB

### Installation avec Helm
```bash
helm install phpbb-forum datascientest/phpbb -n forum --create-namespace --values=values.yaml
```

### Vérifier l'installation
```bash
helm list -n forum
```

### Vérifier les ressources créées
```bash
kubectl get all -n forum
```

### Vérifier les volumes persistants
```bash
kubectl get pv,pvc -n forum
```

## 🌐 Étape 5 : Accès à l'application

### Obtenir le port NodePort
```bash
kubectl get service -n forum
```

### Obtenir l'IP du cluster
```bash
kubectl get nodes -o wide
```

### Accéder à PHPBB
Ouvrez votre navigateur à l'adresse : `http://<IP_DU_CLUSTER>:<NODE_PORT>`

Exemple : `http://192.168.1.100:32080`



## 📈 Étape 6 : Configuration de l'auto-scaling (HPA)

### Créer le fichier HPA
```bash
touch hpa.yaml
nano hpa.yaml
```

**Contenu du fichier `hpa.yaml` :**
```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-phpbb
  namespace: forum
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: phpbb-forum  # Nom du déploiement (vérifier avec kubectl get deploy -n forum)
  minReplicas: 5
  maxReplicas: 15
  targetCPUUtilizationPercentage: 50
```

### Appliquer la configuration HPA
```bash
kubectl apply -f hpa.yaml
```

### Vérifier le HPA
```bash
kubectl get hpa -n forum
```

### Voir les détails du HPA
```bash
kubectl describe hpa hpa-phpbb -n forum
```

## 🔧 Commandes de gestion utiles

### Voir les logs des pods
```bash
# Logs du pod PHPBB
kubectl logs -n forum deployment/phpbb-forum

# Logs de MySQL
kubectl logs -n forum statefulset/phpbb-mariadb
```

### Mettre à jour la configuration
```bash
# Modifier values.yaml puis :
helm upgrade phpbb-forum datascientest/phpbb -n forum --values=values.yaml
```

### Voir l'historique des déploiements
```bash
helm history phpbb-forum -n forum
```

### Rollback en cas de problème
```bash
helm rollback phpbb-forum 1 -n forum
```

## ✅ Validation finale

Une installation réussie devrait montrer :

1. **Helm release déployée :**
   ```bash
   $ helm list -n forum
   NAME         NAMESPACE  REVISION  STATUS    CHART          APP VERSION
   phpbb-forum  forum      1         deployed  phpbb-x.x.x    x.x.x
   ```

2. **Tous les pods en état Running :**
   ```bash
   $ kubectl get pods -n forum
   NAME                           READY   STATUS    RESTARTS   AGE
   phpbb-forum-xxxxxxxxx-xxxxx    1/1     Running   0          5m
   phpbb-mariadb-0                1/1     Running   0          5m
   ```

3. **Service accessible :**
   ```bash
   $ kubectl get svc -n forum
   NAME                TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)
   phpbb-forum         NodePort   10.43.x.x     <none>        80:3xxxx/TCP
   phpbb-mariadb       ClusterIP  10.43.x.x     <none>        3306/TCP
   ```

4. **PHPBB accessible dans le navigateur** à l'adresse `http://<NODE_IP>:<NODE_PORT>`

## 🐛 Dépannage

### Vérifier l'état des pods
```bash
kubectl get pods -n forum -o wide
```

### Décrire un pod en erreur
```bash
kubectl describe pod <nom-du-pod> -n forum
```

### Vérifier les événements du namespace
```bash
kubectl get events -n forum --sort-by='.metadata.creationTimestamp'
```

### Tester la connectivité réseau
```bash
# Test depuis un pod temporaire
kubectl run test-pod --image=busybox -n forum --rm -it --restart=Never -- nslookup phpbb-forum
```



## 🗑️ Nettoyage

### Supprimer le HPA
```bash
kubectl delete -f hpa.yaml
```

### Supprimer l'installation PHPBB
```bash
helm uninstall phpbb-forum -n forum
```

### Supprimer le namespace (optionnel)
```bash
kubectl delete namespace forum
```

### Supprimer le repository (optionnel)
```bash
helm repo remove datascientest
```

## 📚 Ressources supplémentaires

- [Documentation Helm](https://helm.sh/docs/)
- [Documentation Kubernetes HPA](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
- [PHPBB Official Documentation](https://www.phpbb.com/support/docs/)

---

*README créé pour l'exercice de déploiement PHPBB avec Helm et Kubernetes*