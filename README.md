# WordPress Docker Compose Setup

This repository provides a Docker Compose setup for running WordPress with a separate MariaDB database service.

The project demonstrates a reproducible multi-container setup with external configuration, persistent database storage, a shared Docker network, and automatic container restart behavior.

## Table of Contents

* [Repository Structure](#repository-structure)
* [Prerequisites](#prerequisites)
* [Quickstart](#quickstart)
* [Usage](#usage)
* [Configuration](#configuration)
* [Testing and Verification](#testing-and-verification)
* [Security Notes](#security-notes)
* [Troubleshooting](#troubleshooting)

## Repository Structure

```text
.
├── docs/
├── .dockerignore
├── .gitignore
├── docker-compose.yaml
├── example.env
└── README.md
```
The docs/ directory contains the project checklist. The main runtime setup is defined in docker-compose.yaml, while example.env provides a safe environment template without real secrets.

## Prerequisites

The following tools are required:

* Git
* Docker
* Docker Compose

The setup can be run on any machine with Docker and Docker Compose installed.

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
http://localhost:8080
```

When running the setup on a cloud VM, replace `localhost` with the public VM IP address.

## Usage

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

> [!WARNING]
> `docker compose down -v` removes the database volume and deletes the WordPress installation data. Use it only when a full reset is intended.

## Configuration

The project uses environment variables from a local `.env` file.

The repository contains `example.env` as a template. The real `.env` file must be created locally and must not be committed.

The following values must match:

```text
WORDPRESS_DB_NAME = MYSQL_DATABASE
WORDPRESS_DB_USER = MYSQL_USER
WORDPRESS_DB_PASSWORD = MYSQL_PASSWORD
```

If these values do not match, WordPress will not be able to connect to the database.

WordPress connects to the database by using the Compose service name:

```text
WORDPRESS_DB_HOST=db
```

Only WordPress is exposed through the configured host port. The database service is used internally through the Docker network.

The database data is persisted through a Docker volume.

Both services use the following restart policy:

```text
restart: unless-stopped
```

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

### Verify WordPress access

Local access:

```text
http://localhost:8080
```

Cloud VM access:

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

After the restart, the WordPress installation and admin login were still available.

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

## Troubleshooting

### Port 8080 is already allocated

If port `8080` is already used by another container or process, Docker cannot start the WordPress service.

Check running containers:

```bash
docker ps
```

Check whether the machine listens on port `8080`:

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
Verify that the matching values are correctly set as described in [Configuration](#configuration).

If the database volume was already initialized with older credentials and no important data must be kept, reset the setup:

```bash
docker compose down -v
docker compose up -d
```

Use `down -v` only when a full reset is intended.
