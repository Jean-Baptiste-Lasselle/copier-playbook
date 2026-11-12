# 🚀 Ansible Playbook Copier Template

![Copier](https://shields.io)
![Ansible](https://shields.io)
![Molecule](https://shields.io)
![Pre-Commit](https://shields.io)

A production-grade [Copier](https://readthedocs.io) template for scaffolding standardized **Ansible playbook repositories** targeting a homogeneous fleet of `N` dev GNU/Linux servers with **rootless Podman** and **SSH username/password authentication**.

---

## ✨ What's Included

* **Homogeneous `N`-Node Static Inventory:** Generates `dev-node-01` through `dev-node-0N` with validated static IP addresses, structured for easy duplication into `staging` or `prod`.
* **State-of-the-Art Rootless Podman Role (`roles/podman`):**
  * Cross-platform support for **Debian 12/13**, **Ubuntu 24.04**, and **Rocky Linux 9**.
  * Configures `/etc/subuid` and `/etc/subgid` ranges, unprivileged port binding (`>= 80`), `systemd` user lingering, and user `podman.socket`.
  * Enforces SSH password authentication (`PasswordAuthentication yes` via `/etc/ssh/sshd_config.d/`).
* **Distro-Targeted Molecule Testing:** Automatically configures Molecule to test against your chosen GNU/Linux distribution (`debian13`, `debian12`, `ubuntu2404`, or `rocky9`).
* **Automated Quality Gates:** Enforces `pre-commit` installation at scaffolding time, paired with strict `ansible-lint` (`production` profile) and `yamllint`.

---

## 📋 Prerequisites

1. **Git** (`>= 2.30`)
2. **Python** (`>= 3.10`) with [`uv`](https://astral.sh) or `pipx`
3. **Copier** (`>= 9.0.0`)
4. **`pre-commit`** (required during `copier copy` to initialize Git hooks automatically)
5. **`sshpass`** on your Ansible control machine (Linux / WSL2) for SSH password authentication

---

## 🛠️ Usage Instructions

### 🐧 Linux & macOS

#### 1. Install Copier, `pre-commit`, and `sshpass`
```bash
# Install Copier and pre-commit
uv tool install copier
uv tool install pre-commit

# Install sshpass (Debian/Ubuntu)
sudo apt update && sudo apt install -y sshpass

# Or on RHEL/Rocky/Fedora
sudo dnf install -y sshpass
```

#### 2. Generate a New Ansible Playbook Repository
```bash
# From a remote Git repository
copier copy --trust gh:YOUR_ORG/ansible-playbook-copier-template ./my-linux-infra

# Or from a local clone
copier copy --trust /path/to/ansible-playbook-copier-template ./my-linux-infra
```

#### 3. Initialize the Generated Project
```bash
cd my-linux-infra

# Create a virtual environment and install Ansible tooling
python3 -m venv .venv
source .venv/bin/activate
pip install ansible-core ansible-lint molecule molecule-plugins[podman]

# Install required Ansible Galaxy collections
ansible-galaxy collection install -r requirements.yml
```

---

### 🪟 Windows (Git Bash)

> **Important Note for Windows Users:** Ansible cannot run natively on Windows without **WSL2** (Windows Subsystem for Linux). Use **Git Bash** to scaffold and edit your repository, then run `ansible-playbook` and `molecule` inside **WSL2**.

#### 1. Install Copier and `pre-commit` in Git Bash
```bash
# Using uv
uv tool install copier
uv tool install pre-commit

# Or using pip
python -m pip install --user --upgrade copier pre-commit
```

#### 2. Generate a New Ansible Playbook Repository
> ⚠️ **Git Bash Path Warning:** Always use forward slashes (`/c/Users/...`) when passing local paths in Git Bash.

```bash
# From a remote Git repository
winpty uvx copier copy --trust gh:YOUR_ORG/ansible-playbook-copier-template ./my-linux-infra

# Or from a local Windows directory
winpty uvx copier copy --trust /c/Users/$USERNAME/repos/ansible-playbook-copier-template ./my-linux-infra
```

#### 3. Set Up Ansible Control Node in WSL2
Open your **WSL2 Linux terminal** and navigate to the newly scaffolded directory:

```bash
cd /mnt/c/Users/$USER/my-linux-infra

# Install sshpass required for password-based SSH
sudo apt update && sudo apt install -y sshpass python3-venv

# Create a Linux virtual environment
python3 -m venv .venv
source .venv/bin/activate
pip install ansible-core ansible-lint molecule molecule-plugins[podman]

# Install Galaxy collections
ansible-galaxy collection install -r requirements.yml
```

---

## ⚙️ Template Prompts Reference

| Prompt | Default | Description |
| :--- | :--- | :--- |
| `project_name` | *(required)* | Name of the new Ansible repository (e.g., `infra-dev`). |
| `description` | `Ansible playbooks for dev...` | Short description placed in the generated `README.md`. |
| `linux_distro` | `debian13` | Target OS (`debian13`, `debian12`, `ubuntu2404`, or `rocky9`). |
| `server_count` | `1` | Number of identical GNU/Linux servers (`N`) in the `dev` inventory. |
| `server_ips` | `192.168.10.11` | Comma-separated list of `N` static IP addresses. |
| `remote_user` | `ansible` | SSH username and rootless Podman user on target servers. |
| `ssh_password` | *(required, hidden)* | SSH & `sudo` password stored in `vault.yml`. |
| `container_driver` | `podman` | Container driver for Molecule tests (`podman` or `docker`). |

---

## 📁 Generated Repository Layout

```text
my-linux-infra/
├── .ansible-lint                  # Strict production lint rules
├── .copier-answers.yml            # Tracks template version for future updates
├── .pre-commit-config.yaml        # Automated YAML & Ansible lint hooks
├── .yamllint                      # YAML formatting standards
├── ansible.cfg                    # Default inventory -> dev, SSH password auth configured
├── README.md                      # Tailored project documentation with server IP table
├── requirements.yml               # Required Ansible Galaxy collections
├── inventories/
│   └── dev/
│       ├── hosts.yml              # Static IP inventory (dev-node-01 .. dev-node-0N)
│       ├── group_vars/all/
│       │   ├── main.yml           # Non-sensitive environment & SSH user variables
│       │   └── vault.yml          # SSH & sudo password (encrypt with ansible-vault)
│       └── host_vars/
├── molecule/
│   └── default/
│       ├── converge.yml           # Executes the rootless podman role
│       ├── molecule.yml           # Targeted container test for chosen linux_distro
│       └── verify.yml             # Validates rootless Podman & SSH password policy
├── playbooks/
│   └── site.yml                   # Main entry-point playbook
└── roles/
    └── podman/                    # Cross-platform rootless Podman role
```

---

## 🔄 Updating Existing Projects from This Template

Whenever you update this template and push a new Git tag (e.g., `v0.2.0`), update any downstream playbook repository by running:

```bash
cd my-linux-infra
copier update --trust
```
