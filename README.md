# 🧭 Brief Kubernetes — Déploiement d’une API dans un cluster Minikube

> **Auteur** : Hélène  
> **Formation** : Alternance – Ingénieure Data  
> **Thématique** : Découverte et mise en œuvre de Kubernetes pour le déploiement d’applications conteneurisées  
> **Outil principal** : Minikube + Kubectl

---

## 🚀 Objectif du brief

Mettre en place une **infrastructure complète sous Kubernetes** permettant le déploiement d’une **API conteneurisée**, son **exposition via un Ingress**, et la **gestion de la persistance des données** avec des services et volumes.

Ce projet a pour but de :
- Comprendre la structure et les objets fondamentaux de Kubernetes (Pod, Service, Deployment, Ingress, ConfigMap, Secret, PVC).  
- Automatiser le déploiement d’une stack applicative simple.  
- Apprendre à déboguer un cluster local et observer le comportement des ressources déployées.  
- Poser les bases pour une future **CI/CD pipeline** sur un vrai cluster.

---

## 🧩 Architecture cible

## 🏗️ Architecture du projet

L’architecture déployée sur AKS est simple mais robuste : l’API FastAPI communique avec MySQL en interne, et l’accès externe passe par l’Ingress NGINX. Voici le schéma conceptuel :

```mermaid
graph LR
    subgraph External
        A[Client]
    end

    subgraph Ingress
        B[Ingress NGINX]
    end

    subgraph Cluster
        C[Service API ClusterIP]
        D[Pod API FastAPI]
        E[Pod MySQL]
        F[(PVC Azure Disk)]
    end

    A -->|HTTP(S)| B
    B -->|Routage interne| C
    C --> D
    D -->|Connexion MySQL| E
    E --> F
```

## ⚙️ Ressources déployées
| Type       | Nom              | Rôle                                          |
| ---------- | ---------------- | --------------------------------------------- |
| Deployment | `api-deployment` | Héberge le conteneur de l’API                 |
| Service    | `api-service`    | Expose l’API à l’intérieur du cluster         |
| Ingress    | `api-ingress`    | Rend l’API accessible depuis l’extérieur      |
| ConfigMap  | `api-config`     | Variables de configuration non sensibles      |
| Secret     | `api-secret`     | Données confidentielles (ex : credentials DB) |
| PVC        | `api-pvc`        | Persistance de données pour l’API             |

## 📂 Structure du dépôt
``` bash
Brief-Kubernetes/
│
├── api/
│   ├── Dockerfile          # Image de l'API
│   ├── app.py              # Code source Python / FastAPI
│   └── requirements.txt    # Dépendances
│
├── k8s/
│   ├── deployment.yaml     # Déploiement de l’API
│   ├── service.yaml        # Service interne
│   ├── ingress.yaml        # Exposition HTTP externe
│   ├── configmap.yaml      # Configuration externe
│   ├── secret.yaml         # Données sensibles
│   └── pvc.yaml            # Volume persistant
│
└── README.md               # Ce fichier ✨
``` 
## 🧠 Étapes de mise en œuvre
### 1️⃣ Démarrage du cluster
``` bash
minikube start --driver=docker
kubectl get nodes
```

### 2️⃣ Déploiement de l’application
``` bash
kubectl apply -f k8s/
kubectl get all -n lnd
```

### 3️⃣ Exposition de l’API
``` bash
minikube tunnel
# ou
minikube service list


Ensuite, tester avec :

curl http://api-test.local
```

### 4️⃣ Nettoyage du cluster
``` bash
kubectl delete -f k8s/
minikube stop
```

## 🧰 Kit de survie Kubernetes

Parce qu’un bon dev ne part jamais sans son kit d’urgence.

### 🔍 Vérifier les ressources
``` bash
kubectl get pods -A
kubectl describe pod <nom_pod>
kubectl logs <nom_pod>
```

### 🔄 Relancer un pod
``` bash
kubectl rollout restart deployment <nom_deployment>
```

### 🧱 Debug visuel
``` bash
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get ingress -n lnd
kubectl get svc -n lnd
```

### 🐳 Si tout part en vrille
``` bash
minikube delete
minikube start --driver=docker
```

## 🌐 Accès local
| Élément            | URL                     | Description                 |
| ------------------ | ----------------------- | --------------------------- |
| API                | `http://api-test.local` | Endpoint exposé via Ingress |
| Dashboard Minikube | `minikube dashboard`    | Interface visuelle          |
| Service interne    | `ClusterIP`             | Communication interne       |

## 🧩 Points de validation

✅ Le déploiement se fait via des manifests YAML
✅ L’API répond via un service interne et un Ingress
✅ Les configurations sont externalisées (ConfigMap / Secret)
✅ Le volume persiste les données entre deux redéploiements
✅ Le projet est documenté dans ce README

## 🧭 Pistes d’amélioration

🔄 Ajouter un HorizontalPodAutoscaler

📦 Mettre en place un Helm Chart pour automatiser le déploiement

🧑‍💻 Créer une CI/CD GitHub Actions

🧩 Connecter à une vraie base de données (Postgres via StatefulSet)

🛡️ Sécuriser le tout avec un certificat TLS via Ingress

## 💬 Citation du jour

« Les containers, c’est comme les chaussettes : il faut les isoler, les surveiller, et les changer régulièrement. »
— Un admin sys fatigué, mais lucide 🧙‍♂️

## 🧤 Kit de survie express
| Commande                       | Description rapide        |
| ------------------------------ | ------------------------- |
| `kubectl get all -n lnd`       | Voir tout ce qui tourne   |
| `kubectl describe <ressource>` | Inspecter une ressource   |
| `kubectl logs -f <pod>`        | Suivre les logs           |
| `kubectl apply -f .`           | (Re)déployer              |
| `minikube service list`        | Voir les services exposés |
| `minikube tunnel`              | Activer l’accès externe   |

## 🏁 Conclusion

Ce brief constitue une première approche concrète de Kubernetes, de la logique déclarative des manifests YAML et de la gestion d’un cluster local avec Minikube.
C’est un excellent tremplin avant d’aborder des environnements cloud (AKS, EKS, GKE).

🧠 “Kubernetes ne se maîtrise pas, il s’apprivoise.”
— Hélène, alternante ingénieure de données ✨