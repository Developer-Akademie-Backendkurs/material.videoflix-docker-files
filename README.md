# Videoflix - Docker Setup

Dies ist ein Docker Setup, dass dir die Entwicklung und uns die Abnahme des Videoflix Projektes erleichtern soll.

Vor der Verwendung schaue dir bitte die einführenden Videos unter:

[Link zu Videos](https://google.de)

dazu an.

## Table of Contents

<!-- TOC -->

- [Videoflix - Docker Setup](#videoflix---docker-setup)
  - [Table of Contents](#table-of-contents)
  - [Voraussetzungen](#voraussetzungen)
  - [Quickstart](#quickstart)
    - [Aufsetzen und Einrichtung des Projekts](#aufsetzen-und-einrichtung-des-projekts)
      - [Anpassen der settings.py Datei](#anpassen-der-settingspy-datei)
  - [Usage](#usage)
    - [Environment Variablen](#environment-variablen)
    - [Migrations im Docker Container](#migrations-im-docker-container)
    - [requirements.txt](#requirementstxt)
  - [Troubleshooting](#troubleshooting)

<!-- /TOC -->

---

## Voraussetzungen

- **Docker** mit **docker-compose** installiert.

    Siehe [Anleitung](https://docs.docker.com/compose/install/) zur Installation.

    Erforderlich für den Start des Projekts, da es vollständig containerisiert ist.

- **git** ist installiert.

    Siehe [Anleitung](https://git-scm.com/downloads) zur Installation.

    Erforderlich, um das Projekt herunterzuladen.

---

## Quickstart

> [!CAUTION]
> <span style="color: red;">Bitte halte dich genau an die hier beschriebene Anleitung. Wenn du die grundlegene
Konfiguration veränderst, kann das Projekt unter Umständen nicht gestartet werden.</span>
>
> <span style="color: red;">Du kannst Variablen in der `.env` Datei verändern oder neue hinzufügen. Bitte lösche keine
der vorhandenen Variablen.</span>
>
> <span style="color: red;">Bitte ändere nichts, an den im weiteren Verlauf, angegebenen Einträgen in der `settings.py`.</span>
>
> <span style="color: red;">Bitte nimm keine Änderungen an den Dateien `backend.Dockerfile`, `docker-compose` und `backend.entrypoint.sh` vor!<ins></span>
>
> <span style="color: red;">Du kannst (und musst), weitere Packages installieren und auch entsprechende Änderungen an
der `settings.py` Datei vornehmen. <ins>Achte darauf deine `requirements.txt` Datei regelmäßig zu aktualisieren.<ins></span>

1. **Klone das Repository**

    > [!TIP]
    > @Kevin @Dominik
    > Eventuell brauchen wir diesen Punkt nicht?!

    ```bash
    git clone <Link zum Repository>

    # Navigiere zum Projekt Ordner
    cd <Projekt Ordner>
    ```

2. **Definiere die Umgebungsvariablen, unter Benutzung der [.env.template](./.env.template) Datei**. Nutze hierzu die
`git bash Komandozeile`.

    ```bash
    # Erstellt eine .env-Datei mit dem Inhalt von .env.template
    cp .env.template .env
    ```

    > [!IMPORTANT]
    > Stelle sicher, dass die Platzhalterwerte gegebenenfalls durch tatsächliche, für deine Umgebung spezifische Werte
    ersetzt werden.

### Aufsetzen und Einrichtung des Projekts

- Virtual Environment erstellen und aktivieren
- Django installieren
- DRF Installieren
- django rq installieren
- django-redis installieren
- gunicorn installieren
- psycopg2-binary installieren
- python-dotenv installieren
- whitenoise installieren
- aktualisiere deine `requirements.txt` Datei
- erstelle das Django Projekt im aktuellen Ordner
    - projektname => core

#### <ins>Anpassen der `settings.py` Datei

Passe deine `seetings.py` Datei wie folgt an (Bitte lösche unnötige Kommentare, die dir ggf. nur Informationen zum
Editieren liefern. Die ... geben an, dass hier weitere Zeilen stehen, diese müssen auch erhalten bleiben):

```python
# settings.py

from pathlib import Path
# zwei neue Zeilen
import os
from dotenv import load_dotenv

...

# folgende Zeile ändern
SECRET_KEY = os.getenv('SECRET_KEY', default='django-insecure-@#x5h3zj!g+8g1v@2^b6^9$8&f1r7g$@t3v!p4#=g0r5qzj4m3')

# Zwei Zeilen hinzufügen
ALLOWED_HOSTS = os.environ.get("ALLOWED_HOSTS", default="localhost").split(",")
CSRF_TRUSTED_ORIGINS = os.environ.get("CSRF_TRUSTED_ORIGINS", default="http://localhost:4200").split(",")

# Füge django-rq zu deinen Apps hinzu
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'django_rq', # neue Zeile
]

# Füge das whitenoise middleware hinzu
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'whitenoise.middleware.WhiteNoiseMiddleware', # neue Zeile
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

...

# Ändere die Einstellungen für die Datenbak und Füge die Konfiguration für Redis und den RQ-Worker hinzu

# Ersetze die DATABASES Einstellung
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "NAME": os.environ.get("DB_NAME", default="videoflix_db"),
        "USER": os.environ.get("DB_USER", default="videoflix_user"),
        "PASSWORD": os.environ.get("DB_PASSWORD", default="supersecretpassword"),
        "HOST": os.environ.get("DB_HOST", default="db"),
        "PORT": os.environ.get("DB_PORT", default=5432)
    }
}

# Füge die Konfiguration für Redis und RQ hinzu
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": os.environ.get("REDIS_LOCATION", default="redis://redis:6379/1"),
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient"
        },
        "KEY_PREFIX": "videoflix"
    }
}

RQ_QUEUES = {
    'default': {
        'HOST': os.environ.get("REDIS_HOST", default="redis"),
        'PORT': os.environ.get("REDIS_PORT", default=6379),
        'DB': os.environ.get("REDIS_DB", default=0),
        'DEFAULT_TIMEOUT': 900,
        'REDIS_CLIENT_KWARGS': {},
    },
}

...

# Ändere und Erweitere die Konfiguration für static und media Dateien
STATIC_URL = "/static/"
STATIC_ROOT = BASE_DIR / "static"

MEDIA_URL = "/media/"
MEDIA_ROOT = BASE_DIR / "media"

STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'

...

```

1. **Build and start the project using `docker-compose`.**

```bash
docker-compose up --build
```

Open application in browser on [localhost:8000](http://localhost:8000).

---

## Usage

### Environment Variablen

Alle erforderlichen Umgebungsvariablen werden in der [.env](./.env) Datei gespeichert.

> [!IMPORTANT]
> Bitte verändere die Namen der Variablen in dieser Konfiguration nicht. Dies kann unter Umständen dazu führen, dass wir
das Projekt nicht prüfen und abnehmen können.
>
> Ändere bereits vorhandene Variablen gegebenenfalls mit sinnvollen Werten

---

> [!NOTE]
> [backend.entrypoint.sh](backend.entrypoint.sh) erstellt automatisch einen Superuser basierend auf den
Umgebungsvariablen **`DJANGO_SUPERUSER_USERNAME`, `DJANGO_SUPERUSER_PASSWORD` und `DJANGO_SUPERUSER_EMAIL`**

| Name | Type | Description | Default | Mandatory |
| :--- | :---: | :---------- | :----- | :---: |
| **DJANGO_SUPERUSER_USERNAME** | str | Benutzername für das Django-Admin-Superuser-Konto. Dieser Benutzer wird automatisch erstellt wenn er nicht existiert. | `admin` |   |
| **DJANGO_SUPERUSER_PASSWORD** | str |  Passwort für das Django-Admin-Superuser-Konto. Achte darauf, dass es sicher ist. | `adminpassword` |   |
| **DJANGO_SUPERUSER_EMAIL** | str |  E-Mail-Adresse für das Django-Admin-Superuser-Konto. Wird für die Wiederherstellung des Kontos und für Benachrichtigungen verwendet. | `admin@example.com` |   |
| **SECRET_KEY** | str | Ein geheimer Schlüssel für die Kryptografie in Django. Dieser sollte eine lange, zufällige Zeichenfolge sein und vertraulich behandelt werden. |   | x |
| **DEBUG** | bool | Aktiviert oder deaktiviert den Debug-Modus. Sollte in der Produktion auf False gesetzt werden, um die Offenlegung sensibler Informationen zu verhindern. | `True` |   |
| **ALLOWED_HOSTS** | List[str] | Eine Liste von Strings, die die Host-/Domainnamen darstellen, die diese Django-Site bedienen kann. Wichtig für die Sicherheit. | `[localhost]` |   |
| **CSRF_TRUSTED_ORIGINS** | List[str] | Cors-Headers allowed origins. | `[http://localhost:4200]` |   |
| **DB_NAME** | str | Name der PostgreSQL-Datenbank, zu der eine Verbindung hergestellt werden soll. Wichtig für Datenbankoperationen. | `your_database_name` | x |
| **DB_USER** | str | Benutzername für die Authentifizierung bei der PostgreSQL-Datenbank. | `your_database_user` | x |
| **DB_PASSWORD** | str | Passwort für den PostgreSQL-Datenbankbenutzer. | `your_database_password` | x |
| **DB_HOST** | str | Host-Adresse der PostgreSQL-Datenbank. Normalerweise localhost oder der Dienstname in Docker. | `db` |   |
| **DB_PORT** | int | Portnummer für die Verbindung zur PostgreSQL-Datenbank. | `5432` |   |
| **REDIS_LOCATION** | str | Redis location | `redis://redis:6379/1` |   |
| **REDIS_HOST** | str | Redis host | `redis` |   |
| **REDIS_PORT** | int | Redis port | `6379` |   |
| **REDIS_DB** | int | Redis DB | `0` |   |
| **EMAIL_HOST** | str | SMTP-Server-Adresse für den Versand von E-Mails. | `smtp.example.com` | x |
| **EMAIL_PORT** | int | Portnummer für den SMTP-Server. | `587` |   |
| **EMAIL_USE_TLS** | bool | Aktiviert TLS für den E-Mail-Versand. Empfohlen für die Sicherheit. | `True` |   |
| **EMAIL_USE_SSL** | bool | E-Mail verwendet SSL | `False` |   |
| **EMAIL_HOST_USER** | str | Benutzername für das E-Mail-Konto, das zum Senden von E-Mails verwendet wird. | `your_email_user` | x |
| **EMAIL_HOST_PASSWORD** | str | Passwort für das E-Mail-Konto. Achte auf die Sicherheit. | `your_email_password` | x |
| **DEFAULT_FROM_EMAIL** | str | E-Mailadresse die von Django verwendet wird | `EMAIL_HOST_USER` |   |

### Migrations im Docker Container

Um gemachte Änderungen an der Datenbankstruktur an Docker zu übertragen hast du zwei verschiedene Möglichkeiten:

1. Docker Container komplett neu erstellen (nicht empfohlen)

    - stoppe Docker in der Kommandozeile mit der Tastenkombination `Strg+C`
    - starte Docker neu mit dem Befehl `docker-compose up --build`

2. Führe die Migration direkt im Docker Container aus (besser)

    - erstelle die migrations Dateien direkt im Docker Container

    ```bash
    docker-compose exec web python manage.py makemigrations
    ```

    Dieser Befehl wird direk in der Bash des Docker Containers ausgeführt. (Wir erinnern uns, unser Docker Setup
    enthält im Prinzip ein komplettes Betriebssystem)

    - Führe die Migration aus:

    ```bash
    docker-compose exec web python manage.py migrate
    ```

### requirements.txt

Die Dependencies der Anwendung sind in der Datei [requirements.txt](./requirements.txt) aufgeführt.

Um sie in den Docker Container zu ändern, muss die Anwendung neu erstellt werden.

Um nur die primären (Top-Level) Pakete aufzulisten, die du über `pip` installiert hast - ohne ihre Abhängigkeiten
anzuzeigen - verwende:

```bash
pip list --not-required
```

## Troubleshooting

- **Beim Starten von Docker erhalte ich in der Komandozeile diesen Fehler:**

    ```bash
    unable to get image 'postgres:latest': error during connect:
    Get "http://%2F%2F.%2Fpipe%2FdockerDesktopLinuxEngine/v1.48/images/postgres:latest/json":
    open //./pipe/dockerDesktopLinuxEngine: The system cannot find the file specified.
    ```

    > [!NOTE]
    > Bitte stelle sicher, dass du Docker Desktop gestartet hast.

- **Das Starten von Docker bricht mit der folgenden Meldung in der Konsole ab:**

    ```bash
    videoflix_backend   | exec ./backend.entrypoint.sh: no such file or directory
    videoflix_backend exited with code 255
    ```

    > [!NOTE]
    > Bitte stelle sicher, dass die Datei `backend.entrypoint.sh` mit der End of Line Sequence LF abgespeichert ist.
    >
    > Siehe [Google Suche](https://www.google.com/search?sca_esv=81208bf63503b115&rlz=1C1CHBF_deDE1069DE1069&q=cr+lf+lf+in+vscode&spell=1&sa=X&ved=2ahUKEwihofbto4eNAxXK9bsIHXhtCLYQBSgAegQIDxAB&biw=1920&bih=911&dpr=1)

---
