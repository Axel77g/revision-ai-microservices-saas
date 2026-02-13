# Revision AI – SaaS de révisions intelligentes

Revision AI est une plateforme SaaS qui aide les collégiens et lycéens à réviser efficacement. Les utilisateurs peuvent téléverser leurs documents de cours et générer automatiquement des quiz personnalisés grâce à l’intelligence artificielle.

Le projet illustre une architecture distribuée moderne, avec microservices, Kubernetes, RabbitMQ, et des outils d’observabilité (Prometheus, Grafana, Matomo).

## 🚀 Objectif du projet

- Démontrer l’utilisation d’une architecture distribuée résiliente et scalable.
- Appliquer des concepts modernes : microservices découplés, communication asynchrone via RabbitMQ, orchestration Kubernetes.
- Montrer la mise en place d’une observabilité complète pour monitorer et analyser les performances des services.
- Créer une solution pratique d’apprentissage interactif pour les étudiants.

## 🛠️ Compétences démontrées

- Développement backend Node.js / TypeScript multi-service
- Microservices découplés et communication asynchrone avec RabbitMQ
- Bases de données NoSQL (MongoDB) et gestion des tâches
- Orchestration Kubernetes (Ingress, LoadBalancer, secrets)
- Observabilité et monitoring (Prometheus, Grafana, Matomo)
- CI/CD et scripts de build pour dev et production
- Déploiement local et production-ready avec Docker et kind

## 🏗️ Architecture du projet

- **Frontend** : interface web pour téléverser les cours et passer les quiz.
- **API** : point d’entrée central, orchestre les requêtes et services.
- **File Parser** : analyse les fichiers (PDF, images) et extrait les données structurées.
- **Quiz Generator** : crée des quiz variés via IA (QCM, questions ouvertes).
- **MongoDB** : stockage des documents, quiz et état des tâches.
- **Communication asynchrone** : RabbitMQ pour découpler les services et garantir la résilience.

Tous les services sont stateless afin de pouvoir les distribuer/répliquer dans un cluster

Les services **File Parser** et **Quiz Generator** sont entièrement découplés, ce qui offre la flexibilité de les réutiliser dans d'autres projets.

## 🔄 Workflow de génération de quiz

1. L’utilisateur téléverse un fichier via le frontend.
2. L’API reçoit le fichier et l'upload sur un storage (ici un bucket S3).
3. L'API demande le parsing du fichier au File Parser via RabbitMQ
4. Le File Parser recupère le fichier
5. Le File pParser retourne le fichier parsé (au format JSON) via RabbitMQ
6. L’API orchestre le tout, si tous les fichiers du quiz sont parsés, elle envoient une demande de génération (RabbitMQ)
7. Le quiz generator retourne le quiz généré a l'API
8. L’api le stocke, le client peut le récupérer.

<img width="577" height="468" alt="Capture d’écran 2026-02-13 à 20 11 43" src="https://github.com/user-attachments/assets/bbb66b86-8d45-4b84-983b-484f61b8c43d" />


## 📊 Observabilité et monitoring

- **Matomo** : analytics web pour comprendre l’usage de la plateforme.
- **Prometheus** : collecte métriques temps réel sur les services.
- **Grafana** : tableaux de bord interactifs pour visualiser la santé et les performances des services.

<img width="908" height="469" alt="Capture d’écran 2026-02-13 à 20 12 06" src="https://github.com/user-attachments/assets/560e1aa4-f865-4513-9f88-04be28021ff2" />

-----

# Setup dev

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

>
