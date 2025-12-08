# 1. WordPress Docker Umgebung – Schritt-für-Schritt Anleitung

---

## 1.1. Repository auf GitHub anlegen

## 1.2. Repo klonen

```bash
cd /c/Users/deinName/Documents/Git_Repository   # oder dein Projekt-Ordner
git clone https://github.com/DEINUSERNAME/wp-int-JW.git
cd wp-int-JW
```

## 1.3. Verzeichnisstruktur anlegen

### Basis-Dateien

```bash
touch docker-compose.yml
touch .env.example
touch README.md
touch KI_PROMPTS.md
touch .gitignore
```

### WordPress-Struktur

```bash
mkdir -p wordpress/wp-content
mkdir -p wordpress/plugins
mkdir -p wordpress/themes
mkdir -p wordpress/uploads
```

## 1.4. .gitkeep in leere Ordner legen

```bash
touch wordpress/.gitkeep
touch wordpress/wp-content/.gitkeep
touch wordpress/plugins/.gitkeep
touch wordpress/themes/.gitkeep
touch wordpress/uploads/.gitkeep
```

## 1.5. Prüfen

```bash
ls -a
ls wordpress
```

## 1.6. Ersten Commit machen

```bash
git status          # nur zum Schauen
git add .
git commit -m "Initiale Projektstruktur für WordPress-Docker-Umgebung"
```

## 1.7. Alles auf GitHub pushen

```bash
git push -u origin main
```

## 1.8. .env.example vorbereiten

```
DB_NAME=wordpress
DB_USER=wp_user
DB_PASSWORD=********
DB_ROOT_PASSWORD=********

PMA_USER=wp_user
PMA_PASSWORD=********
```

## 1.9. Lokale .env anlegen

```bash
cp .env.example .env
```

## 1.10. .gitignore anpassen

```
.env
wordpress/uploads/*
wordpress/uploads/
```

## 1.11. docker-compose.yml schreiben

```yaml
version: "3.9"

services:
  db:
    image: mariadb:10.11
    container_name: wp-db
    restart: unless-stopped
    environment:
      MYSQL_DATABASE: ${DB_NAME}
      MYSQL_USER: ${DB_USER}
      MYSQL_PASSWORD: ${DB_PASSWORD}
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
    volumes:
      - db_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

  wordpress:
    image: wordpress:latest
    container_name: wp-app
    restart: unless-stopped
    depends_on:
      db:
        condition: service_healthy
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_NAME: ${DB_NAME}
      WORDPRESS_DB_USER: ${DB_USER}
      WORDPRESS_DB_PASSWORD: ${DB_PASSWORD}
    volumes:
      - ./wordpress/wp-content:/var/www/html/wp-content

  phpmyadmin:
    image: phpmyadmin:latest
    container_name: wp-phpmyadmin
    restart: unless-stopped
    depends_on:
      - db
    ports:
      - "8081:80"
    environment:
      PMA_HOST: db
      PMA_USER: ${PMA_USER}
      PMA_PASSWORD: ${PMA_PASSWORD}

volumes:
  db_data:
```

## 1.12. Umgebung starten & testen

```bash
docker compose up -d
docker ps
```

Browser öffnen:

- http://localhost:8080  
- http://localhost:8081  

## 1.13. Git-Workflow (Feature Branch anlegen)

```bash
git switch -c feature/wp-setup
# Änderungen machen
git status
git add .
git commit -m "WordPress Stack mit MariaDB und phpMyAdmin hinzugefügt"
git push -u origin feature/wp-setup
```

## 1.14. Pull Request auf GitHub

- Dein Repo auf GitHub öffnen  
- GitHub zeigt meist einen Button: **„Compare & pull request“** → klicken  
- Base: `main`, Compare: `feature/wp-setup`  
- Titel & Beschreibung kurz ausfüllen  
- **Create pull request**

## 1.15. PR mergen

Wenn alles gut ist:

- Im PR auf **Merge** klicken  

Danach lokal:

```bash
git switch main
git pull --ff-only origin main
```

---

# 2. Erweiterung eurer WordPress-Docker-Umgebung

---

## 2.1 docker-compose.yml um folgendes ergänzen:
```bash
  wiki:
    image: ghcr.io/requarks/wiki:2
    container_name: wp-wiki
    restart: unless-stopped
    environment:
      DB_TYPE: sqlite
      DB_FILEPATH: /wiki/data/wiki.db
    volumes:
      - wiki_data:/wiki/data
    ports:
      - "3002:3000"

  grafana:
    image: grafana/grafana:latest
    container_name: wp-grafana
    restart: unless-stopped
    ports:
      - "3003:3000"
    volumes:
      - grafana_data:/var/lib/grafana
	  
volumes:
  wiki_data:
  grafana_data:
```

## 2.2. Sicherstellen das alle Container ausgeschaltet sind:
```bash
docker compose down
```

## 2.3. Docker initialisieren:
```bash
docker compose up -d
```

## 2.4. Docker Status prüfen:
```bash
docker ps
```

## 2.6. Funktionstest
- Load/Refresh Test
- Anmeldung testen
- Logs überprüfen mit 
```bash 
docker logs <container-name> --tail 30
```

## 2.7. Branch, Commit und Push
```bash 
git switch -c feature/wiki-grafana
git add .
git commit -m "Erweiterung der Docker-Umgebung: Wiki.js + Grafana hinzugefügt, README aktualisiert"
git push -u origin feature/wiki-grafana
```

### 2.8 Pull Request auf GitHub erstellen
- Dein Repo auf GitHub öffnen
- GitHub zeigt automatisch: "Compare & pull request" -> Darauf klicken
- Ziel: Base: Main - Compare: feature/wiki-grafana
- Konflikte mit "Accept current change" lösen
- Beschreibung kurz ausfüllen
- Pull Request erstellen

### 2.9 Pull auf Lokaler Main
```bash
git switch main
git pull --ff-only origin main
```

---

# Zugriff

### WordPress
→ http://localhost:8080

### phpMyAdmin
→ http://localhost:8081

### Wiki.js
→ http://localhost:3002

### Grafana
→ http://localhost:3003

---
