# TODO LIST - ft_transcendence

## 1. Infrastructure & Monitoring
- [Monitoring] **Prometheus & Grafana** :
  - Déployer Prometheus (collecte de métriques) et Grafana (visualisation).
  - Configurer les datasources et importer des dashboards par défaut (Node Exporter, métriques applicatives).
- [Orchestration] **Probes** : Ajouter `livenessProbe` et `readinessProbe` sur tous les services d'infrastructure.

## 2. Développement & Packages Partagés (TypeScript)
*Créer des librairies internes (dans `packages/`) pour standardiser le code des microservices et éviter la duplication :*

- [Package] **@transcendence/types (Shared Types)** :
  - Centraliser les DTOs (Data Transfer Objects), les interfaces API, et les Enums partagés.
  - Définir les payloads des événements RabbitMQ (pour garantir le typage fort entre émetteur et consommateur).
  
- [Package] **@transcendence/database (SQLite)** :
  - Wrapper autour de `better-sqlite3` importable comme un Module Fastify.
  - Gérer la configuration automatique des PRAGMAs (WAL mode, Foreign Keys) et la connexion unique.
  - Exposer un Service prêt à l'emploi pour les microservices.

- [Package] **@transcendence/lifecycle (Graceful Shutdown)** :
  - Remplacer la logique manuelle dans `main.ts`.
  - Gérer l'écoute des signaux (`SIGTERM`, `SIGINT`).
  - Ordonnancer la fermeture propre des ressources (Serveur HTTP -> RabbitMQ -> Base de données) pour éviter la corruption de données.

- [Package] **@transcendence/config (Vault)** :
  - Créer un service capable de lire les secrets injectés (fichiers) OU de requêter l'API Vault directement (pour le chiffrement GDPR/Transit).
  - Centraliser la validation des variables d'environnement (zod/joi).

- [Package] **@transcendence/event (RabbitMQ)** :
  - Abstraire la connexion AMQP et la gestion des échanges/files.
  - Fournir des décorateurs ou services typés pour publier/souscrire aux événements définis dans `EVENTS_ARCHITECTURE.md`.

- [Package] **@transcendence/logger (ELK)** :
  - Configurer un logger (basé sur Pino ou Winston) qui formate les logs en JSON.
  - Envoyer les logs vers le service Logstash (via TCP/UDP) en plus de la console standard.

- [Package] **@transcendence/metrics (Prometheus)** :
  - Exposer un endpoint `/metrics` standard pour le scraping Prometheus.
  - Fournir des helpers pour créer des compteurs/gauges métiers (ex: "parties jouées", "utilisateurs connectés").

## 3. Sécurité Kubernetes & Réseau
- [Ingress] **HTTPS/WSS** : Forcer le HTTPS et le support WebSocket Sécurisé (WSS).
- [Réseau] **NetworkPolicies** : Isoler les flux réseau (ex: seul Logstash parle à Elastic, seul le Gateway parle au Frontend).
- [Context] **SecurityContext** : Durcir les pods (`runAsNonRoot`, `readOnlyRootFilesystem`) pour limiter la surface d'attaque.

## 4. Backend & Robustesse
- [Refactoring] **Utilisation des Packages** : Migrer le `service-template` et le `backend-gateway` pour utiliser les nouveaux packages (lifecycle, database, logger) au lieu du code en dur.

## 5. Automation & Tooling (Developer Experience)
- [CLI] **Service Generator** :
  - Créer un script/outil (ex: `./scripts/create-service.sh` ou via Node) pour générer un nouveau microservice basé sur le `service-template`.
  - Automatiser la création des fichiers associés :
    - Manifeste Kubernetes (StatefulSet/Service).
    - Politiques Vault (.hcl).
    - Rôle Vault.
    - Configuration package.json (Workspace).

## 6. Qualité, Sécurité & Documentation (Nouveaux Ajouts)

- [Package] **@transcendence/eslint-config** :
  - Centraliser la configuration ESLint/Prettier pour garantir un style de code uniforme sur tout le monorepo.
  
- [Package] **@transcendence/testing (Vitest)** :
  - Configurer l'environnement de test unitaire.
  - Fournir des mocks pour les dépendances lourdes (Vault, RabbitMQ) afin de tester la logique métier isolément.

- [Backend] **Database Migrations** (via `@transcendence/database`) :
  - Implémenter un système de versionning du schéma SQLite (fichiers `.sql` incrémentaux).
  - Exécuter automatiquement les migrations au démarrage du pod pour mettre à jour la DB sans perte de données.

- [API] **Documentation (Swagger/OpenAPI)** :
  - Intégrer `@fastify/swagger` dans le `service-template`.
  - Auto-générer la documentation des routes via les métadonnées des décorateurs.
  - Exposer l'UI Swagger sur `/documentation` (protégé en prod ?).

- [Sécurité] **Rate Limiting & Compression** :
  - Configurer `@fastify/rate-limit` sur le Gateway pour prévenir le brute-force/spam.
  - Activer `@fastify/compress` pour optimiser les transferts JSON.

- [Sécurité] **Validation (Zod)** :
  - Standardiser la validation des DTOs entrants avec `zod` pour rejeter les données malformées avant qu'elles n'atteignent la logique métier.

- [Guide] **Création de Service** : Rédiger un guide (`docs/NEW_SERVICE.md`) expliquant le processus manuel ou automatique pour ajouter un service (ServiceAccount, Vault Role, K8s manifest).