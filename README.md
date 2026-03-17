# AutoRent — Infrastructure as Code

> A complete local development environment for a car rental web application, provisioned with **Vagrant + Ansible + Nginx**. A single command brings up the entire infrastructure, identical for every developer.

![Vagrant](https://img.shields.io/badge/Vagrant-1868F2?style=for-the-badge&logo=vagrant&logoColor=white)
![Ansible](https://img.shields.io/badge/Ansible-EE0000?style=for-the-badge&logo=ansible&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu_22.04-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![VirtualBox](https://img.shields.io/badge/VirtualBox-183A61?style=for-the-badge&logo=virtualbox&logoColor=white)
![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)

> 🚀 **Performance highlight:** ~33,000 req/s with 0% error rate on a local VM — measured with Apache Benchmark after OS and Nginx tuning.

---

## Table of Contents

- [Overview](#overview)
- [What I Practiced](#what-i-practiced)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Accessing the Application](#accessing-the-application)
- [Infrastructure Details](#infrastructure-details)
- [Performance](#performance)
- [The Application](#the-application)

---

## Overview

The goal of this project is not simply to host a website — it's to create a **repeatable, resilient, and high-performance environment** where any developer runs a single command and gets the exact same production-like setup locally.

The infamous *"it works on my machine"* problem is eliminated by design: the VM is defined as code and the server state is always enforced by Ansible.

```
vagrant up
# → VM created, Ubuntu provisioned, Nginx configured, site live.
```

<img width="783" height="516" alt="iac" src="https://github.com/user-attachments/assets/9452354f-3cce-4e6b-8fd8-8750b9ccc7bd" />

---

## What I Practiced

- **Infrastructure as Code (IaC)** — entire environment defined and version-controlled as code
- **Idempotency** — Ansible playbook can run multiple times with no side effects
- **Nginx Server Blocks** — multi-tenant web server configuration
- **OS-level performance tuning** — `worker_connections` and file descriptor limits under load
- **Developer Experience (DevEx)** — zero-friction access via nip.io, no `/etc/hosts` editing required
- **Async API integration** — Fetch API with JSON payload and mock backend response handling

---

## Tech Stack

| Layer | Tool | Role |
|---|---|---|
| Provisioning | Vagrant | Isolates and defines the VM as code |
| Configuration | Ansible | Enforces declarative server state |
| Web Server | Nginx | Server Blocks, routing and performance tuning |
| Operating System | Ubuntu 22.04 LTS | Stable, long-term supported base |
| Dev DNS | nip.io | Resolves domain without editing `/etc/hosts` |

---

## Project Structure

```
AutoRent/
├── html/
│   ├── img/
│   │   ├── car1.jpg
│   │   ├── car2.jpg
│   │   └── car3.jpg
│   └── index.html
├── nginx/
│   └── autorent.local.conf
├── playbook.yml
└── Vagrantfile
```

---

## Prerequisites

- [Vagrant](https://www.vagrantup.com/downloads) `>= 2.3`
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads) `>= 6.1`
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/) `>= 2.12` (on the host machine)

---

## Getting Started

**1. Clone the repository**

```bash
git clone https://github.com/luizcosta2191/autorent-iac.git
cd autorent-iac
```

**2. Bring up the environment**

```bash
vagrant up
```

This single command will:
- Create the VM with Ubuntu 22.04
- Automatically run the Ansible playbook
- Install and configure Nginx
- Deploy the site files to the correct directory
- Remove the Nginx default site

**3. (Optional) Re-provision after changes**

```bash
vagrant provision
```

**4. Tear down the environment**

```bash
vagrant destroy
```

---

## Accessing the Application

Once `vagrant up` completes, the application is available at two addresses:

| Method | URL | Requirement |
|---|---|---|
| Via nip.io (recommended) | `http://autorent.192.168.56.10.nip.io` | None — automatic DNS resolution |
| Via local domain | `http://autorent.local` | Add an entry to `/etc/hosts` |

> **Why nip.io?** It's a *Developer Experience* decision: no one needs to edit system files. The public DNS service automatically resolves `autorent.192.168.56.10.nip.io` to the VM's IP `192.168.56.10`.

To use the local domain instead, add the following to your `/etc/hosts`:

```
192.168.56.10   autorent.local
```

---

## Infrastructure Details

### Vagrantfile

- Box: `ubuntu/jammy64` (Ubuntu 22.04 LTS)
- Fixed private network IP: `192.168.56.10`
- Provisioned via Ansible with `become: true`

### Ansible Playbook (`playbook.yml`)

The playbook is **idempotent** — it can be run multiple times with no side effects. It ensures:

1. `apt` cache is updated
2. Nginx is installed and enabled as a service
3. Directory `/var/www/autorent.local/html` is created with correct permissions (`www-data`)
4. Site files are copied to the serving directory
5. Server Block is enabled via symlink in `sites-enabled`
6. Nginx default site is removed (security best practice)
7. Nginx is restarted to apply the new configuration

### Nginx Server Block (`nginx/autorent.local.conf`)

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name autorent.192.168.56.10.nip.io autorent.local;

    root /var/www/autorent.local/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

The Server Block configuration makes the server **multi-tenant** — ready to host multiple sites on the same IP, which makes the environment easy to scale.

---

## Performance

During stress testing with **Apache Benchmark**, the default Linux connection limits and file descriptor caps were found to bottleneck throughput under high load.

After tuning Nginx's `worker_connections` and the operating system limits:

| Metric | Result |
|---|---|
| Requests per second | **~33,000 req/s** |
| Error rate | **0%** |
| Testing tool | Apache Benchmark (`ab`) |

This proves the infrastructure is ready to handle real traffic spikes, even on a local development VM.

---

## The Application

**AutoRent** is a car rental web application demonstrating async API integration.

**Features:**
- Vehicle catalogue with three categories (Economy, Family SUV, Premium Sports)
- Booking modal with dynamic total calculation based on number of days
- Payment simulation via **Fetch API**, sending JSON data to a mock backend ([jsonplaceholder.typicode.com](https://jsonplaceholder.typicode.com))
- Booking confirmation code returned by the API

---

## License

This is a fictional project created for Infrastructure as Code demonstration purposes.
