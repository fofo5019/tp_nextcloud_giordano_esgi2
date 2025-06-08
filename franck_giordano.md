# 🚀 Rapport de Synthèse

## Déploiement d'une stack NextCloud avec Docker

**Auteur :** Franck Giordano
**Dépôt Git :** [tp\_nextcloud\_giordano\_esgi2](https://github.com/fofo5019/tp_nextcloud_giordano_esgi2)

---

### 🎯 Contexte & Objectifs

Ce projet vise à déployer une solution de cloud personnel robuste et maintenable — **NextCloud** — en s’appuyant sur la technologie **Docker**.
Vous découvrirez ici chaque étape : du déploiement initial à l’industrialisation via Docker Compose, jusqu’aux finitions assurant fiabilité et facilité d’administration.

---

## 📦 Étape 1 : Premier Lancement

**Objectif :** Démarrer manuellement les conteneurs PostgreSQL & NextCloud

1. Création d’un réseau privé Docker
2. Lancement d’un conteneur **PostgreSQL**
3. Lancement d’un conteneur **NextCloud** configuré pour pointer sur la base de données

> **Résultat :** NextCloud disponible sur `http://localhost:8080`
> **Limite :** Pas de persistance des données — arrêt/suppression = perte totale.

---

## 💾 Étape 2 : Persistance des Données

**Objectif :** Garantir la pérennité des fichiers et de la base de données via des volumes Docker

* Création de deux volumes nommés :

  * `app-data` → `/var/www/html` (NextCloud)
  * `db-data`  → `/var/lib/postgresql/data` (PostgreSQL)
* Recréation des conteneurs en les attachant aux volumes

> **Validation :** Après upload d’un fichier test, suppression/recréation des conteneurs → fichier intact.

---

## 🛠️ Étape 3 : Industrialisation avec Docker Compose

**Objectif :** Automatiser et fiabiliser le lancement via un unique fichier `docker-compose.yml`

* Centralisation de la configuration (services, réseaux, volumes)
* Réseau interne pour isoler la base de données (pas de port exposé)
* Commande unique : `docker-compose up -d`

> **Bénéfice :** Déploiement reproductible et rapide.

---

## ✨ Étape 4 : Finitions & Optimisations

Pour aller plus loin, intégration de deux outils complémentaires :

1. **Adminer**
   Service web pour administrer visuellement PostgreSQL (inspections, requêtes SQL)

2. **Healthchecks**
   Vérification automatique de l’état :

   * Base de données prête (`pg_isready`)
   * NextCloud opérationnel (`/status.php`)

> **Impact :** Surveillance proactive, alertes en cas de dysfonctionnement.

---

## 📝 Conclusion

Vous passez d’un simple PoC à une solution complète, sécurisée et évolutive.
Chaque itération renforce la robustesse, la maintenabilité et la qualité professionnelle de votre stack NextCloud.

---

## 📎 Annexe : Fichier `docker-compose.yml`

```yaml
version: '3.8'

services:
  db:
    image: postgres:latest
    restart: always
    environment:
      POSTGRES_USER: nextcloud
      POSTGRES_PASSWORD: db_password_securise
      POSTGRES_DB: nextcloud
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - app-net
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U nextcloud -d nextcloud"]
      interval: 10s
      timeout: 5s
      retries: 5

  app:
    image: nextcloud:latest
    restart: always
    ports:
      - "8080:80"
    environment:
      POSTGRES_HOST: db
      POSTGRES_DB: nextcloud
      POSTGRES_USER: nextcloud
      POSTGRES_PASSWORD: db_password_securise
      NEXTCLOUD_ADMIN_USER: admin
      NEXTCLOUD_ADMIN_PASSWORD: admin
    volumes:
      - app-data:/var/www/html
    networks:
      - app-net
    depends_on:
      - db
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost/status.php || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 3

  adminer:
    image: adminer:latest
    restart: always
    ports:
      - "8081:8080"
    networks:
      - app-net
    depends_on:
      - db

volumes:
  app-data:
  db-data:

networks:
  app-net:
    driver: bridge
