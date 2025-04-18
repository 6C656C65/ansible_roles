## 🚀 Ansible Roles

**Ansible roles** is a project containing several Ansible roles that allow you to configure a Debian server easily and efficiently.

> [!WARNING]
> Roles have only been tested with `debian-12.10.0`

---

## 📑 Table of Contents

1. [Integration](#-integration)
2. [Available Roles](#-available-roles)
   - [dns](#-dns)
   - [docker](#-docker)
   - [grub](#-grub)
   - [logindefs](#-logindefs)
   - [ntp](#-ntp)
   - [proxy](#-proxy)
   - [sshd](#-sshd)
   - [sysctl](#-sysctl)

---

## 📦 Integration

You can integrate this repository into a larger Ansible project with:

```bash
git submodule add https://github.com/6C656C65/ansible_roles roles
```

---

## 🔧 Available Roles

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

---

Happy automating 🛠️