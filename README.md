# Revision AI

Revision AI est une solution SaaS innovante conçue pour aider les collégiens et lycéens dans leurs révisions. Notre plateforme permet aux utilisateurs de téléverser leurs documents de cours, puis utilise l'intelligence artificielle pour générer automatiquement des quiz de révision personnalisés. Cette approche dynamique et interactive vise à optimiser l'apprentissage et à améliorer la rétention des connaissances.

Pour garantir une **haute disponibilité**, une **scalabilité** et une **résilience** optimales, Revision AI s'appuie sur une **architecture distribuée** robuste, orchestrée par **Kubernetes**. Cette infrastructure est composée de plusieurs microservices interconnectés, chacun ayant un rôle spécifique dans le processus de création et de gestion des quiz.

> URL : https://revision-ai.com
> Login admin si besoin : axel@axel.com - 123456789@ESGI

-----

## Architecture des services

L'écosystème de Revision AI est constitué de trois services principaux, complétés par un frontend :

* **API (Application Programming Interface)** : C'est le point d'entrée central pour toutes les interactions avec la plateforme. L'API gère les requêtes des utilisateurs, orchestre le flux de données entre les différents services et assure la communication avec la base de données.
* **File Parser** : Ce service est responsable de l'analyse et de l'extraction des informations pertinentes des documents de cours téléversés par les utilisateurs. Il traite différents formats de fichiers (actuellement PDF et images) et transforme le contenu brut en données structurées exploitables par l'IA.
* **Quiz Generator** : C'est le cœur intelligent de Revision AI. Ce service utilise des algorithmes d'intelligence artificielle pour générer des quiz pertinents et variés à partir des données parsées. Il est capable de créer différents types de questions (QCM, questions ouvertes, etc.) adaptées au contenu du cours.
* **Frontend** : L'interface utilisateur web qui permet aux étudiants d'interagir avec la plateforme, de téléverser leurs cours, de lancer la génération de quiz et de les passer.

Les services **File Parser** et **Quiz Generator** sont entièrement découplés, ce qui offre la flexibilité de les réutiliser dans d'autres projets.

<img width="577" height="468" alt="Capture d’écran 2026-02-13 à 20 11 43" src="https://github.com/user-attachments/assets/bbb66b86-8d45-4b84-983b-484f61b8c43d" />

-----

## Interconnexion des services et workflow

Les services de Revision AI sont interconnectés de manière asynchrone et découplée, garantissant ainsi une meilleure résilience en cas de panne de l'un des services. Cette communication est principalement assurée par :

* **RabbitMQ** : Un courtier de messages qui facilite la communication asynchrone entre les microservices. Il permet de mettre en file d'attente les tâches et de garantir que les messages sont livrés même si un service est temporairement indisponible.
* **MongoDB** : Une base de données NoSQL utilisée pour stocker les données des utilisateurs, les documents de cours parsés, les quiz générés, et l'état des tâches de génération de quiz.

Le **workflow de génération de quiz** se déroule comme suit :

1.  **Téléchargement du fichier** : Un utilisateur télécharge un fichier de cours via l'interface web.
2.  **Requête à l'API** : Le frontend envoie le fichier à l'API.
3.  **Délégation au File Parser** : L'API reçoit le fichier et envoie une requête au service **File Parser** via RabbitMQ pour initier le processus d'analyse.
4.  **Parsing et envoi des données** : Le File Parser traite le fichier, extrait les informations et, une fois le parsing terminé, envoie les données structurées à l'API.
5.  **Génération du quiz** : L'API transmet les données parsées au service **Quiz Generator** via RabbitMQ.
6.  **Quiz généré et stockage** : Le Quiz Generator utilise l'IA pour créer le quiz et envoie le résultat à l'API. L'API stocke ensuite le quiz final dans MongoDB, le rendant accessible à l'utilisateur.

-----

## Analytics et monitoring

Pour comprendre l'utilisation de la plateforme et garantir des performances optimales, Revision AI intègre des outils d'analytics et de monitoring :

* **Matomo** : Une solution d'analyse web open-source utilisée pour collecter des données sur l'utilisation de la plateforme, permettant d'améliorer l'expérience utilisateur et d'identifier les tendances. Elle est auto-hébergée sur le cluster Kubernetes, avec une base de données SQL externalisée et managée par Aiven.
* **Prometheus** : Un système de monitoring et d'alerte open-source qui collecte des métriques en temps réel sur les performances des différents services de l'architecture, intégré dans le cluster.
* **Grafana** : Une plateforme d'observabilité open-source qui permet de visualiser les données collectées par Prometheus sous forme de tableaux de bord interactifs, offrant une vue d'ensemble claire de la santé et des performances du système. Grafana est également disponible dans le cluster et utilisable uniquement via port-forwarding.

Enfin, l'accès externe à l'API est géré par un **Ingress NGINX**, agissant comme un point d'entrée unique et sécurisé pour toutes les requêtes entrantes, assurant la répartition de la charge et la terminaison SSL.

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

> Le déploiement complet en production est géré par CI/CD via Github Actions ([`deploy-on-k8s.yml`](./.github/workflows/deploy-on-k8s.yml) nous vous conseillons d'utiliser l'app de production disponible ici [revision-ai.com](https://revision-ai.com) .

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
