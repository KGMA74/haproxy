# HAProxy - Gestion de File d'Attente avec Flask

Ce dépôt contient une infrastructure prête à l'emploi pour utiliser HAProxy comme load balancer HTTP avec gestion de rate-limiting et de file d'attente pour les utilisateurs, via un service Flask personnalisé.

## Fonctionnalités principales

- **Reverse Proxy HTTP avec HAProxy**  
  - Limitation de débit des requêtes par adresse IP (rate limiting).
  - Gestion automatique de la surcharge des backends : bascule vers une page d'attente personnalisée lorsque tous les serveurs sont saturés ou indisponibles.
  - Statistiques administrateur accessibles par socket.
- **Service de file d'attente (waitlist)**  
  - Exposé via Flask (`waitlist_service.py`).
  - Affiche la position de l'utilisateur dans la file basée sur les métriques HAProxy.
  - Actualisation automatique de la page d'attente.
- **Déploiement Docker et Docker Compose**  
  - Fichiers `Dockerfile` et `docker-compose.yml` fournis pour le déploiement simple.

## Structure du dépôt

- `haproxy.cfg` :  
  Configuration principale de HAProxy. Définit :
  - Les frontends HTTP (écoute sur port 80).
  - Le rate limiting via `stick-table` et rejet `429` si trop de requêtes.
  - Les backends :  
    - `servers` : Serveur applicatif principal (personnalisable).
    - `waitlist` : Service Flask de file d'attente.
  - Routage conditionnel :
    - `/with-haproxy` passe aux backends réels.
    - Si tous les serveurs sont indisponibles, ou en cas de surcharge, redirection automatique vers `/waitlist`.
  - Remplacement automatique de la page d'erreur HAProxy 503 par la page d'attente personnalisée.
- `waitlist_service.py` :  
  Application Flask fournissant une page web dynamique d'attente :
  - Interroge les statistiques HAProxy en temps réel via socket UNIX.
  - Calcule la position approximative dans la file et un ETA (temps d'attente estimé).
  - HTML responsive avec auto-refresh (JavaScript).
- `requirements.txt` :  
  Dépendances Python, principalement pour Flask et son écosystème :
  ```
  blinker==1.9.0
  click==8.3.1
  Flask==3.1.2
  itsdangerous==2.2.0
  Jinja2==3.1.6
  MarkupSafe==3.0.3
  Werkzeug==3.1.3
  ```
- `docker-compose.yml` et `Dockerfile` :  
  Pour lancer HAProxy et le service Flask sur la même plateforme, configuration prêtes à l'emploi.

## Prérequis

- Docker et docker-compose
- (optionnel) Python 3.8+ si usage direct de Flask hors conteneur

## Démarrage rapide (via Docker Compose)

```sh
docker-compose up --build
```

HAProxy sera disponible sur le port 80 et le service Flask également accessible via HAProxy.

## Personnalisation

- **Backend principal:**  
  Modifiez dans `haproxy.cfg` la ligne suivante pour indiquer l’IP/hostname de votre vrai serveur applicatif :
  ```
  server S1 10.195.125.85:8000 maxconn 200 check
  ```
- **Temps moyen de service :**  
  Ajustez la variable d'environnement `AVG_SERVICE_TIME` ou modifiez sa valeur par défaut dans `waitlist_service.py`.

## Auteur

Ce dépôt a été généré et maintenu par [KGMA74](https://github.com/KGMA74).

## Licence
