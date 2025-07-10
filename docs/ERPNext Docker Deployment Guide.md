# ERPNext Docker Deployment Guide

This guide walks you through deploying ERPNext with Docker Compose, using a dedicated `~/gitops` folder for environment files and the final Compose YAML. Follow the **exact filenames** and commands below, replacing only the `<placeholder>` values.

---

## 1. Clone the `frappe_docker` Repository

```bash
cd ~
# Clone the Frappe Docker repository
git clone https://github.com/frappe/frappe_docker.git
cd frappe_docker
```

This provides base and override Compose files under `frappe_docker/`.

---

## 2. Create the GitOps Folder

```bash
mkdir -p ~/gitops
```

This folder will contain your custom `.env` files and the merged Compose YAML.

---

## 3. Create Environment Files (Exact Commands and Final Content)

Below are the exact commands to run. **After running each block**, verify the corresponding file’s contents with `cat`.

### 3.1 Traefik Environment

```bash
# 1. Create traefik.env and write domain
echo 'TRAEFIK_DOMAIN=<traefik-domain>' > ~/gitops/traefik.env

# 2. Append email
echo 'EMAIL=<your-email>' >> ~/gitops/traefik.env

# 3. Append hashed password
echo 'HASHED_PASSWORD=<hashed-password>' >> ~/gitops/traefik.env

# Verify:
cat ~/gitops/traefik.env
```

**Expected `traefik.env`:**

```
TRAEFIK_DOMAIN=<traefik-domain>
EMAIL=<your-email>
HASHED_PASSWORD=<hashed-password>
```

### 3.2 MariaDB Environment

```bash
# Create mariadb.env with root password
echo 'DB_PASSWORD=<your-db-password>' > ~/gitops/mariadb.env

# Verify:
cat ~/gitops/mariadb.env
```

**Expected `mariadb.env`:**

```
DB_PASSWORD=<your-db-password>
```

### 3.3 ERPNext Environment

```bash
# 1. Copy the example.env to erpnext-one.env
cp example.env ~/gitops/erpnext-one.env

# 2. Replace default DB password
sed -i 's/DB_PASSWORD=123/DB_PASSWORD=<your-db-password>/' ~/gitops/erpnext-one.env

# 3. Set external MariaDB host and port
sed -i 's/DB_HOST=/DB_HOST=<mariadb-host>/'    ~/gitops/erpnext-one.env
sed -i 's/DB_PORT=/DB_PORT=<mariadb-port>/'    ~/gitops/erpnext-one.env

# 4. Update site list
sed -i 's|SITES=erp.example.com|SITES=`<site-domain>`|' ~/gitops/erpnext-one.env

# 5. Append router and network identifiers
echo 'ROUTER=erpnext-one' >> ~/gitops/erpnext-one.env
echo 'BENCH_NETWORK=erpnext-one' >> ~/gitops/erpnext-one.env

# Verify:
cat ~/gitops/erpnext-one.env
```

**Expected `erpnext-one.env`:**

```
ERPNEXT_VERSION=<your-version>
DB_PASSWORD=<your-db-password>
DB_HOST=<mariadb-host>
DB_PORT=<mariadb-port>
REDIS_CACHE=
REDIS_QUEUE=
LETSENCRYPT_EMAIL=<your-email>
SITES=`<site-domain>`
ROUTER=erpnext-one
BENCH_NETWORK=erpnext-one
```

---

## 4. Generate the Final Compose YAML

```bash
docker compose \
  --project-name erpnext-one \
  --env-file ~/gitops/erpnext-one.env \
  -f compose.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.multi-bench.yaml \
  -f overrides/compose.multi-bench-ssl.yaml \
  config > ~/gitops/erpnext-one.yaml
```

The file `~/gitops/erpnext-one.yaml` now contains the fully interpolated configuration.

---

## 5. Launch Services

```bash
# Traefik
docker compose \
  --project-name traefik \
  --env-file ~/gitops/traefik.env \
  -f overrides/compose.traefik.yaml \
  -f overrides/compose.traefik-ssl.yaml \
  up -d

# MariaDB
docker compose \
  --project-name mariadb \
  --env-file ~/gitops/mariadb.env \
  -f overrides/compose.mariadb-shared.yaml \
  up -d

# ERPNext Stack
docker compose \
  --project-name erpnext-one \
  -f ~/gitops/erpnext-one.yaml \
  up -d
```

Verify:

```bash
docker compose --project-name erpnext-one ps
```

---

## 6. Create a New Site

```bash
# Enter backend container
docker compose --project-name erpnext-one exec backend bash
cd frappe-bench

# Create site
bench new-site <your-site-domain> \
  --db-root-password <your-db-password> \
  --install-app erpnext \
  --admin-password <your-admin-password>
```

Follow prompts to finish site setup.

---

## 7. Enable Scheduler & Verify

```bash
bench --site <your-site-domain> enable-scheduler
bench --site <your-site-domain> doctor
```

Ensure the scheduler is **Enabled** and workers are **Online**.
Exit container:

```bash
exit
```

Your ERPNext instance should now be available at `https://<your-site-domain>`. Replace placeholders accordingly.

*Last updated: July 2025*
