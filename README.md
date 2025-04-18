## 🚀 Ansible Roles

**Ansible roles** is a project containing several Ansible roles that allow you to configure a Debian server easily and efficiently.

> [!WARNING]
> Roles have only been tested with `debian-12.10.0`

---

## 📑 Table of Contents

1. [Integration](#-integration)
2. [Available Roles](#-available-roles)
   - [dns](#-dns)
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
<summary>Click to expand the `dns` role documentation</summary>

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

### 📄 `ntp`

<details>
<summary>Click to expand the `ntp` role documentation</summary>

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
<summary>Click to expand the `proxy` role documentation</summary>

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
<summary>Click to expand the `sshd` role documentation</summary>

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
<summary>Click to expand the `sysctl` role documentation</summary>

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