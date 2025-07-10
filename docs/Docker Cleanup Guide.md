# Docker Cleanup Guide

This guide walks you through each step to completely clean up all your Docker resources, including containers, images, volumes, networks, and build cache.

## Numbered Commands with Explanations

1. **Stop All Running Containers**

   ```bash
   docker stop $(docker ps -q)
   ```

   * `docker ps -q` lists the IDs of all currently running containers, and `docker stop` sends a SIGTERM to each, shutting them down gracefully.

2. **Remove All Containers (Running and Stopped)**

   ```bash
   docker rm $(docker ps -aq)
   ```

   * `docker ps -aq` returns the IDs of all containers (both running and stopped), and `docker rm` deletes each one from your system.

3. **Remove All Images**

   ```bash
   docker rmi -f $(docker images -q)
   ```

   * `docker images -q` lists all image IDs, and `docker rmi -f` forces removal of each image, even if it’s associated with existing tags or containers.

4. **Remove All Volumes**

   ```bash
   docker volume rm $(docker volume ls -q)
   ```

   * `docker volume ls -q` lists all volume names, and `docker volume rm` deletes them one by one (beware: volumes may contain important data).

5. **Remove All Custom Networks (Except `bridge`, `host`, `none`)**

   ```bash
   docker network rm $(docker network ls | grep "bridge\|host\|none" -v | awk '{print $1}')
   ```

   * `docker network ls` lists all networks; `grep -v` filters out the default ones (`bridge`, `host`, `none`); `awk '{print $1}'` extracts each network’s ID for removal.

6. **Prune Build Cache**

   ```bash
   docker builder prune -a -f
   ```

   * `docker builder prune -a` removes all unused build cache; the `-f` flag skips confirmation prompts.

7. **(Optional) System-Wide Cleanup Including Volumes**

   ```bash
   docker system prune -a --volumes -f
   ```

   * Combines pruning of containers, images, networks, and volumes; `-a` removes unused data, `--volumes` includes volumes, and `-f` forces the action without prompting.

---

> **Warning:** These commands will **permanently delete** all Docker resources on your machine. Be absolutely sure you no longer need any of these items before running them.

*Last updated: July 2025*
