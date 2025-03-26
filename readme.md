# 🧱 Postgres Stack Template with Dynamic Overrides

This repository contains the core infrastructure definition used to deploy a postgres server. It serves as a template for building a production-ready stack using **Podman Compose**. This repo is designed to be extended through [overrides](https://github.com/codeworks-templates/posgres-stack-overrides) repo who want to customize their application stack with their own backend and frontend services.

## 🧩 Overview

This stack includes:
- PostgreSQL database with SSL, backups, and monitoring
- Prometheus + Grafana for metrics and observability
- Caddy as a reverse proxy for HTTPS and static file serving
- Ready extensions

## 🔐 Secrets and Configuration

This core template does not include any sensitive secrets or specific configurations. All environment-specific values (e.g., environment variables, custom services, domain names) are expected to be injected by the **override repositories** using Ansible and GitHub Actions.


---

## 📦 Services Overview

| Service             | Purpose                                                                 |
| ------------------- | ----------------------------------------------------------------------- |
| `postgres`          | The main PostgreSQL 16 database server                                  |
| `pg_backup`         | A scheduled service that dumps backups of your app DB every 24 hours    |
| `caddy`             | Reverse proxy for HTTPS (Let's Encrypt), serves the frontend and APIs   |
| `node_exporter`     | Monitors server-level metrics (CPU, memory, disk) for Prometheus        |
| `postgres_exporter` | Monitors PostgreSQL metrics (queries, locks, connections)               |
| `prometheus`        | Collects metrics from exporters and provides an interface to query them |
| `grafana`           | Visual dashboard for your Prometheus metrics                            |

---

## 🚀 Running Locally

### Prerequisites

- Podman + podman-compose
- A `.env` file (see `.env.dev` or create your own)
- Ansible (for provisioning new environments)

### Steps

1. Clone this repo
2. Create `.env`  
3. Update the variables in `.env` for your local environment
4. Run:

```bash
podman-compose --env-file .env up -d
```

## 🔒 GitHub Actions & Secrets

This template includes GitHub Actions for:

- Automatic deployment to **staging** on every push to `main`
- Manual promotion to **production** via workflow dispatch

### Required Secrets

To use GitHub Actions for deployment, set the following repository secrets:

| Name                     | Description                         |
| ------------------------ | ----------------------------------- |
| `SSH_USER`               | SSH username for your server        |
| `SSH_PRIVATE_KEY`        | Private key for SSH authentication  |
| `STAGING_HOST`           | IP or domain of your staging server |
| `PROD_HOST`              | IP or domain of your prod server    |
| `ANSIBLE_VAULT_PASSWORD` | Password for Ansible vault          |

Set these in your GitHub repo:  
**Settings → Secrets and variables → Actions → New repository secret**

GitHub Actions will:
- SSH into your server
- Pull the latest changes
- Run `podman-compose up -d` with the appropriate `.env` file

## 🔧 First-Time Environment Setup (Ansible)

Use the `Provision New Environment` GitHub Action to automatically prepare a new server:

- Installs Podman and dependencies
- Clones this repo
- Copies the appropriate `.env` file
- Starts the stack using `podman-compose`

---

## 🔐 DNS and HTTPS via Cloudflare

- Create `A` records in your Cloudflare dashboard pointing to your server
- Use "DNS Only" (disable the orange cloud) so Let's Encrypt works
- Caddy handles certificate generation and HTTPS automatically

---

## 🧰 Directory Structure

```bash
postgres-stack-template/
├── docker-compose.yml
├── .env.dev / .env.prod
├── www/                     # static frontend files (built from your app)
├── caddy/
│   └── Caddyfile            # reverse proxy routes
├── pgadmin-init/            # auto-register pgAdmin connections
├── postgresql-config/       # custom Postgres config
├── postgres-ssl/            # SSL certs for postgres
├── monitoring/              # prometheus.yml, grafana setup
```

---

## 📈 Monitoring and Observability

- **Prometheus** collects metrics from `node_exporter` and `postgres_exporter`
- **Grafana** provides a dashboard to visualize these metrics
- Access Grafana at `http://grafana.dev.example.com` (default login: admin/admin)
- Customize your Grafana dashboards by importing pre-built templates or creating your own
- Prometheus and Grafana are configured to scrape metrics from the respective exporters every 15 seconds
- You can adjust the scrape intervals in `monitoring/prometheus.yml` as needed
- Grafana can be extended with plugins for additional visualization options
- Use Grafana's alerting features to set up notifications for critical metrics
- Monitor your PostgreSQL performance, query times, and system resource usage to ensure optimal operation
- **Node Exporter** collects system-level metrics (CPU, memory, disk usage) and exposes them to Prometheus
- **PostgreSQL Exporter** collects database-level metrics (queries, locks, connections) and exposes them to Prometheus
- Both exporters are configured to run as separate containers and are included in the `docker-compose.yml` file
- They are set to restart automatically unless stopped, ensuring continuous monitoring of your system and database performance

## 📁 inventory.ini — What It's For

While this repo includes a sample `inventory.ini` file, **it's only a reference for instructors or local testing**.

During deployment, `inventory.ini` is dynamically generated in the student's GitHub Action workflow like this:

```bash
echo "[staging]" > inventory.ini
echo "target ansible_host=${{ secrets.STAGING_HOST }} ansible_user=${{ secrets.SSH_USER }}" >> inventory.ini
```

This allows students to deploy to their own servers without editing core Ansible files or maintaining forked infrastructure code.

## 🧪 Local Testing (Optional)

If you are running this locally (as an instructor or devops maintainer), you can:

1. Fill in the real IP addresses and SSH users in `inventory.ini`
2. Run the playbook:

```bash
ansible-playbook -i inventory.ini playbook.yml --extra-vars "environment=prod"
```

This will install Podman, pull the latest stack, and launch it based on the provided `.env`.