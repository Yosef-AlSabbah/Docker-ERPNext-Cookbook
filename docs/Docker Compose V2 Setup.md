# Installing Docker Compose v2 on Linux (Simplified)

A quick, straightforward way to get Docker Compose v2 installed on most Debian/Ubuntu-based systems using the official package.

## 1. Update Package Lists

```bash
sudo apt-get update
```

## 2. Install the Compose v2 Plugin

```bash
sudo apt-get install docker-compose-plugin -y
```

This single package adds the `docker compose` command (with a space) to your Docker CLI.

## 3. Verify the Installation

```bash
docker compose version
```

You should see output like:

```
Docker Compose version v2.x.x
```

---

> **Note:**
>
> * You no longer need the old `docker-compose` Python tool; use `docker compose` instead.
> * Updates are handled automatically when you run:
>
>   ```bash
>   sudo apt-get upgrade
>   ```

*Last updated: July 2025*
