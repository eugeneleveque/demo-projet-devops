# Guide des Commandes du Projet (Docker, Kubernetes & Helm)

Ce document regroupe toutes les commandes utiles pour lancer et gérer le projet dans les différents environnements.

---

## 🐳 Docker Compose

### Mode Développement (`docker-compose.yml`)

- **Construire et démarrer les conteneurs en arrière-plan :**
  ```bash
  docker-compose up -d --build
  ```

- **Voir les logs des conteneurs :**
  ```bash
  docker-compose logs -f
  ```

- **Arrêter et supprimer les conteneurs :**
  ```bash
  docker-compose down
  ```

### Mode Production (`docker-compose.prod.yml`)

- **Construire et démarrer en production :**
  ```bash
  docker-compose -f docker-compose.prod.yml up -d --build
  ```

- **Arrêter l'environnement de production :**
  ```bash
  docker-compose -f docker-compose.prod.yml down
  ```

---

## ☸️ Kubernetes (Fichiers YAML statiques)

Si vous souhaitez utiliser les anciens fichiers statiques du dossier `k8s/` :

- **Déployer toutes les ressources (Services et Deployments) :**
  ```bash
  kubectl apply -f ./k8s
  ```

- **Voir l'état des Pods :**
  ```bash
  kubectl get pods
  ```

- **Voir l'état des Services (pour récupérer les ports IP/NodePort) :**
  ```bash
  kubectl get services
  ```

- **Afficher les logs d'un pod spécifique :**
  ```bash
  kubectl logs -f <nom-du-pod>
  ```

- **Supprimer tous les déploiements du dossier `k8s/` :**
  ```bash
  kubectl delete -f ./k8s
  ```

---

## 🚢 Helm (Recommandé)

Le déploiement via Helm nécessite que la base de données soit en cours d'exécution et que les images Docker soient construites localement avec les bons paramètres.

### 1. Préparation de l'environnement

- **Démarrer la base de données PostgreSQL (requise pour tasks-service) :**
  ```bash
  docker-compose up -d postgres
  ```

- **Construire les images Docker pour Kubernetes :**
  *Pour le frontend, il est indispensable de préciser l'URL des APIs K8s (NodePorts 30001 et 30002) lors de la construction.*
  ```bash
  docker build --build-arg VITE_TASKS_API_URL=http://localhost:30001/api/tasks --build-arg VITE_STATS_API_URL=http://localhost:30002/api/stats -t demo-projet-devops-frontend:latest ./frontend
  docker build -t stats-service:latest ./stats-service
  docker build -t demo-projet-devops-tasks-service:latest ./tasks-service
  ```

### 2. Déploiement Kubernetes avec Helm

- **Tester la génération des templates :**
  ```bash
  helm template frontend ./helm/frontend
  helm template stats-service ./helm/stats-service
  helm template tasks-service ./helm/tasks-service
  ```

- **Installer les chartes K8s sur le cluster (crée automatiquement le namespace) :**
  ```bash
  helm install frontend ./helm/frontend -n demo-devops --create-namespace
  helm install stats-service ./helm/stats-service -n demo-devops --create-namespace
  helm install tasks-service ./helm/tasks-service -n demo-devops --create-namespace
  ```

- **Accéder à l'application :**
  Ouvrez **http://localhost:30080** dans votre navigateur.

- **Mettre à jour une charte après une modification dans son `values.yaml` :**
  ```bash
  helm upgrade frontend ./helm/frontend -n demo-devops
  # idem pour stats-service et tasks-service
  ```

- **Voir la liste des chartes installées :**
  ```bash
  helm ls -n demo-devops
  ```

- **Désinstaller/Supprimer les applications du cluster :**
  ```bash
  helm uninstall frontend stats-service tasks-service -n demo-devops
  ```

---

## ⚡ Raccourcis via Taskfile (Recommandé)

Les tâches Helm lisent automatiquement les secrets depuis le fichier `.env` à la racine du projet.

- **Déployer uniquement le tasks-service (avec injection du DATABASE_URL) :**
  ```bash
  task helm-deploy-tasks
  ```

- **Déployer le stats-service :**
  ```bash
  task helm-deploy-stats
  ```

- **Déployer le frontend :**
  ```bash
  task helm-deploy-frontend
  ```

- **Déployer tous les services en une seule commande :**
  ```bash
  task helm-deploy-all
  ```

> [!NOTE]
> Le fichier `.env` est déjà dans votre `.gitignore`. Ne jamais le committer dans votre dépôt.
