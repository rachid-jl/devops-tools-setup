# Setting Up GitLab with Docker Compose

This guide provides step-by-step instructions to set up a GitLab instance using Docker Compose with persistent data and proper configuration. The setup ensures your data is not lost even if the container fails or is rebuilt.

---

## Prerequisites

1. **Docker and Docker Compose installed**
   - Install Docker: [Docker Installation Guide](https://docs.docker.com/get-docker/)
   - Install Docker Compose: [Docker Compose Installation Guide](https://docs.docker.com/compose/install/)

2. **Create Required Directories**
   Create directories to store GitLab data:

   ```bash
   mkdir -p ~/Documents/gitlab/{config,logs,data}
   sudo chown -R $USER:$USER ~/Documents/gitlab
   sudo chmod -R 755 ~/Documents/gitlab
   ```

---

## Step 1: Create `docker-compose.yml`

Create a file named `docker-compose.yml` in your working directory with the following content:

```yaml
version: "3.8"

services:
  gitlab:
    image: gitlab/gitlab-ce:latest
    container_name: gitlab
    hostname: mygitlab.com
    restart: always
    ports:
      - "8888:80"   # HTTP
      - "4443:443"  # HTTPS
      - "2222:22"   # SSH
    volumes:
      - ./config:/etc/gitlab        # GitLab configuration
      - ./logs:/var/log/gitlab      # GitLab logs
      - ./data:/var/opt/gitlab      # GitLab repositories and database
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://mygitlab.com'
        gitlab_rails['gitlab_shell_ssh_port'] = 2222
```

---

## Step 2: Start the GitLab Container

Run the following command to start the container:

```bash
docker-compose up -d
```

- **Verify**: Access GitLab at `http://localhost:8888` or replace `localhost` with your server's IP/hostname.

---

## Step 3: Retrieve Initial Root Password

For the first connection to GitLab, you need the initial root password:

1. Connect to the running GitLab container:

   ```bash
   docker exec -it gitlab /bin/bash
   ```

2. View the initial root password stored in the file:

   ```bash
   cat /etc/gitlab/initial_root_password
   ```

3. The output will show the password. Use it to log in as the `root` user at `http://localhost:8888`.

   - You can change the password after logging in.

---

## Step 4: Managing the Container

### Start/Stop the Container
- Start: `docker-compose start`
- Stop: `docker-compose stop`

### Rebuild the Container
To rebuild the container without losing data:

```bash
docker-compose down
docker-compose up -d
```

---

## Step 5: Backing Up Data

### Create a Backup
Run a backup using GitLab's built-in tools:

```bash
docker exec gitlab gitlab-backup create
```

Backup files are stored in `/var/opt/gitlab/backups` (mapped to `~/Documents/gitlab/data/backups`).

### Copy Backup Files
To ensure safety, copy backups to an external location:

```bash
cp ~/Documents/gitlab/data/backups/* /path/to/external/backup/location
```

---

## Step 6: Restoring a Failed Container

If the container fails, follow these steps to restore it:

1. **Stop and Remove the Old Container**
   ```bash
   docker stop gitlab
   docker rm gitlab
   ```

2. **Recreate the Container**
   Recreate the container using the same `docker-compose.yml` file:

   ```bash
   docker-compose up -d
   ```

3. **Verify Data Integrity**
   Check your projects, configurations, and logs to ensure everything is intact.

---

## Step 7: Best Practices

1. **Periodic Backups**: Schedule regular backups to avoid data loss.
2. **Monitor Resources**: Use tools like `docker stats` to monitor container performance.
3. **Secure Access**: Configure firewalls and SSH keys to protect your GitLab instance.

---

Your GitLab instance is now ready and resilient! 🚀
