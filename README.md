# LimeSurvey Docker Image

This repository contains Dockerized version of [LimeSurvey](https://limesurvey.org/), an open-source survey software. The Docker images are built for both AMD64 (standard x86_64) and ARM64 (Apple Silicon) architectures and published to GitHub Container Registry at [ghcr.io/sistemica/limesurvey-docker](https://github.com/sistemica/limesurvey-docker/pkgs/container/limesurvey-docker).

## Quick Start

### Pull the Image

```bash
docker pull ghcr.io/sistemica/limesurvey-docker:latest
```

### Run with Docker Compose

Create a `docker-compose.yml` file:

```yaml
services:
  limesurvey:
    image: ghcr.io/sistemica/limesurvey-docker:latest
    restart: always
    depends_on:
      - db
    environment:
      - DB_TYPE=mysql
      - DB_HOST=db
      - DB_PORT=3306
      - DB_NAME=limesurvey
      - DB_USERNAME=limesurvey
      - DB_PASSWORD=limesurvey_password
      - ADMIN_USER=admin
      - ADMIN_PASSWORD=my_secure_password
      - ADMIN_NAME=Administrator
      - ADMIN_EMAIL=admin@example.com
      # URL configuration
      - BASE_URL=
      - PUBLIC_URL=http://localhost:8009
      - URL_FORMAT=path
      - SHOW_SCRIPT_NAME=false
    volumes:
      - limesurvey-data:/var/www/html

  db:
    image: mysql:8.0
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=root_password
      - MYSQL_DATABASE=limesurvey
      - MYSQL_USER=limesurvey
      - MYSQL_PASSWORD=limesurvey_password
    volumes:
      - db-data:/var/lib/mysql

  webserver:
    image: nginx:alpine
    restart: always
    ports:
      - "8009:80"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
      - limesurvey-data:/var/www/html
    depends_on:
      - limesurvey

volumes:
  limesurvey-data:
  db-data:
```

Create an `nginx.conf` file:

```nginx
server {
    listen 80;

    # Set the root directory where all files will be served from
    root /var/www/html;
    index index.php;

    # Handle PHP files
    location ~ \.php$ {
        fastcgi_pass limesurvey:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_intercept_errors on;
        fastcgi_buffer_size 16k;
        fastcgi_buffers 4 16k;
    }

    # For any non-PHP file, try the exact file first then try the directory,
    # and finally fall back to index.php (which will execute LimeSurvey's router)
    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    # Basic settings
    client_max_body_size 100M;
    server_tokens off;
}
```

Then start the containers:

```bash
docker compose up -d
```

Access LimeSurvey at http://localhost:8009

## Environment Variables

### Database Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `DB_TYPE` | Database type (mysql or pgsql) | `mysql` |
| `DB_HOST` | Database host | `mysql` |
| `DB_PORT` | Database port | `3306` |
| `DB_SOCK` | Unix socket (if not using TCP) | - |
| `DB_NAME` | Database name | `limesurvey` |
| `DB_TABLE_PREFIX` | Table prefix | `lime_` |
| `DB_USERNAME` | Database user | `limesurvey` |
| `DB_PASSWORD` | Database password | - |
| `DB_MYSQL_ENGINE` | MySQL storage engine | `MyISAM` |

### Admin Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `ADMIN_USER` | Admin username | `admin` |
| `ADMIN_NAME` | Admin display name | `admin` |
| `ADMIN_EMAIL` | Admin email | `foobar@example.com` |
| `ADMIN_PASSWORD` | Admin password | - |

### Application Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `BASE_URL` | Base URL for the application | - |
| `PUBLIC_URL` | Public URL for the application | - |
| `URL_FORMAT` | URL format (path or get) | `path` |
| `SHOW_SCRIPT_NAME` | Show script name in URL | `true` |
| `DEBUG` | Enable debug mode | `0` |
| `DEBUG_SQL` | Enable SQL debugging | `0` |

### Encryption Configuration

| Variable | Description |
|----------|-------------|
| `ENCRYPT_KEYPAIR` | Encryption key pair |
| `ENCRYPT_PUBLIC_KEY` | Public encryption key |
| `ENCRYPT_SECRET_KEY` | Secret encryption key |
| `ENCRYPT_NONCE` | Encryption nonce |
| `ENCRYPT_SECRET_BOX_KEY` | Secret box key |

## Volumes

For data persistence, the following volumes are used:

- `limesurvey-data:/var/www/html`: Contains all LimeSurvey files including uploads and configuration

## Updating LimeSurvey Version

### For Users

To use a different version of LimeSurvey, simply pull the specific tag:

```bash
docker pull ghcr.io/sistemica/limesurvey-docker:6.12.1+250317
```

Then update your docker-compose.yml to use the specific version:

```yaml
  limesurvey:
    image: ghcr.io/sistemica/limesurvey-docker:6.12.1+250317
```

### For Administrators

The GitHub workflow in this repository allows for easy updates when new LimeSurvey versions are released:

1. Go to the [Actions tab](https://github.com/sistemica/limesurvey-docker/actions) in the GitHub repository
2. Select the "Build and Push Docker Images" workflow
3. Click "Run workflow"
4. Enter the new version (e.g., "6.12.1+250317")
5. Enter the SHA256 checksum for the new version (can be obtained from LimeSurvey's GitHub releases)
6. Click "Run workflow"

The workflow will:
1. Build new Docker images for both AMD64 and ARM64 architectures
2. Tag them with the specified version and push to GitHub Container Registry
3. Update the Dockerfile with the new version and SHA256 checksum
4. Commit the changes with [skip ci] to prevent recursive triggering

## License

This Docker setup is provided under the same license as LimeSurvey itself. LimeSurvey is released under the [GPL License](https://www.limesurvey.org/license).
