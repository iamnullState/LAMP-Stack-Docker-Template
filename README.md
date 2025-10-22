# **LAMP Stack Docker Template**

  

A minimal Dockerized LAMP skeleton you can clone or copy/paste to start small PHP projects quickly.

It includes:

- Apache + PHP (serving /app/public as the document root)
    
- MariaDB 11
    
- phpMyAdmin
    
- MailHog (local SMTP sink + web UI)
    
- A committed .env (intentionally kept in Git for educational purposes)
    

  

> Note: Keeping real secrets in Git is unsafe. This template commits .env on purpose for learning and quick starts. For anything beyond local learning, do not commit secrets.

---

## **Prerequisites**

- Docker Desktop (or Docker Engine + docker compose v2)
    
- Optional: Composer (inside the container or on your host) if your app uses PHP dependencies (e.g., vlucas/phpdotenv)
    

---

## **Project Layout**

```
.
├─ docker-compose.yml
├─ app/
│  ├─ public/
│  │  └─ index.php
│  └─ .env                 # committed for learning
├─ config/
│  ├─ php.ini              # PHP overrides
│  └─ vhost.conf           # Apache vhost; docroot = /var/www/html/public
└─ (named volume) db_data  # MariaDB data (created by Docker)
```

---

## **Services and Ports**

- App (Apache/PHP): http://localhost:8080
    
- phpMyAdmin: http://localhost:8081
    
- MailHog UI: http://localhost:8025
    
- MariaDB host port: 3306 (change if you already run MySQL locally)
    

---

## **Environment Variables**

  

This template keeps a committed app/.env for convenience. These variables are mounted into containers via env_file in docker-compose.yml.

  

**app/.env (example included):**

```
APP_NAME=LAMP_PROJECT
APP_ENV=local
APP_DEBUG=true
APP_TIMEZONE=UTC
APP_URL=http://localhost

MARIADB_CONNECTION=mysql
MARIADB_ROOT_PASSWORD=root
MARIADB_DATABASE=LAMP_PROJECT
MARIADB_USER=lamp
MARIADB_PASSWORD=pass
MARIADB_CHARSET=utf8mb4
MARIADB_COLLATION=utf8mb4_unicode_ci
MARIADB_PORT=3306

MAIL_MAILER=smtp
MAIL_HOST=mailhog
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_FROM_ADDRESS=no-reply@lamp.local
MAIL_FROM_NAME="LAMP Project"
```

> Important: In docker-compose.yml, the db service uses env_file: ./app/.env. Avoid duplicating the same MariaDB variables under a separate environment: block. If both exist, the explicit environment: values will win. Keep **one single source of truth** (recommended: the env_file).

---

## **Quick Start**

1. **Clone** or copy the template into a new directory.
    
2. **Verify** **.env** **values** in app/.env.
    
    For a first run, the values in the provided file are fine.
    
3. **Start the stack**
    

```
docker compose up -d
```

3.   
    
4. **Visit services**
    
    - App: http://localhost:8080
        
    - phpMyAdmin: http://localhost:8081
        
        - Server: db
            
        - User: lamp
            
        - Password: pass
            
        - Or log in as root with root (if you set MARIADB_ROOT_PASSWORD=root)
            
        
    - MailHog: http://localhost:8025
        
    
5. **(Optional) Install PHP dependencies**
    
    If your app requires Composer packages (e.g., Dotenv), either:
    
    - Run inside the container:
        
    

```
docker compose exec app bash
cd /var/www/html
composer install
```

5. -   
        
    - Or run Composer on your host in app/ and mount vendor/ (already mounted via ./app:/var/www/html).
        
    

---

## **How It Works**

- **Apache/PHP**
    
    The container serves from /var/www/html/public. Your repo’s ./app is mounted to /var/www/html. The APACHE_DOCUMENT_ROOT is set to /var/www/html/public.
    
- **MariaDB**
    
    Uses a named volume db_data for persistence. First boot will initialize the database using credentials from app/.env. If the data dir already exists, those credentials are not re-applied.
    
- **phpMyAdmin**
    
    Convenience admin UI connected to the db service.
    
- **MailHog**
    
    SMTP sink for local dev. Point your app’s mailer to mailhog:1025. View captured emails in the MailHog web UI.
    

---

## **Common Commands**

```
# Start in the background
docker compose up -d

# Follow logs for a service
docker compose logs -f app
docker compose logs -f db
docker compose logs -f phpmyadmin
docker compose logs -f mailhog

# Exec into a container
docker compose exec app bash
docker compose exec db bash

# Stop and remove containers (data persists in named volumes)
docker compose down

# Stop and remove containers AND volumes (wipes MariaDB data)
docker compose down -v
```

---

## **Troubleshooting**

  

### **Compose shows “variable is not set… Defaulting to a blank string”**

  

Example:

```
WARN The "MARIADB_USER" variable is not set. Defaulting to a blank string.
```

Cause: You used ${MARIADB_*} in docker-compose.yml without providing those values in a project-root .env file (Compose-time substitution), while you actually rely on env_file: ./app/.env (container-time env).

Fix: Prefer **only** env_file: ./app/.env in services that need DB vars and **remove** ${...} from environment:. Or, if you want to use ${...}, put those variables in a .env next to docker-compose.yml.

  

### **MariaDB exits with “Database is uninitialized and password option is not specified”**

  

Ensure MARIADB_ROOT_PASSWORD (and other DB vars) are present and readable via env_file. If the db_data volume already has partial or old state, first-time init won’t rerun. To force a fresh init:

```
docker compose down -v
docker compose up -d
```

Warning: This deletes the DB data volume.

  

### **Port already in use (3306, 8080, 8081, 8025)**

  

Change the **host** ports in docker-compose.yml:

```
# examples
app:
  ports:
    - "8088:80"
db:
  ports:
    - "3307:3306"
phpmyadmin:
  ports:
    - "8090:80"
mailhog:
  ports:
    - "8026:8025"
```

---

## **Using a Hardcoded Simple Login**

  

This template doesn’t enforce auth. If you add a hardcoded login in your PHP code, it will just work as usual. The committed .env remains available to PHP via vlucas/phpdotenv if you choose to read configuration values at runtime.

---

## **Production Notes**

- Don’t commit real secrets. For real projects, put secrets in a non-committed .env or a secret manager, and ensure docker-compose.yml does not hardcode credentials.
    
- Use strong, unique passwords.
    
- Configure proper mail delivery (not MailHog) when deploying.
    
- Add HTTPS termination (e.g., via a reverse proxy) if exposing publicly.
    

---

## **License**

  

Use, modify, and adapt freely for your own projects.