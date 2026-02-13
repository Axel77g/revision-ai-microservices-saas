# 🎓 Revision AI – Architecture Microservices SaaS 

**Plateforme SaaS de révisions intelligentes avec architecture distribuée moderne**

[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=flat&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![TypeScript](https://img.shields.io/badge/TypeScript-007ACC?style=flat&logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![NestJS](https://img.shields.io/badge/NestJS-E0234E?style=flat&logo=nestjs&logoColor=white)](https://nestjs.com/)
[![Vue.js](https://img.shields.io/badge/Vue.js-4FC08D?style=flat&logo=vue.js&logoColor=white)](https://vuejs.org/)
[![RabbitMQ](https://img.shields.io/badge/RabbitMQ-FF6600?style=flat&logo=rabbitmq&logoColor=white)](https://www.rabbitmq.com/)
[![MongoDB](https://img.shields.io/badge/MongoDB-47A248?style=flat&logo=mongodb&logoColor=white)](https://www.mongodb.com/)

---

## 📋 Contexte & Objectif

**Problématique métier** : Les étudiants collégiens/lycéens manquent d'outils efficaces pour créer des quiz personnalisés à partir de leurs documents de cours.

**Solution développée** : SaaS permettant de téléverser des documents (PDF, images) et de générer automatiquement des quiz via intelligence artificielle.

**Objectif technique principal** : Démontrer la maîtrise d'une architecture distribuée moderne, résiliente et scalable en production.

---

## 🏆 Réalisations techniques

### Architecture & Scalabilité
- ✅ **Architecture microservices découplée** (3 services backend + frontend)
- ✅ **Communication asynchrone** via RabbitMQ pour la résilience
- ✅ **Services stateless** permettant la distribution en cluster
- ✅ **Orchestration Kubernetes** avec Ingress, LoadBalancer, secrets
- ✅ **CI/CD complet** via GitHub Actions

### Observabilité & Production
- ✅ **Stack d'observabilité complète** : Prometheus + Grafana + Matomo
- ✅ **Métriques business personnalisées** et monitoring applicatif
- ✅ **Déploiement production-ready** avec gestion des ressources
- ✅ **Gestion des secrets** et configurations sécurisées

### Patterns & Développement
- ✅ **Clean Architecture** avec séparation des responsabilités
- ✅ **Event-driven architecture** avec queues persistantes
- ✅ **Domain-Driven Design** dans l'organisation du code
- ✅ **Tests unitaires** et pipeline de qualité

---

## 🏗️ Architecture

### Vue d'ensemble
```
Frontend (Vue.js) → API Gateway (NestJS) → [RabbitMQ] → Workers (TS/Node.js)
                            ↓
                    MongoDB (données/jobs)
```

### Composants principaux

| Service | Technologie | Responsabilité | Patterns utilisés |
|---------|-------------|----------------|-------------------|
| **API Gateway** | NestJS + TypeScript | Orchestration, authentification, API REST | CQRS, Repository Pattern |
| **File Parser** | Node.js + TypeScript | Parsing PDF/images, extraction de contenu | Worker Pattern, Clean Architecture |
| **Quiz Generator** | Node.js + TypeScript | Génération IA de quiz, validation qualité | Retry Pattern, Circuit Breaker |
| **Frontend** | Vue.js + TypeScript | Interface utilisateur, upload de fichiers | Composition API, State Management |

### Infrastructure & DevOps

```yaml
Production Stack:
├── Kubernetes (orchestration)
│   ├── Ingress NGINX (load balancing)
│   ├── RabbitMQ Cluster (messaging)
│   └── MongoDB Replica Set (persistence)
├── Observabilité
│   ├── Prometheus (métriques)
│   ├── Grafana (dashboards)
│   └── Matomo (analytics web)
└── CI/CD
    ├── GitHub Actions
    ├── Docker multi-stage builds
    └── Automated deployments
```

---

## 🔄 Workflow de génération de quiz

**Pattern Event-Driven implementé :**

1. **Upload** → L'utilisateur téléverse un fichier (frontend)
2. **Ingestion** → L'API stocke le fichier sur S3 et envoie un événement `file-uploaded`
3. **Parsing** → Le File Parser traite le document et retourne `file-parsed`
4. **Génération** → L'API orchestre et envoie `generate-quiz` au Quiz Generator
5. **Finalisation** → Le quiz généré est stocké et accessible à l'utilisateur

**Avantages de cette approche :**
- Résilience : pannes de service isolées
- Scalabilité : chaque worker peut être répliqué indépendamment  
- Monitoring : tracking complet du pipeline via métriques

<img width="577" height="468" alt="Architecture Workflow" src="https://github.com/user-attachments/assets/bbb66b86-8d45-4b84-983b-484f61b8c43d" />

---

## 📊 Fonctionnalités clés

### Côté utilisateur
- 📁 Upload multi-formats (PDF, images) avec validation
- 🤖 Génération automatique de quiz IA (QCM, questions ouvertes)
- 📈 Suivi des performances et analytics d'apprentissage
- 💳 Système d'abonnements Stripe intégré

### Côté technique  
- 🔄 Pipeline de traitement asynchrone robuste
- 📊 Métriques business en temps réel (temps de génération, taux d'erreur, retry)
- 🔐 Authentification JWT + gestion des rôles
- 📦 API REST documentée (OpenAPI/Swagger)

---

## 💡 Choix techniques / Patterns

### Architecture
- **Microservices** : Séparation claire des responsabilités, réutilisabilité
- **Event-Driven** : Découplage via RabbitMQ, résilience aux pannes
- **CQRS** : Séparation lecture/écriture pour optimiser les performances

### DevOps & Production
- **Kubernetes** : Orchestration, auto-healing, scaling horizontal
- **Monitoring observability** : Détection proactive des issues
- **Infrastructure as Code** : Manifests K8s versionnés, reproductibilité

### Développement
- **TypeScript** : Type safety, maintenabilité à grande échelle
- **NestJS** : Architecture modulaire, injection de dépendances
- **Clean Architecture** : Testabilité, isolation des couches métier

<img width="908" height="469" alt="Monitoring Grafana" src="https://github.com/user-attachments/assets/560e1aa4-f865-4513-9f88-04be28021ff2" />

---

## 🚀 Mise en route

### Prérequis
- Node.js 22+
- Docker & Docker Compose
- Kubernetes (kind pour local)

-----

## Backend

Configurez le fichier `.env` en suivant les instructions du fichier `.env.example`. Ce fichier contient les variables d'environnement nécessaires au bon fonctionnement de l'application.

Installez les dépendances dans chaque service backend :

```bash
cd services/api
npm install
cd ../file-parser
npm install
cd ../quiz-generator
npm install
```

Lancez le backend avec Docker Compose :

```bash
docker compose up -d --build
```

-----

## Frontend

Configurez le fichier `.env` en vous inspirant du fichier [`.env.production`](frontend/.env.production). Ce fichier contient les variables d'environnement nécessaires au bon fonctionnement de l'application.

```bash
cd frontend
npm install
npm run dev
```

-----

## Tests

Pour exécuter les tests, vous devez d'abord installer les dépendances dans chaque service backend comme indiqué précédemment. Ensuite, vous pouvez exécuter les tests via la commande suivante à la racine du projet :

```bash
sh services/scripts/tests.sh
```

-----

# Setup production

-----

## Création des images production

D'abord, commencez par builder l'application frontend, en utilisant `.env.kind` :

```bash
npm run build-only -- --mode kind
```

Pour créer l'ensemble des images Docker nécessaires au déploiement de l'application, exécutez le script suivant à la racine du projet :

```bash
sh scripts/build-docker-images.sh
```

-----

## Déploiement sur Kubernetes (Local avec kind)

> Le déploiement complet en production est géré par CI/CD via Github Actions ([`deploy-on-k8s.yml`](./.github/workflows/deploy-on-k8s.yml) l'app de production n'est plus disponible.

### Prérequis

* `kind` doit être installé.
* `kubectl` doit être installé.

Créez les secrets Kubernetes nécessaires en base64 en vous basant sur [`.env-secrets.yml.example`](./k8s/secrets/env-secret.yml.example) et en les plaçant dans `env-secret.yml`.

### Création du cluster kind et installation des services

```bash
sh scripts/full-setup-cluster.sh
```

### Accès aux services

Pour accéder aux services déployés sur le cluster Kubernetes, vous pouvez utiliser `kubectl port-forward` pour rediriger les ports locaux vers les services du cluster, ou installer le load balancer `cloud-provider-kind`.

#### Installation du load balancer kind

```bash
go install sigs.k8s.io/cloud-provider-kind@latest
```

Exécutez-le en mode `sudo` :

```bash
sudo ~/go/bin/cloud-provider-kind
```

#### Récupérer l'IP externe du cluster

```bash
kubectl get svc -n ingress-nginx
# Exemple de sortie
NAME                                      TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                         AGE
ingress-nginx-controller                  LoadBalancer   10.96.101.63    192.168.148.5   80:32736/TCP,443:32614/TCP      164m
ingress-nginx-controller-admission        ClusterIP      10.96.116.137   <none>          443/TCP                         164m
```

Modifiez votre fichier `/etc/hosts` pour ajouter l'IP externe du cluster avec les noms de domaine `revision-ai.local` et `api.revision-ai.local` :

```
# Ajoutez cette ligne à votre fichier /etc/hosts, en remplaçant l'IP par celle de votre cluster
192.168.148.5 revision-ai.local api.revision-ai.local
```

Rendez-vous sur `http://revision-ai.local` pour accéder au frontend. L'API devrait être accessible sur `http://api.revision-ai.local`.

--- 

## 📝 Licence

Projet académique — ESGI 5ème année

---

<p align="center">
  <b>Développé par <a href="https://github.com/Insane-Bob">Insane-Bob</a> <a href="https://github.com/Robiinf">Robiinf</a> <a href="https://github.com/Prumme">Prumme</a> <a href="https://github.com/Axel77g">Axel77g</a></b><br>
</p>
