## 🚀 Ansible Roles

**Ansible roles** is a project containing several Ansible roles that allow you to configure a Debian server easily and efficiently.

> [!WARNING]
> Roles have only been tested with `debian-12.10.0`

---

## 📑 Table of Contents

1. [Integration](#-integration)
2. [Available Roles](#-available-roles)
   - [apt](#-apt)
   - [auditd](#-auditd)
   - [blackbox](#-blackbox)
   - [caddy](#-caddy)
   - [cadvisor](#-cadvisor)
   - [convertx](#-convertx)
   - [crontab](#-crontab)
   - [dns](#-dns)
   - [docker](#-docker)
   - [fail2ban](#-fail2ban)
   - [gitea](#-gitea)
   - [grafana](#-grafana)
   - [grub](#-grub)
   - [homepage](#-homepage)
   - [iptables](#-iptables)
   - [journalctl](#-journalctl)
   - [lldap](#-lldap)
   - [logindefs](#-logindefs)
   - [loki](#-loki)
   - [nginx](#-nginx)
   - [ntp](#-ntp)
   - [pfsense](#-pfsense)
   - [privatebin](#-privatebin)
   - [prometheus](#-prometheus)
   - [promtail](#-promtail)
   - [proxmox](#-proxmox)
   - [proxy](#-proxy)
   - [pythonrunner](#-pythonrunner)
   - [rsyslog](#-rsyslog)
   - [semaphore](#-semaphore)
   - [snmpexporter](#-snmpexporter)
   - [squid](#-squid)
   - [sshd](#-sshd)
   - [sshkeys](#-sshkeys)
   - [sysctl](#-sysctl)
   - [trust_ca](#-trust_ca)
   - [uptime_kuma](#-uptime_kuma)
   - [users](#-users)
   - [vaultwarden](#-vaultwarden)
   - [version](#-version)

---

## 📦 Integration

You can integrate this repository into a larger Ansible project with:

```bash
git submodule add https://github.com/6C656C65/ansible_roles roles
```

---

## 🔧 Available Roles

### 📄 `apt`

<details>
<summary>Click to expand the `apt` role documentation</summary>

Manages the installation and update of system packages using `apt`.

**✅ Features**

- Updates the `apt` package cache to ensure the latest package information.
- Upgrades all installed packages to the latest version.
- Installs a predefined set of system packages.
- Purges old packages that are no longer needed.

**📁 Structure**

```text
apt/
├── defaults/
│   └── main.yml
├── handlers/
│   └── main.yml
├── tasks/
│   └── main.yml
```

**⚙️ Defaults (`defaults/main.yml`)**

```yaml
packages_to_install:
  - "ntp"
  - "openssh-server"
  - "ca-certificates"
```

- `packages_to_install`: List of packages to be installed on the system. You can add additional packages to this list as needed.

**📋 Tasks**

- **Update apt package cache**: Updates the local package cache to ensure that the latest package versions are available.
- **Upgrade all packages**: Upgrades all installed packages to the latest version available.
- **Install required system packages**: Installs the packages specified in `packages_to_install`. This includes essential packages such as NTP, OpenSSH, and CA certificates.
  - This task triggers the handler `Purge old packages` to clean up obsolete packages after installation.

**🔁 Handlers**

- **Purge old packages**: Removes packages that are no longer needed (those marked for removal) by purging them from the system.

</details>

### 📄 `auditd`

<details>
<summary>Click to expand the <code>auditd</code> role documentation</summary>

This role installs and configures the `auditd` daemon on Debian-based systems. It enforces a custom set of audit rules, manages logging configuration, and ensures log rotation with tailored settings.

**✅ Features**

* Installs `auditd` and required packages
* Applies audit rules via `defaults/main.yml`
* Configures `/etc/audit/auditd.conf` using a Jinja2 template
* Deploys a custom `logrotate` configuration
* Ensures `auditd` service is restarted when necessary

**📁 Structure**

```text
auditd/
├── defaults/
│   └── main.yml
├── files/
│   └── logrotate.d/
│       └── audit
├── handlers/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
│   └── auditd.conf
```

**⚙️ Defaults (defaults/main.yml)**

```yaml
auditd_rules:
  - "-D"
  - "-b 8192"
  - "--backlog_wait_time 60000"
  - "-f 1"
  - "-a always,exit -F arch=b64 -S execve -k exec_commands"
  - "-a always,exit -F arch=b32 -S execve -k exec_commands"
  - "-w /etc/passwd -p wa -k passwd_changes"
  - "-w /etc/shadow -p wa -k shadow_changes"
  - "-a always,exit -F arch=b64 -S setuid,setgid -k privilege_change"
  - "-a always,exit -F arch=b32 -S setuid,setgid -k privilege_change"
```

* `auditd_rules`: List of audit rules to apply. These are written directly to `/etc/audit/rules.d/audit.rules`.

**📋 Tasks**

* Installs the `auditd` package
* Configures `/etc/audit/auditd.conf` using a Jinja2 template
* Generates `/etc/audit/rules.d/audit.rules` using the `auditd_rules` list
* Deploys a custom `logrotate` configuration for audit logs
* Notifies handler to restart `auditd` after any changes

**🔁 Handlers**

```yaml
- name: Restart auditd via systemd
  ansible.builtin.systemd:
    name: auditd.service
    state: restarted
    enabled: true
```

This handler ensures that changes are applied immediately by restarting the service.

**📄 Template: auditd.conf**

The template defines audit daemon behavior, including file paths, disk space handling, and log format. It follows secure and production-friendly defaults, with `log_format = ENRICHED` and log rotation triggers.

**🌀 Log Rotation: logrotate.d/audit**

```conf
/var/log/audit/audit.log {
    daily
    missingok
    rotate 0
    size 100M
    create 0600 root root
    postrotate
        /usr/sbin/service auditd reload > /dev/null 2>&1 || true
    endscript
    compress
    delaycompress
    notifempty
}
```

This configuration ensures logs are rotated safely and auditd is reloaded afterward.

</details>

### 📄 `blackbox`

<details>
<summary>Click to expand the <code>blackbox</code> role documentation</summary>

Installs and configures the [Blackbox Exporter](https://github.com/prometheus/blackbox_exporter), a Prometheus exporter for blackbox probing, using Docker Compose to manage the service.

**✅ Features**

* Creates the necessary directory structure for Blackbox Exporter configuration
* Deploys a templated `docker-compose.yml` for Docker container setup
* Configures Blackbox Exporter with a custom `blackbox.yml` configuration file
* Supports HTTP probing with customizable timeout, proxy settings, and SSL verification
* Allows easy modifications of the configuration file via a Jinja2 template

**📁 Directory Structure**

```text
blackbox/
├── defaults/
│   └── main.yml        # Default variables for blackbox exporter configuration
├── handlers/
│   └── main.yml        # Handlers for restarting the blackbox-exporter service
├── tasks/
│   └── main.yml        # Main tasks for setup and configuration
├── templates/
│   ├── blackbox.yml    # Blackbox exporter configuration template
│   └── docker-compose.yml  # Docker Compose template for Blackbox Exporter setup
```

**⚙️ Default Configuration (`defaults/main.yml`)**

```yaml
blackbox_directory: /opt/blackbox  # Directory where Blackbox Exporter configuration is stored
```

* `blackbox_directory`: Specifies the root path where Blackbox Exporter will be installed.

**📋 Tasks**

* Creates necessary directories for Blackbox Exporter configuration
* Deploys the `docker-compose.yml` file using Jinja2 templating
* Templates the `blackbox.yml` configuration file and notifies Blackbox Exporter restart
* Starts the Blackbox Exporter service using Docker Compose (`docker-compose up -d`)

**📝 Templates**

* `templates/blackbox.yml`: Defines the Blackbox Exporter configuration for probing HTTP services.
* `templates/docker-compose.yml`: Defines the Blackbox Exporter container setup, volumes, and configuration command.

**🔧 Requirements**

* Docker and Docker Compose must be installed on the target machine

**📋 Handlers**

* **Restart Blackbox Exporter**: Ensures that Blackbox Exporter restarts when the configuration is updated.

```yaml
- name: Restart blackbox-exporter
  ansible.builtin.command: docker-compose restart blackbox-exporter
  args:
    chdir: "{{ blackbox_directory }}"
  changed_when: false
```

</details>

### 📄 `caddy`

<details>
<summary>Click to expand the <code>caddy</code> role documentation</summary>

Deploys and configures the [Caddy](https://caddyserver.com/) web server using Docker with flexible reverse proxy rules and support for custom TLS certificates. Configuration is fully dynamic per host and domain.

**✅ Features**

* Manages Caddy deployment per host using Docker Compose
* Supports multiple domains per host with custom reverse proxy logic
* Allows full customization of `Caddyfile` via Jinja templating
* Accepts user-provided TLS certificates (intermediate CA)
* Automatically sets up directories, configurations, and starts Caddy
* Dynamic logging with per-host hostname injection

**📁 Structure**

```text
caddy/
├── defaults/
│   └── main.yml
├── files/
│   ├── intermediate_ca.crt
│   └── intermediate_ca.key
├── handlers/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
│   ├── docker-compose.yml
│   └── Caddyfile
```

**⚙️ Defaults (defaults/main.yml)**

```yaml
caddy_directory: /opt/caddy
# caddy_cert_source_override: /opt/caddy/certs

backend:
  hostA:
    ...
```

* `caddy_directory`: Path where Caddy configuration and Docker setup are deployed.
* `caddy_cert_source_override` (*optional*): Override the source directory for certificates.
* `backend`: Per-host domain configuration block with custom reverse proxy and logging logic (templated).

**📋 Tasks**

* Detect backends for the current host
* Create necessary directories (`/opt/caddy` and `/opt/caddy/certs`)
* Render and deploy `Caddyfile` and `docker-compose.yml` for the host
* Copy provided TLS certificate files into the correct path
* Start Caddy with Docker Compose
* Automatically restart the Caddy container if configuration or certs change

**🔁 Handlers**

* `Restart caddy docker`: Restarts the Caddy service in the deployment directory via Docker Compose

**📦 Docker Compose Template**

* Runs Caddy in a container
* Mounts host configuration and certificates
* Exposes standard HTTP/HTTPS ports (80, 443)
* Uses environment variables if necessary (template-ready)

**📄 Caddyfile Template**

* Generates a custom `Caddyfile` with dynamic domain blocks
* Supports advanced reverse proxying, including WebSocket support
* Logs requests with host-specific data
* Injects backend rules from `defaults/main.yml`

</details>

### 📄 `cadvisor`

<details>
<summary>Click to expand the <code>cadvisor</code> role documentation</summary>

Deploys and manages cAdvisor using Docker Compose to monitor container resource usage on the host system.

✅ Features

- Creates required directory structure for cAdvisor deployment.
- Deploys a customizable docker-compose.yml using a Jinja2 template.
- Starts the cAdvisor container with Docker Compose.
- Provides a handler to restart cAdvisor if configuration changes.

📁 Structure

```text
cadvisor/
├── defaults/
│   └── main.yml
├── handlers/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
│   └── docker-compose.yml
```

⚙️ Defaults (defaults/main.yml)

```yaml
cadvisor_directory: "/opt/cadvisor"
```

- `cadvisor_directory`: Base directory on the target system where cAdvisor will be deployed.

📋 Tasks

- Create required directories: Ensures the base directory for cAdvisor exists with correct permissions.
- Copy docker-compose.yml: Renders and copies the Docker Compose file from template to the target directory. Triggers the handler to restart the service if the file is changed.
- Start cadvisor containers: Uses docker-compose to bring up the cAdvisor service in detached mode.

🔁 Handlers

- `Restart cadvisor`: Restarts the cAdvisor container using docker-compose when configuration is updated.

</details>

### 📄 `convertx`

<details>
<summary>Click to expand the <code>convertx</code> role documentation</summary>

Installs and configures **ConvertX**, a self-hosted file conversion service, using **Docker Compose** on Debian-based systems.

This role deploys ConvertX as a Docker container, manages its persistent storage, and ensures the service is started automatically.

## ✅ Features

* Creates the directory structure for ConvertX
* Deploys a templated `docker-compose.yml`
* Runs ConvertX as a Docker container
* Uses a named Docker volume for persistent data
* Simple and minimal configuration

## 📁 Directory Structure

```text
convertx/
├── defaults/
│   └── main.yml
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
│   └── docker-compose.yml
```

## ⚙️ Default Configuration (`defaults/main.yml`)

```yaml
convertx_directory: /opt/convertx
```

### Variables

* `convertx_directory`
  Base directory where the ConvertX Docker Compose configuration is deployed.

## 📋 Tasks

* **Create base directory**
  Creates the ConvertX installation directory.

* **Deploy docker-compose.yml**
  Renders and copies the Docker Compose configuration using a Jinja2 template.

* **Start ConvertX**
  Launches the ConvertX container using `docker-compose up -d`.

```yaml
- name: Start ConvertX using Docker Compose
  ansible.builtin.command: docker-compose up -d
  args:
    chdir: "{{ convertx_directory }}"
  changed_when: false
```

## 🔁 Handlers

* **Restart ConvertX**
  Restarts the ConvertX service when the Docker Compose configuration changes.

```yaml
- name: Restart ConvertX
  ansible.builtin.command: docker-compose restart
  args:
    chdir: "{{ convertx_directory }}"
  changed_when: false
```

## 🐳 Docker Configuration

* Image: `ghcr.io/c4illin/convertx`
* Container name: `convertx`
* Restart policy: `unless-stopped`
* Persistent data stored in a named Docker volume: `convertx-data`
* Uses a dedicated Docker network: `convertx`

```yaml
volumes:
  - convertx-data:/app/data
```

## 🔧 Requirements

* Debian-based system
* Docker installed
* Docker Compose installed

## ⚠️ Notes

* This role does **not** expose ports by default — adjust the `docker-compose.yml` template if external access is required.
* Data persistence is handled via a Docker named volume.
* Service lifecycle is fully managed via Docker Compose.

</details>

### 📄 `crontab`

<details>
<summary>Click to expand the <code>crontab</code> role documentation</summary>

Hardens system cron configuration by enforcing strict permissions on periodic cron directories on Debian systems.

**✅ Features**

* Secures cron execution directories by enforcing restrictive permissions
* Reduces risk of unauthorized script execution via cron
* Applies consistent permission policy across all periodic cron jobs

**📁 Structure**

```text
crontab/
├── meta/
│   └── main.yml
└── tasks/
    └── main.yml
```

**📋 Tasks**

* **Fix cron permissions**: Ensures the following cron directories are restricted to the owner (`root`) only:

  * `/etc/cron.hourly`
  * `/etc/cron.daily`
  * `/etc/cron.weekly`
  * `/etc/cron.monthly`

```text
Permissions applied: 0700 (rwx------)
```

**🔧 Requirements**

* Cron service installed (`cron` package)

**⚠️ Notes**

* This role assumes all periodic cron jobs are managed by `root`.
* Custom or user-managed cron jobs placed in these directories will no longer be accessible.
* This role does not modify:

  * `/etc/crontab`
  * User crontabs (`crontab -e`)
  * `/etc/cron.d/`

</details>

### 📄 `dns`

<details>
<summary>Click to expand the <code>dns</code> role documentation</summary>

Configures the DNS settings for the system, including the domain, search domains, and nameservers.

**✅ Features**

- Configures the `/etc/resolv.conf` file with domain, search, and nameserver settings.
- Restarts the networking service after DNS configuration changes.

**📁 Structure**

```text
dns/
├── defaults/
│   └── main.yml
├── handlers/
│   └── main.yml
├── tasks/
│   └── main.yml
```

**⚙️ Defaults (`defaults/main.yml`)**

```yaml
domain: local
search: local
nameserver: 10.0.0.254
```

- `domain`: The default DNS domain name.
- `search`: The domain search list.
- `nameserver`: The IP address of the DNS server.

**📋 Tasks**

- Copies the DNS configuration to `/etc/resolv.conf` with the values specified in `defaults/main.yml`.
- Notifies the `Restart networking service` handler to apply the changes.

**🔁 Handlers**

- `Restart networking service`: Restarts the networking service to apply the new DNS settings.

</details>

### 📄 `docker`

<details>
<summary>Click to expand the <code>docker</code> role documentation</summary>

Installs Docker and Docker Compose, configures proxy settings, and manages user access.

**✅ Features**

- Installs Docker and Docker Compose packages.
- Adds specified users to the `docker` group.
- Configures system-wide Docker proxy settings via systemd drop-in.
- Reloads systemd and restarts Docker when proxy settings change.

**📁 Structure**

```text
docker/
├── defaults/
│   └── main.yml
├── handlers/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
│   └── override.conf
```

**⚙️ Defaults (`defaults/main.yml`)**

```yaml
http_proxy: "http://proxy.company.com:3128"
https_proxy: "https://proxy.company.com:3128"
no_proxy: "localhost,127.0.0.1,10.0.0.0/16,192.168.0.0/16,172.16.0.0/12"

docker_users:
  - user
```

- `http_proxy`: Proxy server for HTTP traffic.
- `https_proxy`: Proxy server for HTTPS traffic.
- `no_proxy`: Comma-separated list of addresses that bypass the proxy.
- `docker_users`: List of users to add to the `docker` group.

**📋 Tasks**

- Installs `docker` and `docker-compose` using the system package manager.
- Adds each user listed in `docker_users` to the `docker` group.
- Creates the directory `/etc/systemd/system/docker.service.d` if it doesn't exist.
- Applies proxy settings by templating `override.conf`.
- Notifies handlers to reload systemd and restart Docker.

**🔁 Handlers**

- `Reload systemd`: Runs `systemctl daemon-reload` to apply service changes.
- `Restart docker`: Restarts the Docker service to apply updated configuration.

</details>

### 📄 `fail2ban`

<details>
<summary>Click to expand the <code>fail2ban</code> role documentation</summary>

Configures and installs Fail2Ban to protect the system against brute-force attacks and suspicious login attempts.

**✅ Features**

- Installs the Fail2Ban package from the system repository.
- Ensures jail.local exists as a safe copy of the default jail.conf.
- Prepares the system for further custom jail configurations.
- Sets correct file ownership and permissions for jail.local.

**📁 Structure**

```text
fail2ban/
├── defaults/
│   └── main.yml
├── handlers/
│   └── main.yml
├── tasks/
│   └── main.yml
```

**⚙️ Defaults (`defaults/main.yml`)**

```yaml
fail2ban_config_file: /etc/fail2ban/jail.local
fail2ban_source_file: /etc/fail2ban/jail.conf
```

- `fail2ban_config_file`: Path to the fail2ban configuration file to be used (usually jail.local).
- `fail2ban_source_file`: Default configuration file used as the template for jail.local.

**📋 Tasks**

- **Install Fail2Ban package**: Installs the fail2ban package using apt and ensures the package cache is up to date.
- **Ensure jail.local exists from default jail.conf**: Copies the default jail.conf file to jail.local to allow custom configuration while preserving upstream updates. File permissions are set to 0644 and owned by root.

**🔁 Handlers**

- `Restart fail2ban`: Restarts the Fail2Ban service when the configuration file is modified.

</details>

### 📄 `gitea`

<details>
<summary>Click to expand the <code>gitea</code> role documentation</summary>

Deploys and manages [Gitea](https://gitea.io/), a self-hosted Git service, using Docker Compose on Debian systems.

**✅ Features**

* Deploys Gitea using Docker Compose
* Creates and manages a dedicated installation directory
* Uses a persistent Docker volume for repositories and configuration
* Exposes SSH access for Git operations on a custom port
* Simple restart handler for configuration changes

**📁 Structure**

```text
gitea/
├── defaults/
│   └── main.yml
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
│   └── docker-compose.yml
```

**⚙️ Defaults (`defaults/main.yml`)**

```yaml
gitea_directory: /opt/gitea
gitea_url: gitea.company.com
gitea_ssh_port: 2222
```

* `gitea_directory`: Base directory on the target system where Gitea is deployed.
* `gitea_url`: Public URL used to access the Gitea web interface.
* `gitea_ssh_port`: SSH port exposed by the Gitea container for Git operations.

**📋 Tasks**

* **Create necessary directories**: Ensures the Gitea installation directory exists with proper permissions.
* **Copy docker-compose configuration**: Renders and copies the `docker-compose.yml` file using a Jinja2 template.
* **Start Gitea service**: Launches the Gitea container in detached mode using Docker Compose.

**🔁 Handlers**

* **Restart Gitea Docker**: Restarts the Gitea container when configuration changes.

```yaml
- name: Restart gitea docker
  ansible.builtin.command: docker compose restart
  args:
    chdir: "{{ gitea_directory }}"
  changed_when: false
```

**📦 Docker Compose**

* Uses the official Gitea Docker image (`docker.gitea.com/gitea`)
* Persists repositories, configuration, and data using a named Docker volume (`gitea-data`)
* Exposes SSH for Git access on a configurable port (`gitea_ssh_port`)
* Synchronizes container timezone with the host
* Uses an isolated Docker network (`gitea`)

</details>

### 📄 `grafana`

<details>
<summary>Click to expand the <code>grafana</code> role documentation</summary>

Installs and configures Grafana using Docker, customizes users, alerts, dashboards, and datasources.

**✅ Features**

* Installs Grafana via Docker Compose.
* Configures Grafana settings including global settings, security, and user preferences.
* Sets up SMTP for email notifications.
* Provisions Grafana dashboards and datasources.
* Allows customization of alerting settings.

**📁 Structure**

```text
grafana/
├── defaults/
│   └── main.yml
├── files/
│   ├── dashboards/
│   └── provisioning/
├── handlers/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
│   └── docker-compose.yml
```

**⚙️ Defaults (`defaults/main.yml`)**

```yaml
grafana_directory: /opt/grafana
grafana_external_networks:
  ...
grafana_global:
  ...
grafana_security:
  ...
grafana_users:
  ...
grafana_smtp:
  ...
```

* `grafana_directory`: Directory where Grafana will be installed.
* `grafana_external_networks`: List of external networks to connect Grafana to.
* `grafana_global`: Global configuration settings for Grafana.
* `grafana_security`: Security settings for Grafana (admin credentials).
* `grafana_users`: User preferences for Grafana.
* `grafana_smtp`: SMTP settings for email notifications.

**📋 Tasks**

* Installs Grafana and Docker Compose.
* Configures Grafana based on `defaults/main.yml` variables.
* Creates necessary directories for Grafana and copies configuration files.
* Provisions dashboards, datasources, and alerting rules.
* Starts Grafana using Docker Compose.

**🔁 Handlers**

* `Restart grafana`: Restarts the Grafana container to apply updated configurations.

</details>

### 📄 `grub`

<details>
<summary>Click to expand the <code>grub</code> role documentation</summary>

Configures a GRUB password for boot-time protection and blacklists specific kernel modules to harden system security.

**✅ Features**

- Sets a `password_pbkdf2` in `/etc/grub.d/40_custom` to protect GRUB access.
- Regenerates GRUB configuration using `update-grub`.
- Blacklists USB storage and FireWire kernel modules for security.

**📁 Structure**

```text
grub/
├── defaults/
│   └── main.yml
├── tasks/
│   └── main.yml
```

**⚙️ Defaults (`defaults/main.yml`)**

```yaml
grub_password: "grub.pbkdf2.sha512.10000.929D04D84DD6906946D134E7A7FB1644DB5785B8B71B9900D5B45E01B50128E66486865B2222644ACAE29778CE61161AD6680470D827B508A61458C302C5B66C.ACD515811D6CC1948F6EF89FB58881EA3670583275D3014C510C5C6478B55046FF5DDDD7669FA56451D90680ACF4968891338BD1710CCBA653433BE7B4E313B0"
```

- `grub_password`: The PBKDF2 hash for the GRUB superuser password.
  - **Default password is** `changeme`
  - **⚠️ It is strongly recommended to change it.**

To generate a new GRUB password hash, use:

```bash
grub-mkpasswd-pbkdf2
```

**📋 Tasks**

- Inserts a GRUB `superuser` and `password_pbkdf2` block into `/etc/grub.d/40_custom`.
- Runs `update-grub` to apply the new GRUB configuration.
- Adds the following kernel modules to the blacklist:
  - `usb_storage`
  - `firewire_core`
  - `firewire_ohci`

</details>

### 📄 `homepage`

<details>
<summary>Click to expand the <code>homepage</code> role documentation</summary>

Installs and configures **Homepage** ([gethomepage.dev](https://gethomepage.dev)), a modern self-hosted dashboard for services and bookmarks, using **Docker Compose** on Debian-based systems.

This role deploys Homepage as a Docker container, manages its configuration files and custom icons, and ensures the service is started automatically.

## ✅ Features

* Creates the directory structure for Homepage
* Deploys a templated `docker-compose.yml`
* Deploys Homepage configuration files (`services`, `bookmarks`, `widgets`, `settings`)
* Supports custom icons
* Allows overriding default configuration files
* Runs Homepage as a Docker container via Docker Compose

## 📁 Directory Structure

```text
homepage/
├── defaults/
│   └── main.yml
├── files/
│   ├── config/
│   │   ├── bookmarks.yaml
│   │   ├── services.yaml
│   │   ├── settings.yaml
│   │   └── widgets.yaml
│   └── icons/
│       └── service_b.svg
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
│   └── docker-compose.yml
```

## ⚙️ Default Configuration (`defaults/main.yml`)

```yaml
homepage_directory: /opt/homepage
# homepage_files_override: ./homepage_files
```

### Variables

* `homepage_directory`
  Base directory where Homepage is installed.

* `homepage_files_override` *(optional)*
  Path to a custom directory containing Homepage configuration files (`config/`, `icons/`).
  If not set, the role uses the default files shipped with the role.

## 📋 Tasks

* **Create directory structure**
  Creates base, config, and icons directories.

* **Deploy docker-compose.yml**
  Renders and copies the Docker Compose configuration template.

* **Copy configuration files**
  Copies Homepage configuration files and icons to the target host.
  Triggers a service restart when files change.

* **Start Homepage**
  Launches Homepage using `docker-compose up -d`.

```yaml
- name: Start HomePage using Docker Compose
  ansible.builtin.command: docker-compose up -d
  args:
    chdir: "{{ homepage_directory }}"
  changed_when: false
```

## 🔁 Handlers

* **Restart HomePage**
  Restarts the Homepage container when configuration files change.

```yaml
- name: Restart HomePage
  ansible.builtin.command: docker-compose restart
  args:
    chdir: "{{ homepage_directory }}"
  changed_when: false
```

## 🐳 Docker Configuration

* Image: `ghcr.io/gethomepage/homepage:latest`
* Container name: `homepage`
* Restart policy: `unless-stopped`
* Mounted volumes:

  * `./config` → `/app/config`
  * `./icons` → `/app/public/icons`
* Environment variables:

  * `HOMEPAGE_ALLOWED_HOSTS`

## ⚠️ Notes

* Homepage configuration files are **static YAML files** and not templated by default.
* To customize configuration:

  * copy the `files/` directory
  * set `homepage_files_override` to your custom path
* Network exposure (ports, reverse proxy) must be handled externally.
* The role assumes Docker networking is sufficient for service access.

</details>

### 📄 `iptables`

<details>
<summary>Click to expand the <code>iptables</code> role documentation</summary>

Configures and manages firewall rules using `iptables` to secure the system. This role allows you to define custom firewall rules, set default policies, log blocked traffic, and persist the rules across reboots.

**✅ Features**

* Sets the default policy for incoming traffic to `ACCEPT` and later switches it to `DROP`.
* Adds custom firewall rules from a predefined list with conditions.
* Logs blocked incoming traffic for monitoring purposes.
* Saves the `iptables` rules to a persistent configuration file.

**📁 Structure**

```text
iptables/
├── defaults/
│   └── main.yml
├── tasks/
│   └── main.yml
```

**⚙️ Defaults (`defaults/main.yml`)**

```yaml
iptables_rules:
  - { description: "Allow HTTP 80/tcp from anywhere", port: 80, proto: tcp }
  - { description: "Allow DNS 53/udp from anywhere", port: 53, proto: udp }
  - { description: "Allow SSH 22/tcp from firewall", port: 22, proto: tcp, source: "10.0.0.254" }
  - { description: "Allow SNMP 162/udp on specific interface", port: 162, proto: udp, interface: "ens18" }
  - { description: "Allow LDAP 389/tcp on specific host", port: 389, proto: tcp, condition: "{{inventory_hostname != 'hostA'}}" }
  - { description: "Established connections", match: conntrack, ctstate: "ESTABLISHED,RELATED" }
```

* `iptables_rules`: A list of firewall rules with their description, port, protocol, source IP, and conditions (if any).

  * **`port`**: Port number for the rule.
  * **`proto`**: Protocol (TCP or UDP).
  * **`source`**: Source IP address for the rule (optional).
  * **`condition`**: A condition to apply the rule (optional).
  * **`match`** and **`ctstate`**: Used for connection tracking states (optional).
  * **`interface`**: The network interface (optional).

**📋 Tasks**

* **Set default policy to ACCEPT**: Sets the default policy for incoming traffic to `ACCEPT`.
* **Flush existing rules**: Clears any existing `iptables` rules in the `INPUT` chain.
* **Add IPtables rules**: Loops through the `iptables_rules` list and applies each rule to the `INPUT` chain.
* **Log blocked traffic**: Logs any blocked traffic on the `INPUT` chain with the prefix `Blocked Input Traffic`.
* **Set default policy to DROP**: Sets the default policy for incoming traffic to `DROP` to block all traffic except the allowed rules.
* **Save iptables rules**: Persists the firewall rules by saving them to `/etc/iptables/rules.v4`.

This role applies firewall rules and logs blocked traffic as per the configuration defined in `defaults/main.yml`.

</details>

### 📄 `journalctl`

<details>
<summary>Click to expand the <code>journalctl</code> role documentation</summary>

Configures `systemd-journald` to control log retention, disk usage, and automatic cleanup on Debian systems.

**✅ Features**

* Configures persistent journald storage limits
* Enforces maximum disk usage and file size for logs
* Automatically cleans up old logs exceeding defined size limits
* Restarts `systemd-journald` to apply changes immediately

**📁 Structure**

```text
journalctl/
├── meta/
│   └── main.yml
└── tasks/
    └── main.yml
```

**⚙️ Configuration**

This role directly manages `/etc/systemd/journald.conf` using the `ini_file` module.

Applied settings:

* `SystemMaxUse`: Maximum disk space used by journald
* `SystemKeepFree`: Minimum free disk space to preserve
* `SystemMaxFileSize`: Maximum size per journal file

```ini
[Journal]
SystemMaxUse=500M
SystemKeepFree=1G
SystemMaxFileSize=100M
```

**📋 Tasks**

* **Configure journald**: Applies disk usage and retention limits to `journald.conf`
* **Restart systemd-journald**: Ensures configuration changes take effect immediately
* **Clean up journald logs**: Removes old logs exceeding 500 MB using `journalctl --vacuum-size`

**🔧 Requirements**

* Debian 12 (tested)
* `systemd` and `systemd-journald`
* `community.general` collection installed (for `ini_file`)

```bash
ansible-galaxy collection install community.general
```

**⚠️ Notes**

* Existing logs may be truncated immediately after applying this role.
* Ensure disk space limits are compatible with system workload and log volume.
* This role does not manage volatile vs persistent storage (`Storage=` option).

</details>

### 📄 `lldap`

<details>
<summary>Click to expand the <code>lldap</code> role documentation</summary>

Deploys and bootstraps [LLDAP](https://github.com/lldap/lldap) in Docker using a configurable directory layout. Supports LDAPS, SMTP for password reset, and user/group bootstrapping via JSON files.

**✅ Features**

* Runs LLDAP in Docker with full environment configuration
* Supports LDAPS via custom certificate files
* Allows SMTP configuration for password reset
* Bootstraps users and groups at first run
* Uses Docker Compose for service management
* Optional override for file source path (e.g., bootstrap or certs)

**📁 Structure**

```text
lldap/
├── defaults/
│   └── main.yml
├── files/
│   ├── bootstrap/
│   │   ├── group-configs/
│   │   │   └── groups.json
│   │   ├── group-schemas/
│   │   │   └── groups.json
│   │   └── user-configs/
│   │       └── users.json
│   └── certs/
│       ├── fullchain.pem
│       └── privkey.pem
├── handlers/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
│   └── docker-compose.yml
```

**⚙️ Defaults (defaults/main.yml)**

```yaml
lldap_directory: /opt/lldap
# lldap_files_source_override: /opt/lldap/files

lldap_global:
  ...

lldap_smtp:
  ...

lldap_ldaps:
  ...
```

* `lldap_directory`: Path where LLDAP will be deployed (config, certs, and bootstrap files).
* `lldap_files_source_override` (*optional*): Custom path to files directory (instead of default `roles/lldap/files`).
* `lldap_global`, `lldap_smtp`, `lldap_ldaps`: Define runtime behavior of LLDAP service.

**📋 Tasks**

* Create the required directories for LLDAP deployment
* Render and copy `docker-compose.yml` from a Jinja2 template
* Copy bootstrap files (users, groups) and certificates into the target directory
* Start the LLDAP container with Docker Compose
* Run the LLDAP bootstrap script (`/app/bootstrap.sh`) inside the container

**🔁 Handlers**

* `Start lldap`: Runs `docker-compose up -d` in the deployment directory
* `Apply bootstrap lldap`: Runs the bootstrap script inside the container

**📦 Docker Compose Template**

* Exposes default ports: 389 (LDAP), 636 (LDAPS), 17170 (Web UI)
* Mounts bootstrap and cert directories
* Configures environment variables for LLDAP behavior (LDAP base DN, SMTP, LDAPS, admin credentials, etc.)

</details>

### 📄 `logindefs`

<details>
<summary>Click to expand the <code>logindefs</code> role documentation</summary>

Manages the `/etc/login.defs` file to enforce system-wide login and password policies.

**✅ Features**

- Deploys a custom `/etc/login.defs` file using a Jinja2 template.
- Ensures correct permissions and ownership.

**📁 Structure**

```text
logindefs/
├── tasks/
│   └── main.yml
├── templates/
│   └── login.defs
```

**📋 Tasks**

- Uses a template (`login.defs`) to overwrite `/etc/login.defs`.
- Ensures the file is owned by `root:root` with read-only permissions (`0444`).

**📄 Template (`templates/login.defs`)**

The `login.defs` template should define system-wide settings for login, password expiration, UID/GID ranges, and other authentication parameters.

</details>

### 📄 `loki`

<details>
<summary>Click to expand the <code>loki</code> role documentation</summary>

Installs and configures [Loki](https://grafana.com/oss/loki/), a horizontally-scalable, highly-available, multi-tenant log aggregation system inspired by Prometheus. This role uses Docker Compose to deploy Loki with a custom configuration.

**✅ Features**

* Creates necessary directory structure for Loki configuration
* Configures Loki with TLS certificates and a custom configuration file
* Deploys Loki container using Docker Compose
* Supports retention policy and compactor configuration
* Supports custom TLS certificates for secure connections

**📁 Directory Structure**

```text
loki/
├── defaults/
│   └── main.yml        # Default variables for Loki configuration
├── files/
│   ├── certs/
│   │   ├── fullchain.pem  # Full certificate chain for TLS
│   │   └── privkey.pem    # Private key for TLS
│   └── local-config.yaml  # Loki configuration file
├── handlers/
│   └── main.yml        # Handlers for restarting Loki service
├── tasks/
│   └── main.yml        # Main tasks for setup and configuration
├── templates/
│   └── docker-compose.yml  # Docker Compose template for Loki setup
```

**⚙️ Default Configuration (`defaults/main.yml`)**

```yaml
loki_directory: /opt/loki  # Directory where Loki configuration and data will be stored
```

* `loki_directory`: Specifies the directory where Loki and its configuration will be installed.

**📋 Tasks**

* Creates necessary directories for Loki and its certificates
* Deploys the `docker-compose.yml` file using Jinja2 templating
* Configures and copies Loki's `local-config.yaml` file to the specified directory
* Copies TLS certificates (`fullchain.pem` and `privkey.pem`) for secure connections
* Starts the Loki service using Docker Compose (`docker-compose up -d`)

**📝 Templates**

* `templates/docker-compose.yml`: Defines the Loki container setup, including ports, volumes, and configuration file paths.
* `files/local-config.yaml`: Configuration file for Loki, including retention policies, compactor configuration, and TLS settings.

**🔧 Requirements**

* Docker and Docker Compose must be installed on the target machine

**📋 Example Loki Configuration (`files/local-config.yaml`)**

```yaml
auth_enabled: false

server:
  http_listen_port: 3100
  http_tls_config:
    cert_file: /etc/loki/certs/fullchain.pem
    key_file: /etc/loki/certs/privkey.pem

limits_config:
  retention_period: 7d

compactor:
  working_directory: /tmp/loki/retention
  retention_enabled: true
  retention_delete_delay: 2h
  delete_request_store: filesystem

common:
  instance_addr: 127.0.0.1
  path_prefix: /loki
  storage:
    filesystem:
      chunks_directory: /loki/chunks
      rules_directory: /loki/rules
  replication_factor: 1
  ring:
    kvstore:
      store: inmemory

schema_config:
  configs:
    - from: 2020-10-24
      store: tsdb
      object_store: filesystem
      schema: v13
      index:
        prefix: index_
        period: 24h

analytics:
  reporting_enabled: false
```

**📋 Tasks Breakdown**

* **Create directories**: Ensures that the necessary directories for configuration and certificates exist.
* **Copy docker-compose.yml**: Deploys the `docker-compose.yml` file for starting the Loki container.
* **Copy loki config**: Copies the custom Loki configuration file (`local-config.yaml`) to the specified directory.
* **Copy loki certs**: Copies the TLS certificates to secure the Loki HTTP server.
* **Start docker**: Launches the Loki container using Docker Compose.

**🔄 Handlers**

* **Restart Loki**: Restarts the Loki container when configuration files or certificates are updated.

```yaml
- name: Restart loki
  ansible.builtin.command: docker-compose restart loki
  args:
    chdir: "{{ loki_directory }}"
  changed_when: false
```

</details>

### 📄 `nginx`

<details>
<summary>Click to expand the <code>nginx</code> role documentation</summary>

Deploys and configures a reverse-proxy using Nginx in Docker with HTTPS support per host. The configuration is fully dynamic and defined through variables.

**✅ Features**

- Supports multiple hosts with different domains and backend services
- Automatically deploys SSL certificates (privkey.pem and fullchain.pem)
- Uses Docker Compose to manage Nginx as a container
- Dynamically generates nginx.conf and docker-compose.yml from templates
- Handler to restart Nginx when configuration changes

**📁 Structure**

```text
nginx/
├── defaults/
│   └── main.yml
├── files/
│   └── hostX/
│       └── domain/
│           ├── fullchain.pem
│           └── privkey.pem
├── handlers/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
│   ├── docker-compose.yml
│   └── nginx.conf
```

**⚙️ Defaults (defaults/main.yml)**

```yaml
nginx_directory: /opt/nginx
#nginx_cert_source_override: /opt/nginx/certs

backend:
  hostA:
    - domain: example.local
      network: example_net
      backend:
        - location: /
          conf: |
            proxy_pass http://backend:3000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
...
```

- `nginx_directory`: Path where the Nginx configuration and Docker setup will be stored.
- `nginx_cert_source_override` (*optional*): If defined, it overrides the default path used to copy certificates.

By default, the role expects certificates under roles/nginx/files/{{ inventory_hostname }}/.
If you want to manage certs outside the role, define this variable with an absolute path.

You can define multiple domains per host, and each domain can contain multiple backend location blocks.

**📋 Tasks**

- Check if certificates exist for the current host
- Create necessary directories for deployment
- Render and copy host-specific docker-compose.yml and nginx.conf
- Copy corresponding TLS certificates from files/
- Start the Nginx service using docker-compose
- Only runs if certificates are present for the host

**🔁 Handlers**

- `Restart nginx docker`: Restarts the Nginx service via docker-compose

**📦 Docker Compose Template**

- Exposes ports 80 and 443
- Mounts host configs and certs
- Supports multiple external networks as defined per backend

**📄 Nginx Config Template**

- Creates one server block per domain
- Sets SSL cert paths per domain
- Creates dynamic location blocks as configured

</details>

### 📄 `ntp`

<details>
<summary>Click to expand the <code>ntp</code> role documentation</summary>

Installs and configures an NTP (Network Time Protocol) server to synchronize the system time with a specified NTP server.

**✅ Features**

- Creates a log directory for the NTP service
- Copies a custom `ntp.conf` configuration file
- Restarts the NTP service after updating the configuration

**📁 Structure**

```text
ntp/
├── defaults/
│   └── main.yml
├── handlers/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
│   └── ntp.conf
```

**⚙️ Defaults (`defaults/main.yml`)**

```yaml
ntp_server: 10.0.0.254
```

- `ntp_server`: The NTP server address to synchronize the system time.

**📋 Tasks**

- Ensures the `/var/log/ntpsec` directory exists with the correct permissions
- Copies the `ntp.conf` file to `/etc/ntpsec/`
- Restarts the NTP service to apply configuration changes

**📝 Templates**

- `templates/ntp.conf`: A custom configuration file for the NTP service.

**🔁 Handlers**

- `Restart service`: Restarts the NTP service (`ntp`) after the configuration is updated.

**🔧 Requirements**

- The `ntp` or `ntpsec` service must be installed on the target machine.

</details>

### 📄 `pfsense`

<details>
<summary>Click to expand the <code>pfsense</code> role documentation</summary>

Configures a **pfSense firewall** using Ansible and the
[`pfsensible.core`](https://galaxy.ansible.com/pfsensible/core) collection.

This role manages core pfSense components such as interfaces, aliases, groups, DHCP, LDAP authentication, NAT, logging, and OpenVPN servers in an **idempotent and declarative way**.

## ✅ Features

* Global pfSense system configuration (hostname, DNS, timezone, GUI theme)
* Interface configuration (WAN, LAN, custom interfaces)
* Alias management (hosts, networks, ports)
* User group management
* LDAP authentication server configuration
* DHCP server configuration
* DHCP static mappings
* Gateway configuration (IPv4 / IPv6)
* NAT port forwarding
* Centralized log settings
* OpenVPN server configuration
* Fully modular tasks with tags

## 📁 Directory Structure

```text
pfsense/
├── defaults/
│   └── main.yml
├── meta/
│   └── main.yml
├── tasks/
│   ├── aliases.yml
│   ├── authserver_ldap.yml
│   ├── dhcp_server.yml
│   ├── dhcp_static.yml
│   ├── gateways.yml
│   ├── groups.yml
│   ├── interfaces.yml
│   ├── log_settings.yml
│   ├── nat_port_forward.yml
│   ├── openvpn_server.yml
│   ├── setup.yml
│   └── main.yml
```

## ⚙️ Default Configuration (`defaults/main.yml`)

This role is entirely driven by variables. Below are the main configuration blocks.

### 🔹 Global Setup

```yaml
pfsense_setup:
  hostname: "pfsense"
  domain: "company.com"
  dns_addresses: 1.1.1.1 9.9.9.9
  dns_gateways: WAN_DHCP WAN_DHCP
  dns_hostnames: one.one.one.one dns.quad9.net
  dnsallowoverride: false
  dnslocalhost: remote
  timezone: Europe/New York
  timeservers: 2.pfsense.pool.ntp.org
  language: en_US
  webguicss: pfSense-dark
```

### 🔹 Interfaces

```yaml
pfsense_interfaces:
  - descr: WAN
    interface: vtnet0
    enable: true
    ipv4_type: dhcp
    blockbogons: true
  - descr: SERVER
    interface: vtnet1
    enable: true
    ipv4_type: static
    ipv4_address: 192.168.101.254
    ipv4_prefixlen: 24
```

### 🔹 Aliases

```yaml
pfsense_aliases:
  - name: "ALIAS_VPN"
    address: "192.168.100.0/24"
    type: "network"
    descr: "VPN Network"
```

Supports `host`, `network`, and `port` aliases.

### 🔹 Groups

```yaml
pfsense_groups:
  - name: "groupA"
    descr: "Group A"
    scope: "remote"
    priv: ["page-all"]
```

### 🔹 LDAP Authentication Server

```yaml
pfsense_authserver_ldap:
  name: LDAP
  host: identity.company.com
  port: 636
  transport: ssl
  ca: company-root-ca
  basedn: dc=company,dc=com
  binddn: uid=pfsense_service,cn=users,dc=company,dc=com
  bindpw: "changeme"
```

⚠️ **Secrets should be stored securely using Ansible Vault.**

### 🔹 DHCP Server

```yaml
pfsense_dhcp_servers:
  - interface: SERVER
    enable: true
    range_from: 192.168.101.10
    range_to: 192.168.101.100
```

### 🔹 DHCP Static Mappings

```yaml
pfsense_dhcp_static_hosts:
  - netif: SERVER
    hostname: hostA
    macaddr: 38:6d:1c:5a:5e:75
    ipaddr: 192.168.101.1
```

### 🔹 Gateways

```yaml
pfsense_gateways:
  - gateway: WAN_DHCP
    ipprotocol: inet
```

### 🔹 NAT Port Forwarding

```yaml
pfsense_nat_port_forwards:
  - descr: "NAT 53/udp"
    interface: WAN
    protocol: udp
    destination: IP:WAN:53
    target: hostA:53
    associated_rule: associated
    state: present
```

### 🔹 OpenVPN Server

```yaml
pfsense_openvpn_servers:
  - name: VPN
    mode: server_tls_user
    authmode: LDAP
    protocol: UDP4
    local_port: 1194
    tunnel_network: 192.168.100.0/24
```

Supports TLS, LDAP authentication, custom push options, DNS configuration, and more.

### 🔹 Log Settings

```yaml
pfsense_log_settings:
  logformat: rfc5424
  nentries: 500
  rotatecount: 10
```

## 🧩 Task Execution & Tags

Each feature can be run independently using tags:

```bash
ansible-playbook site.yml --tags interfaces,dhcp_server
```

Available tags:

* `setup`
* `interfaces`
* `aliases`
* `groups`
* `authserver_ldap`
* `gateways`
* `dhcp_server`
* `dhcp_static`
* `nat_port_forward`
* `openvpn_server`
* `log_settings`

## 🔧 Requirements

* pfSense 2.6+ / 23.x
* Ansible ≥ 2.14
* `pfsensible.core` collection
* API access enabled on pfSense
* Proper credentials / API token

```bash
ansible-galaxy collection install pfsensible.core
```

## ⚠️ Notes & Best Practices

* This role **modifies live firewall configuration**
* Test in a staging environment before production
* Always use **Ansible Vault** for secrets (LDAP bind password, certs)
* Idempotency depends on pfSense API consistency
* Configuration order matters (interfaces → DHCP → VPN)

</details>

### 📄 `privatebin`

<details>
<summary>Click to expand the <code>privatebin</code> role documentation</summary>

Installs and configures [PrivateBin](https://privatebin.info/), a minimalist, open-source online pastebin where the server has zero knowledge of pasted data.

**✅ Features**

- Creates the PrivateBin directory structure
- Deploys configuration files (`conf.php`, `docker-compose.yml`) from templates
- Starts PrivateBin via Docker Compose
- Automatically restarts the container when configuration changes

**📁 Structure**

```text
privatebin/
├── defaults/
│   └── main.yml
├── handlers/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
│   ├── conf.php
│   └── docker-compose.yml
```

**⚙️ Defaults (`defaults/main.yml`)**

```yaml
privatebin_directory: /opt/privatebin

privatebin:
  name: "Company - PrivateBin"
  enable_password: true
  enable_fileupload: false
  burnafterreadingselected: true
  defaultformatter: "plaintext"
  sizelimit: 10485760
  templateselection: false
  languageselection: false
  languagedefault: "en"
  expire_default: "1week"
  traffic_limit: 10
  traffic_exempted: "10.0.0.0/24"
  traffic_creators: "10.0.0.0/24"
```

- `privatebin_directory`: Root path for the PrivateBin installation.
- `privatebin`: Configuration dictionary used in the template `conf.php`.

**📋 Tasks**

- Creates the PrivateBin directory at the specified location
- Deploys `docker-compose.yml` and `conf.php` using Jinja2 templates
- Launches the PrivateBin container using Docker Compose
- Notifies a handler to restart the container if needed

**📝 Templates**

- `templates/conf.php`: PrivateBin main configuration file
- `templates/docker-compose.yml`: Defines the containerized PrivateBin service and its persistent volume

**🔁 Handlers**

- `Restart privatebin`: Restarts the container using Docker Compose after any configuration change

**🔧 Requirements**

- Docker and Docker Compose must be installed on the target machine

</details>

### 📄 `prometheus`

<details>
<summary>Click to expand the <code>prometheus</code> role documentation</summary>

Installs and configures [Prometheus](https://prometheus.io/), a powerful open-source monitoring and alerting toolkit, to collect and store metrics from configured targets. This role sets up Prometheus with Docker Compose and provides a custom configuration for scraping multiple targets.

**✅ Features**

* Creates necessary directories for Prometheus data and configuration
* Deploys Prometheus container using Docker Compose
* Configures Prometheus with a custom `prometheus.yml` file
* Supports integration with SNMP, Blackbox exporter, Node exporter, and cAdvisor
* Configures multiple external networks for Prometheus container communication
* Handles network and volume configurations in Docker Compose

**📁 Directory Structure**

```text
prometheus/
├── defaults/
│   └── main.yml        # Default variables for Prometheus configuration
├── handlers/
│   └── main.yml        # Handlers for restarting Prometheus service
├── tasks/
│   └── main.yml        # Main tasks for setup and configuration
├── templates/
│   ├── docker-compose.yml  # Docker Compose template for Prometheus setup
│   └── prometheus.yml     # Prometheus configuration template
```

**⚙️ Default Configuration (`defaults/main.yml`)**

```yaml
prometheus_directory: /opt/prometheus  # Directory where Prometheus will be installed
prometheus_external_networks:
  - blackbox_default  # External networks Prometheus will use for scraping
  - snmpexporter_default

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'snmp'
    metrics_path: /snmp
    params:
      module:
        - if_mib
    static_configs:
      - targets: ['hostC']
    relabel_configs:
      - source_labels: ['__address__']
        target_label: '__param_target'
      - source_labels: ['__param_target']
        target_label: 'instance'
      - target_label: '__address__'
        replacement: 'snmp-exporter:9116'
  # More scrape configurations (Node Exporter, Blackbox Exporter, cAdvisor) here...
```

* `prometheus_directory`: Specifies the directory where Prometheus and its configuration will be installed.
* `prometheus_external_networks`: List of external networks Prometheus will join for scraping metrics.

**📋 Tasks**

* **Create directories**: Ensures that the necessary directories for Prometheus data and configuration exist.
* **Copy docker-compose.yml**: Deploys the `docker-compose.yml` file for starting the Prometheus container.
* **Template prometheus config**: Templates the `prometheus.yml` configuration file, which contains the scrape targets and other settings.
* **Start docker**: Launches the Prometheus container using Docker Compose (`docker-compose up -d`).

**📝 Templates**

* `templates/docker-compose.yml`: Defines the Prometheus container setup, including ports, volumes, and network configuration.
* `templates/prometheus.yml`: The main Prometheus configuration file, which includes global settings and scrape configurations for various exporters (e.g., Node Exporter, Blackbox Exporter, SNMP).

**🔧 Requirements**

* Docker and Docker Compose must be installed on the target machine

**📋 Example Prometheus Configuration (`templates/prometheus.yml`)**

```yaml
global:
  scrape_interval: 30s  # Scrape interval for collecting metrics

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'snmp'
    metrics_path: /snmp
    params:
      module:
        - if_mib
    static_configs:
      - targets: ['hostC']
    relabel_configs:
      - source_labels: ['__address__']
        target_label: '__param_target'
      - source_labels: ['__param_target']
        target_label: 'instance'
      - target_label: '__address__'
        replacement: 'snmp-exporter:9116'
  # Additional scrape configurations...
```

**📋 Tasks Breakdown**

* **Create directories**: Ensures that necessary directories exist for Prometheus.
* **Copy docker-compose.yml**: Deploys the `docker-compose.yml` file for starting the Prometheus container.
* **Template prometheus config**: Templates the `prometheus.yml` file based on the variables defined in the role.
* **Start docker**: Launches the Prometheus service using Docker Compose, allowing for monitoring and metrics collection.

**🔄 Handlers**

* **Restart Prometheus**: Restarts the Prometheus container when configuration changes.

```yaml
- name: Restart prometheus
  ansible.builtin.command: docker-compose restart prometheus
  args:
    chdir: "{{ prometheus_directory }}"
  changed_when: false
```

</details>

### 📄 `promtail`

<details>
<summary>Click to expand the <code>promtail</code> role documentation</summary>

This role installs and configures **Promtail**, the log collection agent used to send logs to **Loki** for centralized log aggregation and analysis on Debian-based systems. It handles installation, configuration, and management of the Promtail service.

**✅ Features**

* Installs `promtail` and required dependencies
* Configures the Promtail service using a Jinja2 template
* Allows dynamic configuration of log scraping jobs based on the target host
* Ensures the Promtail service is started and enabled to run on boot
* Ensures proper systemd management of the Promtail service

**📁 Structure**

```text
promtail/
├── defaults/
│   └── main.yml
├── handlers/
│   └── main.yml
├── tasks/
│   └── main.yml
└── templates/
    └── config.yml.j2
````

**⚙️ Defaults (defaults/main.yml)**

```yaml
loki_url: https://loki.company.com:3100/
promtail_folder: /etc/promtail

promtail_jobs:
  all:
    - job_name: docker
      labels:
        job: docker
        __path__: /var/lib/docker/containers/*/*.log
  hostA:
    - job_name: squid
      labels:
        job: squid
        __path__: /var/log/squid/access.log
  hostB:
    - job_name: vaultwarden
      labels:
        job: vaultwarden
        __path__: /var/log/vaultwarden/vaultwarden.log
    - job_name: caddy
      labels:
        job: caddy
        __path__: /var/log/caddy/access.log
```

* `loki_url`: The URL of the Loki server where Promtail sends logs.
* `promtail_folder`: Directory for storing Promtail configuration files.
* `promtail_jobs`: Defines the jobs Promtail will scrape logs from, including different log sources based on the host.

**📋 Tasks**

* Installs the `promtail` package
* Configures Promtail's main configuration file using a Jinja2 template
* Generates dynamic Promtail job configurations based on the host
* Ensures the Promtail service is started and enabled to run at boot
* Notifies handlers to restart the Promtail service when necessary

**🔁 Handlers**

```yaml
- name: Restart promtail via systemd
  ansible.builtin.systemd:
    name: promtail.service
    state: restarted
    enabled: true
```

This handler ensures that changes to the configuration are applied immediately by restarting the Promtail service.

**📄 Template: config.yml.j2**

The template defines Promtail's behavior, including log scraping jobs, the Loki server to send logs to, and the file paths for log positions. It dynamically adapts to different hosts and log sources.

</details>

### 📄 `proxmox`

<details>
<summary>Click to expand the <code>proxmox</code> role documentation</summary>

Automate the creation and deployment of Debian-based virtual machines on a Proxmox server, including building custom preseeded ISO images.

---

**✅ Features**

* Builds a custom Debian ISO using preseed configuration
* Uploads ISO to specified Proxmox node(s)
* Creates and configures virtual machines via Proxmox API
* Allows detailed VM customization: memory, CPU, disk, network, tags, etc.
* Fully local ISO generation via [custom\_iso project](https://github.com/6C656C65/custom_iso)

---

**📁 Structure**

```text
proxmox/
├── defaults/
│   └── main.yml
├── tasks/
│   ├── main.yml
│   ├── iso.yml
│   └── vm.yml
```

---

**⚙️ Defaults (`defaults/main.yml`)**

```yaml
custom_iso_path: "{{ playbook_dir }}/custom_iso"

build:
  in: https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-12.10.0-amd64-netinst.iso
  out: debian-12.10.0-amd64-netinst-preseed-ansible.iso
  preseed: ../preseed.cfg

deploy:
  proxmox_host: proxmox.company.com
  proxmox_nodes: proxmox-001
  token_id: root@pam!ansible

proxmox:
  api_host: proxmox.company.com:8006
  api_user: root@pam
  api_pass: changeme
  node: proxmox-001

vm_list:
  - name: "vm-100"
    ...
  - name: "vm-200"
    ...
```

* `custom_iso_path`: Path to the `custom_iso` project directory.
* `build.in`: URL or path of base Debian ISO.
* `build.out`: Output ISO filename.
* `build.preseed`: Path to the preseed file used for customization.
* `deploy`: Proxmox access details for ISO upload.
* `proxmox`: API credentials and target node.
* `vm_list`: List of VMs to create with full specs (RAM, CPU, network, etc.).

---

**📋 Tasks**

* **`iso.yml`**:

  * Ensures necessary scripts are executable
  * Builds a custom preseed ISO locally
  * Renames ISO with timestamp
  * Uploads ISO to Proxmox node(s)
  * Runs a cleanup script

* **`vm.yml`**:

  * Creates virtual machines on the Proxmox node using the uploaded ISO
  * Each VM is configured based on `vm_list`

* **`main.yml`**:

  * Includes ISO build/upload and VM creation tasks
  * Allows tagged execution (`--tags iso` or `--tags vm`)

---

**🏷️ Tags**

Run specific tasks using tags:

```bash
ansible-playbook your_playbook.yml --tags iso     # Only build and upload ISO
ansible-playbook your_playbook.yml --tags vm      # Only create VMs
ansible-playbook your_playbook.yml --tags proxmox # Run all proxmox role tasks
```

---

**📦 Prerequisites**

* `custom_iso` project ([https://github.com/6C656C65/custom\_iso](https://github.com/6C656C65/custom_iso)) cloned at the specified path
* A `preseed.cfg` file located appropriately
* Proxmox server with API access
* Ansible collections:

  * `community.general` (for `proxmox_kvm` module)

</details>

### 📄 `proxy`

<details>
<summary>Click to expand the <code>proxy</code> role documentation</summary>

Configure system-wide and user-wide HTTP/HTTPS proxy settings for APT, environment variables, and shell configuration.

**✅ Features**

- Defines and applies HTTP/HTTPS proxy settings
- Cleans old or duplicate proxy entries
- Configures APT proxy in `/etc/apt/apt.conf.d/01proxy`
- Sets both global (`/etc/environment`) and user shell (`bashrc`) proxy variables

**📁 Structure**

```text
proxy/
├── defaults/
│   └── main.yml
├── tasks/
│   └── main.yml
```

**⚙️ Defaults (`defaults/main.yml`)**

```yaml
http_proxy: "http://proxy.company.com:3128"
https_proxy: "https://proxy.company.com:3128"
no_proxy: "localhost,127.0.0.1,10.0.0.0/16,192.168.0.0/16,172.16.0.0/12"
```

- `http_proxy`: HTTP proxy to use.
- `https_proxy`: HTTPS proxy to use.
- `no_proxy`: Comma-separated list of domains/IPs to exclude from proxying.

**📋 Tasks**

- Removes old proxy settings from `bashrc`, `/etc/environment`, and APT config
- Adds updated proxy configuration for APT
- Updates global `/etc/environment` with proxy variables
- Adds export lines in `/etc/bash.bashrc` for interactive shells

</details>

### 📄 `pythonrunner`

<details>
<summary>Click to expand the <code>pythonrunner</code> role documentation</summary>

Installs and manages **PythonRunner**, a Python-based automation service, deployed as a systemd service on Debian systems.
This role installs the Python package, deploys extensions from Git, configures the service, and ensures it runs persistently.

**✅ Features**

* Installs PythonRunner via `pip`
* Deploys extensions from a Git repository
* Manages a YAML configuration file via Jinja2 template
* Creates and manages a dedicated systemd service
* Automatically reloads and restarts the service on configuration changes
* Runs the service under a non-root user

**📁 Structure**

```text
pythonrunner/
├── defaults/
│   └── main.yml
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
│   └── config.yml
```

**⚙️ Defaults (`defaults/main.yml`)**

```yaml
pythonrunner:
  path: "/home/ansible/pythonrunner"
  package_name: "pyrunx"
  extensions_url: "https://github.com/6C656C65/pythonrunner-extensions.git"
  service_name: "pythonrunner"
  config_template: config.yml
```

* `path`: Working directory for PythonRunner and its extensions
* `package_name`: Python package installed via `pip`
* `extensions_url`: Git repository containing PythonRunner extensions
* `service_name`: Name of the systemd service
* `config_template`: Configuration template file name

**📋 Tasks**

* **Ensure pip is installed**: Installs `python3-pip`
* **Install the Python package**: Installs PythonRunner using `pip`
* **Clone extensions repository**: Deploys extensions into the working directory
* **Deploy configuration file**: Renders and copies `config.yml`, triggers service restart
* **Create systemd service**: Installs a custom service unit file
* **Enable and start service**: Ensures PythonRunner starts on boot

**🔁 Handlers**

* **Reload systemd**: Reloads systemd daemon after service file changes
* **Restart pythonrunner service**: Restarts the PythonRunner service

```yaml
- name: Restart pythonrunner service
  ansible.builtin.systemd:
    name: "{{ pythonrunner.service_name }}"
    state: restarted
```

**📄 Systemd Service Behavior**

* Runs as user: `ansible`
* Working directory: `{{ pythonrunner.path }}`
* Automatically restarts on failure
* Logs stdout and stderr to `journald`

```ini
ExecStart=/usr/local/bin/pythonrunner -e extensions/rootme
Restart=always
```

**📄 Configuration Template (`config.yml`)**

* Supports multiple modules (e.g. `crtsh`, `dns`, `rootme`, `ssl_monitor`)
* Handles secrets such as API keys and Discord tokens
* Fully managed by Ansible (template-driven)

**⚠️ Notes**

* Secrets should be stored securely (Ansible Vault recommended)
* `pip` is used with `break_system_packages: true`
* The service execution path and extensions are currently hardcoded in the systemd unit
* This role assumes PythonRunner exposes no network ports directly

</details>

### 📄 `rsyslog`

<details>
<summary>Click to expand the <code>rsyslog</code> role documentation</summary>

Installs and configures [rsyslog](https://www.rsyslog.com/), a reliable and highly configurable system logging daemon, using Docker Compose to manage the service.

**✅ Features**

* Creates the necessary directory structure for rsyslog configuration
* Deploys a templated `docker-compose.yml` for Docker container setup
* Configures rsyslog to listen on specific UDP ports and route logs to designated files
* Supports easy modification of log file paths and ports
* Allows customization of rsyslog configuration via a Jinja2 template

**📁 Directory Structure**

```text
rsyslog/
├── defaults/
│   └── main.yml        # Default variables for rsyslog configuration
├── handlers/
│   └── main.yml        # Handlers for restarting the rsyslog service
├── tasks/
│   └── main.yml        # Main tasks for setup and configuration
├── templates/
│   ├── docker-compose.yml  # Docker Compose template for rsyslog setup
│   └── rsyslog.conf       # rsyslog configuration template
```

**⚙️ Default Configuration (`defaults/main.yml`)**

```yaml
rsyslog_directory: /opt/rsyslog  # Directory where rsyslog configuration is stored

rsyslog_udp_ports:
  - port: 514
    log_file: "/var/log/app1.log"
  - port: 515
    log_file: "/var/log/app2.log"
```

* `rsyslog_directory`: Specifies the root path where rsyslog will be installed.
* `rsyslog_udp_ports`: List of UDP ports to be used by rsyslog, along with the log file path for each port.

**📋 Tasks**

* Creates necessary directories for rsyslog configuration and logs
* Deploys the `docker-compose.yml` file using Jinja2 templating
* Templates the `rsyslog.conf` configuration file and notifies rsyslog restart
* Starts the rsyslog service using Docker Compose (`docker-compose up -d`)

**📝 Templates**

* `templates/docker-compose.yml`: Defines the rsyslog container, ports, volumes, and mounts.
* `templates/rsyslog.conf`: Defines the rsyslog configuration, including the UDP input and file outputs for each port.

**🔧 Requirements**

* Docker and Docker Compose must be installed on the target machine

**📋 Handlers**

* **Restart rsyslog**: Ensures that rsyslog restarts when the configuration is updated.

```yaml
- name: Restart rsyslog
  ansible.builtin.command: docker-compose restart rsyslog
  args:
    chdir: "{{ rsyslog_directory }}"
  changed_when: false
```

</details>

### 📄 `semaphore`

<details>
<summary>Click to expand the <code>semaphore</code> role documentation</summary>

Installs and configures [Semaphore](https://github.com/ansible-semaphore/semaphore), a modern open-source Ansible UI and dashboard, using Docker Compose.

**✅ Features**

- Creates the directory structure for Semaphore
- Deploys a templated `docker-compose.yml` with configuration variables
- Starts Semaphore using Docker Compose
- Supports proxy configuration and email notifications
- Uses the embedded BoltDB database for simplicity

**📁 Structure**

```text
semaphore/
├── defaults/
│   └── main.yml
├── handlers/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
│   └── docker-compose.yml
```

**⚙️ Defaults (`defaults/main.yml`)**

```yaml
semaphore_directory: /opt/semaphore

semaphore:
  admin: admin
  admin_password: changeme
  admin_name: Admin
  admin_email: admin@company.com
  email_sender: semaphore@company.com
  email_host: smtp.company.com
  email_port: 465
  email_username: semaphore@company.com
  email_password: changeme
  email_secure: "True"
  web_root: "https://semaphore.company.com"

http_proxy: "http://proxy.company:3128"
https_proxy: "https://proxy.company:3128"
no_proxy: "localhost,127.0.0.1,10.0.0.0/16,192.168.0.0/16,172.16.0.0/12"
```

- `semaphore_directory`: Root path for the Semaphore installation.
- `semaphore`: Dictionary of admin credentials and email configuration for Semaphore.
- `http_proxy`, `https_proxy`, `no_proxy`: Optional proxy settings for container environment.

**📋 Tasks**

- Creates the base directory for Semaphore with appropriate permissions
- Deploys the `docker-compose.yml` file using Jinja2 templating
- Launches Semaphore using `docker-compose up -d`

**📝 Templates**

- `templates/docker-compose.yml`: Defines the Semaphore container, volumes, environment variables, and proxy settings

**🔧 Requirements**

- Docker and Docker Compose must be installed on the target machine

</details>

### 📄 `snmpexporter`

<details>
<summary>Click to expand the <code>snmpexporter</code> role documentation</summary>

Installs and configures the [SNMP Exporter](https://github.com/prometheus/snmp_exporter), a Prometheus exporter for collecting SNMP metrics, using Docker Compose to manage the service.

**✅ Features**

* Creates the necessary directory structure for SNMP Exporter configuration
* Deploys a templated `docker-compose.yml` for Docker container setup
* Configures SNMP Exporter using the official Prometheus SNMP Exporter image
* Supports time zone and local time configuration using Docker volumes

**📁 Directory Structure**

```text
snmpexporter/
├── defaults/
│   └── main.yml        # Default variables for SNMP Exporter configuration
├── tasks/
│   └── main.yml        # Main tasks for setup and configuration
├── templates/
│   └── docker-compose.yml  # Docker Compose template for SNMP Exporter setup
```

**⚙️ Default Configuration (`defaults/main.yml`)**

```yaml
snmpexporter_directory: /opt/snmpexporter  # Directory where SNMP Exporter configuration is stored
```

* `snmpexporter_directory`: Specifies the root path where SNMP Exporter will be installed.

**📋 Tasks**

* Creates necessary directories for SNMP Exporter configuration
* Deploys the `docker-compose.yml` file using Jinja2 templating
* Starts the SNMP Exporter service using Docker Compose (`docker-compose up -d`)

**📝 Templates**

* `templates/docker-compose.yml`: Defines the SNMP Exporter container setup, volumes, and configuration for time zone and local time.

**🔧 Requirements**

* Docker and Docker Compose must be installed on the target machine

**📋 Tasks**

* **Create directories**: Creates the directory structure for SNMP Exporter.
* **Copy docker-compose.yml**: Deploys the `docker-compose.yml` file using a template.
* **Start docker**: Launches the SNMP Exporter container with `docker-compose up -d`.

```yaml
- name: Start docker
  ansible.builtin.command: docker-compose up -d
  args:
    chdir: "{{ snmpexporter_directory }}"
  changed_when: false
```

</details>

### 📄 `squid`

<details>
<summary>Click to expand the <code>squid</code> role documentation</summary>

Installs and configures [Squid](http://www.squid-cache.org/), a caching proxy for the web, supporting HTTP, HTTPS, and FTP, using Docker Compose.

**✅ Features**

- Creates the directory structure for Squid
- Deploys a templated `docker-compose.yml` with configuration variables
- Supports optional log volume mapping
- Copies Squid configuration and IP/domain whitelist files
- Starts Squid using Docker Compose
- Reloads Squid when configuration is updated

**📁 Structure**

```text
squid/
├── defaults/
│   └── main.yml
├── files/
│   ├── lan.txt
│   └── squid.conf
├── handlers/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
│   └── docker-compose.yml
```

**⚙️ Defaults (`defaults/main.yml`)**

```yaml
squid_directory: /opt/squid
# squid_conf_source_override: /opt/squid/conf

squid:
  port: 3128:3128
  timezone: "UTC"
#  log_file_volume: /var/log/squid/access.log:/var/log/squid/access.log
```

- `squid_directory`: Root path for the Squid deployment.
- `squid.port`: Port mapping for the proxy container.
- `squid.timezone`: Timezone set in the container.
- `squid.log_file_volume`: (Optional) Path to bind mount the access log file.
- `squid_conf_source_override`: (Optional) Path override for Squid config sources.

**📋 Tasks**

- Creates the base and log directories if needed
- Touches the access log file with correct permissions (if defined)
- Deploys the `docker-compose.yml` using Jinja2 templating
- Copies `squid.conf` and `lan.txt` configuration files
- Starts Squid with `docker-compose up -d`

**📝 Templates**

- `templates/docker-compose.yml`: Defines the Squid container, volumes, environment variables, and network

**🔁 Handlers**

- `Reload squid configuration`: Runs `squid -k reconfigure` inside the container to reload the config

**🔧 Requirements**

- Docker and Docker Compose must be installed on the target machine

</details>

### 📄 `sshd`

<details>
<summary>Click to expand the <code>sshd</code> role documentation</summary>

Configure and customize the OpenSSH server (`sshd`) and enhance the login experience with an ASCII banner and system uptime.

**✅ Features**

- Custom dynamic ASCII MOTD using `art.txt`
- Overwrites default `sshd_config`
- Deletes static `/etc/motd`
- Only keeps the configured MOTD script
- Reloads SSH service automatically on changes

**📁 Structure**

```text
sshd/
├── defaults/
│   └── main.yml
├── files/
│   └── art.txt
├── handlers/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
│   ├── ascii_motd
│   └── sshd_config
```

**⚙️ Defaults (`defaults/main.yml`)**

```yaml
motd_ascii_file: "99-ascii"
```

- `motd_ascii_file`: Name of the MOTD script to generate.

**📂 Files**

- `files/art.txt`: ASCII art displayed on SSH login.

**📝 Templates**

- `templates/ascii_motd`: Bash script displayed on login with color + uptime.
- `templates/sshd_config`: SSH daemon configuration.

**📋 Tasks**

- Loads ASCII art from file
- Deploys custom MOTD and `sshd_config`
- Removes legacy MOTD files
- Notifies handler to restart SSHD if needed

**🔁 Handlers**

- `Restart sshd daemon`: Ensures SSH service is restarted after configuration updates.

**🔧 Requirements**

- OpenSSH server must be installed (`sshd`)

</details>

### 📄 `sshkeys`

<details>
<summary>Click to expand the <code>sshkeys</code> role documentation</summary>

Deploys and enforces SSH public keys in `authorized_keys` for a dedicated user on Debian systems.

**✅ Features**

* Deploys SSH public keys from `.pub` files
* Supports an override directory for key sources
* Enforces exclusive access (removes unmanaged keys)
* Centralizes SSH key management via Ansible
* Simple and deterministic key deployment

**📁 Structure**

```text
sshkeys/
├── defaults/
│   └── main.yml
├── files/
│   └── example.pub
├── meta/
│   └── main.yml
└── tasks/
    └── main.yml
```

**⚙️ Defaults (`defaults/main.yml`)**

```yaml
# sshkeys_folder_override: /opt/sshkeys/keys
```

* `sshkeys_folder_override` (*optional*): Absolute path to a directory containing SSH public key files (`*.pub`).

  * If not defined, keys are read from the role’s `files/` directory.

**📋 Tasks**

* **Read SSH public keys**:

  * Collects all `.pub` files from the configured key directory
  * Aggregates their content into a single list

* **Enforce SSH keys for ansible user**:

  * Deploys the collected keys to the `authorized_keys` file of the `ansible` user
  * Uses `exclusive: true` to remove any unmanaged keys
  * Ensures a fully declarative SSH access model

**🔐 Security Behavior**

* Only keys provided via Ansible will remain authorized
* Any manual modification to `authorized_keys` will be reverted
* If no keys are found, no changes are applied

**🔧 Requirements**

* `ansible` user must exist on the target system
* `ansible.posix` collection installed

```bash
ansible-galaxy collection install ansible.posix
```

**⚠️ Notes**

* Using `exclusive: true` may lock you out if no valid keys are provided.
* Ensure SSH access is validated before applying on remote systems.

</details>

### 📄 `sysctl`

<details>
<summary>Click to expand the <code>sysctl</code> role documentation</summary>

Harden kernel networking and memory behavior using persistent sysctl rules for IPv4, IPv6, and core dump settings.

**✅ Features**

- Disables IPv6 and ICMP redirects
- Hardens IPv4 settings (source routing, martian logging, filters)
- Disables core dumps for better security
- Automatically reloads sysctl rules after changes

**📁 Structure**

```text
sysctl/
├── defaults/
│   └── main.yml
├── handlers/
│   └── main.yml
├── tasks/
│   └── main.yml
```

**⚙️ Defaults (`defaults/main.yml`)**

```yaml
sysctl_ipv6:
  - { name: net.ipv6.conf.all.accept_ra, value: "0" }
  ...
sysctl_ipv4:
  - { name: net.ipv4.conf.all.send_redirects, value: "0" }
  ...
sysctl_core_dump:
  - { name: fs.suid_dumpable, value: "0" }
```

- Define system-level kernel parameters for IPv6, IPv4, and core dumps.

**📋 Tasks**

- Applies all kernel parameters with `ansible.posix.sysctl`
- Uses structured defaults for readability and flexibility
- Notifies handler to reload all sysctl settings via `sysctl --system`

**🔁 Handlers**

- `Reload the sysctl configuration`: Ensures new sysctl values are applied system-wide.

**🔧 Requirements**

- `ansible.posix` collection must be installed
- System should support `sysctl --system` (common on systemd-based distributions)

</details>

### 📄 `trust_ca`

<details>
<summary>Click to expand the `trust_ca` role documentation</summary>

Manages the addition of custom Certificate Authorities (CAs) to the system's trusted CA store.

**✅ Features**

- Copies custom `.crt` certificates to `/usr/local/share/ca-certificates/`.
- Notifies the system to update CA certificates.
- Ensures correct file permissions and ownership for the certificates.

**📁 Structure**

```text
trust_ca/
├── files/
│   └── .gitkeep
├── handlers/
│   └── main.yml
├── tasks/
│   └── main.yml
```

**⚙️ Files (`files/)**

The `files/` directory contains the custom `.crt` files that will be added to the trusted CA store. 

**📋 Tasks**

- **Copy certificates**: Copies all `.crt` files from `files/` to `/usr/local/share/ca-certificates/`.
- **Notify CA update**: Triggers the update of the CA certificates.

**🔁 Handlers**

- **Updating CA certificates**: Runs the `update-ca-certificates` command to update the system's CA certificates.

</details>

### 📄 `uptime_kuma`

<details>
<summary>Click to expand the <code>uptime_kuma</code> role documentation</summary>

Deploys and manages [Uptime Kuma](https://github.com/louislam/uptime-kuma), a self-hosted monitoring tool, using Docker Compose on Debian systems.

**✅ Features**

* Deploys Uptime Kuma using Docker Compose
* Creates and manages a dedicated installation directory
* Supports HTTP/HTTPS proxy configuration via environment variables
* Uses a persistent Docker volume for monitoring data
* Simple restart handler for configuration changes

**📁 Structure**

```text
uptime_kuma/
├── defaults/
│   └── main.yml
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
│   └── docker-compose.yml
```

**⚙️ Defaults (`defaults/main.yml`)**

```yaml
uptime_kuma_directory: /opt/uptime_kuma
```

* `uptime_kuma_directory`: Base directory on the target system where Uptime Kuma is deployed.

**📋 Tasks**

* **Create Uptime Kuma base directory**: Ensures the installation directory exists with proper permissions.
* **Deploy docker-compose configuration**: Renders and copies the `docker-compose.yml` file using a Jinja2 template.
* **Start Uptime Kuma using Docker Compose**: Launches the Uptime Kuma container in detached mode.

**🔁 Handlers**

* **Restart Uptime Kuma**: Restarts the Uptime Kuma service using Docker Compose when configuration changes.

```yaml
- name: Restart Uptime Kuma
  ansible.builtin.command: docker-compose restart
  args:
    chdir: "{{ uptime_kuma_directory }}"
  changed_when: false
```

**📦 Docker Compose**

* Uses the `louislam/uptime-kuma` official Docker image
* Exposes the default Uptime Kuma web interface port (3001) via Docker networking
* Persists application data using a named Docker volume (`uptime-kuma-data`)
* Supports proxy configuration using:

  * `http_proxy`
  * `https_proxy`
  * `no_proxy`

</details>

### 📄 `users`

<details>
<summary>Click to expand the <code>users</code> role documentation</summary>

This role is used to configure the Ansible and root user accounts on Debian-based systems. It generates secure, random passwords for the users and updates their credentials accordingly.

**✅ Features**

- Changes the password for the `ansible` user with a secure password hash.
- Generates a random password for the `root` user with configurable length.
- Displays the newly generated root password for reference.

**📁 Structure**

```text
users/
├── defaults/
│   └── main.yml
├── tasks/
│   └── main.yml
````

**⚙️ Defaults (`defaults/main.yml`)**

```yaml
before_setup_ansible_ssh_pass: "changeme"
before_setup_ansible_become_pass: "changeme"
password_length: 15
```

* `before_setup_ansible_ssh_pass`: The initial SSH password for the `ansible` user (before setting the actual secure password).
* `before_setup_ansible_become_pass`: The initial `become` password for `ansible` user (before setting the actual secure password).
* `password_length`: Defines the length of the generated random password for the `root` user (default is 15 characters).

**📋 Tasks**

1. **Change the Ansible user password**
   Updates the `ansible` user's password to a secure hash stored in the `vault_ansible_password` variable.

2. **Generate a random password**
   Creates a secure random password for the `root` user, with a length defined by `password_length`.

3. **Set the generated password in a variable**
   Stores the generated password for the `root` user in the `my_pass` variable.

4. **Change the root user password**
   Updates the `root` user's password to the newly generated password, securely hashed.

5. **Display the generated root password**
   Outputs the newly generated `root` password to the debug output for reference.

</details>

### 📄 `vaultwarden`

<details>
<summary>Click to expand the <code>vaultwarden</code> role documentation</summary>

This role sets up Vaultwarden using Docker. It includes configuration for Vaultwarden, SMTP settings for email notifications, and backup configurations.

**✅ Features**

- Deploys Vaultwarden using Docker Compose
- Configures SMTP settings for email notifications
- Creates directories and files for Vaultwarden logs and data
- Supports automatic backup configuration with optional email notifications
- Provides custom CA certificate support for secure communication
- Configures the Vaultwarden domain and admin token
- Allows backup retention and email notifications on backup success or failure

**📁 Structure**

```text
vaultwarden/
├── defaults/
│   └── main.yml
├── handlers/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
│   └── docker-compose.yml
```

**⚙️ Defaults (`defaults/main.yml`)**

```yaml
vaultwarden_directory: /opt/vaultwarden
vaultwarden_log_file: /var/log/vaultwarden/vaultwarden.log
vaultwarden_ca_volume: /etc/ssl/certs/example.pem:/etc/ssl/certs/example.pem

vaultwarden_domain: https://vault.local

#vaultwarden_admin_token:
#vaultwarden_yubico:
#  id:
#  key:

vaultwarden_smtp:
  host: smtp.example.com
  from: vaultwarden@company.com
  port: 465
  security: force_tls
  username: vaultwarden@company.com
  password: changeme

vaultwarden_backup:
  keep_days: 2
  smtp_enable: "true"
  mail_smtp_variables: "
    -S smtp=smtps://smtp.example.com:465 \
    -S smtp-auth=login \
    -S smtp-auth-user=vaultwarden@company.com \
    -S smtp-auth-password=changeme \
    -S from=vaultwarden@company.com"
  mail_to: admin@company.com
  mail_when_success: "false"
  mail_when_failure: "true"
```

- Define the directory and log file paths for Vaultwarden.
- Configure SMTP settings for sending email notifications.
- Set up backup retention period and email notification preferences.

**📋 Tasks**

- Creates the necessary Vaultwarden base directory
- Deploys the Docker Compose configuration for Vaultwarden
- Creates log directory and log file, ensuring proper ownership and permissions
- Starts Vaultwarden using Docker Compose
- Backs up Vaultwarden data and notifies if configured

**🔁 Handlers**

- `Restart vaultwarden`: Restarts the Vaultwarden service by restarting the Docker container using Docker Compose.

**🔧 Requirements**

- Docker and Docker Compose should be installed
- The system should support Docker networking for proper operation
- Ensure that your SMTP server is correctly configured if email notifications are enabled
- Backup functionality relies on a pre-configured backup service, which may require specific container images (e.g., `ttionya/vaultwarden-backup`)

</details>

### 📄 `version`

<details>
<summary>Click to expand the <code>version</code> role documentation</summary>

This role retrieves the latest Git commit information and writes it to a specified file on the target machine. It is useful for tracking the current version of the application or system by storing the commit ID and the corresponding date.

**✅ Features**

* Fetches the latest Git commit hash and the commit date using `git log -1`.
* Writes the commit ID and date to a specified file on the target machine.
* Helps to keep track of the current version of the system or application.

**📁 Structure**

```text
version/
├── defaults/
│   └── main.yml
├── tasks/
│   └── main.yml
```

**⚙️ Defaults (`defaults/main.yml`)**

```yaml
version_path: /etc/version.txt
```

* `version_path`: The file path where the commit details will be stored (default: `/etc/version.txt`).

**📋 Tasks**

* **Retrieve the latest commit**: Uses the `git log` command to retrieve the latest commit hash and date.
* **Write to the machine**: Writes the commit ID and date into the specified file (`version_path`).

</details>

---

Happy automating 🛠️