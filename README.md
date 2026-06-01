# WordPress Docker Compose Setup

## Table of Contents

* [Repository Description](#repository-description)
* [Repository Structure](#repository-structure)
* [Prerequisites](#prerequisites)
* [Quickstart](#quickstart)
* [Usage](#usage)
* [Configuration](#configuration)
* [Testing and Verification](#testing-and-verification)
* [Security Notes](#security-notes)
* [Troubleshooting](#troubleshooting)

## Repository Description

This repository contains a Docker Compose setup for running a WordPress application with a separate MariaDB database service.

The purpose of this project is to provide a reproducible multi-container setup with environment-based configuration, persistent database storage, an isolated Docker network, and automatic container restart behavior.

The setup consists of two services:

* `wordpress` - runs the WordPress application
* `db` - runs the MariaDB database used by WordPress

## Repository Structure

```text
.
├── .dockerignore
├── .gitignore
├── docker-compose.yaml
├── example.env
└── README.md
```

| File                  | Description                                                                                |
| --------------------- | ------------------------------------------------------------------------------------------ |
| `.dockerignore`       | Excludes local and unnecessary files from the Docker build context.                        |
| `.gitignore`          | Prevents local environment files and temporary files from being committed.                 |
| `docker-compose.yaml` | Defines the WordPress and MariaDB services, volumes, network, ports, and restart behavior. |
| `example.env`         | Provides a safe example configuration without real secrets.                                |
| `README.md`           | Contains the project documentation and usage instructions.                                 |

## Prerequisites

The following tools are required:

* Git
* Docker
* Docker Compose

The project is intended to be deployed on a Linux-based cloud VM. WordPress is exposed on port `8080`.

## Quickstart

Clone the repository via SSH:

```bash
git clone git@github.com:StevanAleksandrov/wordpress-docker-compose.git
cd wordpress-docker-compose
```

Create a local environment file:

```bash
cp example.env .env
```

Edit the `.env` file and replace the placeholder values:

```bash
nano .env
```

Start the setup:

```bash
docker compose up -d
```

Check the running containers:

```bash
docker compose ps
```

Open WordPress in the browser:

```text
http://<VM-IP>:8080
```

Replace `<VM-IP>` with the public IP address of the cloud VM.

## Usage

### Start the setup

```bash
docker compose up -d
```

This starts both services:

* `wordpress`
* `db`

### Stop the setup

```bash
docker compose down
```

This stops the containers but keeps the database volume. The WordPress installation and stored data remain available after starting the setup again.

### Restart the setup

```bash
docker compose down
docker compose up -d
```

### View running containers

```bash
docker compose ps
```

### View logs

```bash
docker compose logs wordpress
docker compose logs db
```

### View only the latest log entries

```bash
docker compose logs wordpress --tail=50
docker compose logs db --tail=50
```

### Reset the setup completely

```bash
docker compose down -v
```

⚠️This removes the database volume and deletes the WordPress installation data. Use this only when a full reset is intended.

## Configuration

The project uses environment variables from a local `.env` file.

The repository contains `example.env` as a template. The real `.env` file must be created locally and must not be committed.

Important variables:

| Variable                | Description                                                                          |
| ----------------------- | ------------------------------------------------------------------------------------ |
| `WORDPRESS_PORT`        | Host port used to access WordPress from the browser.                                 |
| `WORDPRESS_DB_HOST`     | Database host used by WordPress. In this setup it is `db`, the Compose service name. |
| `WORDPRESS_DB_NAME`     | Database name used by WordPress.                                                     |
| `WORDPRESS_DB_USER`     | Database user used by WordPress.                                                     |
| `WORDPRESS_DB_PASSWORD` | Database password used by WordPress.                                                 |
| `MYSQL_DATABASE`        | Database created by MariaDB.                                                         |
| `MYSQL_USER`            | MariaDB user for the WordPress database.                                             |
| `MYSQL_PASSWORD`        | Password for the MariaDB WordPress user.                                             |
| `MYSQL_ROOT_PASSWORD`   | Root password for MariaDB.                                                           |

The following values must match:

```text
WORDPRESS_DB_NAME = MYSQL_DATABASE
WORDPRESS_DB_USER = MYSQL_USER
WORDPRESS_DB_PASSWORD = MYSQL_PASSWORD
```

The WordPress service connects to the database service by using the Compose service name:

```text
WORDPRESS_DB_HOST=db
```

The database data is persisted in the named Docker volume:

```text
db_data
```

Both services are attached to the same Docker network:

```text
wordpress_network
```

The containers use the following restart policy:

```text
restart: unless-stopped
```

This allows Docker to restart the containers automatically if a service process terminates unexpectedly.

## Testing and Verification

### Validate the Compose configuration

```bash
docker compose config
```

### Verify that both containers are running

```bash
docker compose ps
```

Expected result:

```text
wordpress-docker-compose-wordpress-1   Up   0.0.0.0:8080->80/tcp
wordpress-docker-compose-db-1          Up   3306/tcp
```

### Verify that WordPress is reachable on the VM

```text
http://<VM-IP>:8080
```

The WordPress installation page or the configured WordPress site should be visible.

### Verify database persistence

The persistence test was performed by installing WordPress, creating an admin user, stopping the setup, and starting it again without removing volumes:

```bash
docker compose down
docker compose up -d
```

After the restart, the WordPress installation and admin login were still available. This confirms that the database data is persisted through the named Docker volume.

### Verify restart behavior

The restart policy was verified by stopping the main service process inside each container and confirming that Docker restarted the container automatically.

WordPress container verification:

```bash
docker inspect wordpress-docker-compose-wordpress-1 \
  --format 'RestartPolicy={{.HostConfig.RestartPolicy.Name}} Status={{.State.Status}} RestartCount={{.RestartCount}} ExitCode={{.State.ExitCode}}'
```

Verified result:

```text
RestartPolicy=unless-stopped Status=running RestartCount=1 ExitCode=0
```

Database container verification:

```bash
docker inspect wordpress-docker-compose-db-1 \
  --format 'RestartPolicy={{.HostConfig.RestartPolicy.Name}} Status={{.State.Status}} RestartCount={{.RestartCount}} ExitCode={{.State.ExitCode}}'
```

Verified result:

```text
RestartPolicy=unless-stopped Status=running RestartCount=1 ExitCode=0
```

## Security Notes

* No SSH keys are stored in this repository.
* No real passwords, tokens, usernames, or IP addresses are stored in this repository.
* The `.env` file is ignored by Git and must be created locally.
* `example.env` contains only placeholder values.
* Real credentials must be provided only through a local `.env` file or through the required project submission channel.
* The database service is not exposed to the public internet.
* Only WordPress is exposed through the configured host port.

## Troubleshooting

### Port 8080 is already allocated

If port `8080` is already used by another container or process, Docker cannot start the WordPress service.

Check running containers:

```bash
docker ps
```

Check whether the VM listens on port `8080`:

```bash
sudo ss -lntp | grep :8080
```

Stop the conflicting container if it is no longer needed:

```bash
docker stop <container-name>
```

### Error establishing a database connection

This usually means that WordPress cannot connect to MariaDB.

Check logs:

```bash
docker compose logs db --tail=80
docker compose logs wordpress --tail=80
```

Verify that the following values match in `.env`:

```text
WORDPRESS_DB_NAME = MYSQL_DATABASE
WORDPRESS_DB_USER = MYSQL_USER
WORDPRESS_DB_PASSWORD = MYSQL_PASSWORD
```

If the database volume was already initialized with older credentials and no important data must be kept, reset the setup:

```bash
docker compose down -v
docker compose up -d
```

Use `down -v` only when a full reset is intended.
