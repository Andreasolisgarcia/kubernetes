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