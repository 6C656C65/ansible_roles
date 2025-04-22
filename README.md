## 🚀 Ansible Roles

**Ansible roles** is a project containing several Ansible roles that allow you to configure a Debian server easily and efficiently.

> [!WARNING]
> Roles have only been tested with `debian-12.10.0`

---

## 📑 Table of Contents

1. [Integration](#-integration)
2. [Available Roles](#-available-roles)
   - [apt](#-apt)
   - [cadvisor](#-cadvisor)
   - [dns](#-dns)
   - [docker](#-docker)
   - [fail2ban](#-fail2ban)
   - [grub](#-grub)
   - [logindefs](#-logindefs)
   - [nginx](#-nginx)
   - [ntp](#-ntp)
   - [privatebin](#-privatebin)
   - [proxy](#-proxy)
   - [semaphore](#-semaphore)
   - [squid](#-squid)
   - [sshd](#-sshd)
   - [sysctl](#-sysctl)
   - [trust_ca](#-trust_ca)
   - [vaultwarden](#-vaultwarden)

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

### 📄 nginx

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

---

Happy automating 🛠️