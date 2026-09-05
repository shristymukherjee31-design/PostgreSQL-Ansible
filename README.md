# PostgreSQL-Ansible

```text
postgres-ansible/
│
├── inventory
├── ansible.cfg
├── postgres.yml
│
├── templates/
│   └── postgresql.conf.j2
│
├── screenshots/
│   ├── playbook-success.png
│   └── postgresql-service-active.png
│
└── README.md
```

## `README.md`

```markdown
# PostgreSQL Automation using Ansible

## Project Overview

This project demonstrates the automation of PostgreSQL installation, configuration, service management, and verification using Ansible.

Ansible is used from a control node to configure an Ubuntu server running on AWS EC2.

The project focuses on practical implementation of core Ansible concepts such as Inventory, Ad-hoc Commands, Playbooks, Variables, Facts, Apt, Service, Template, Jinja2, Register, Debug, and Handlers.

---

## Objectives

The main objectives of this project are:

- Automate PostgreSQL installation using Ansible.
- Manage PostgreSQL service using Ansible.
- Use variables for PostgreSQL configuration.
- Collect operating system information using Ansible Facts.
- Configure PostgreSQL using Jinja2 templates.
- Use handlers to restart PostgreSQL when configuration changes.
- Register command output using the Register keyword.
- Display command results using the Debug module.
- Verify PostgreSQL service status after deployment.

---

## Technologies Used

| Technology | Purpose |
|------------|---------|
| Ansible | Automation and configuration management |
| Ubuntu | Managed Node |
| AWS EC2 | Remote Server |
| PostgreSQL | Database Server |
| YAML | Ansible Playbook |
| Jinja2 | Dynamic Configuration |
| WSL | Ansible Control Node |

---

## Architecture

```text
                    Ansible Control Node
                         WSL / Linux
                              |
                              |
                         SSH Connection
                              |
                              v
                    AWS EC2 Ubuntu Server
                              |
                              v
                       PostgreSQL 16
                              |
             +----------------+----------------+
             |                |                |
             v                v                v
        Installation     Configuration     Service
             |                |                |
          apt module      Jinja2 Template   service module
                              |
                              v
                         Verification
                              |
                              v
                    Register + Debug
```

---

# Project Structure

```text
postgres-ansible/
│
├── inventory
├── ansible.cfg
├── postgres.yml
│
├── templates/
│   └── postgresql.conf.j2
│
├── screenshots/
│   ├── playbook-success.png
│   └── postgresql-service-active.png
│
└── README.md
```

### File Description

### `inventory`

Contains the managed PostgreSQL server details.

Example:

```ini
[Database]
postgres ansible_host=YOUR_EC2_IP ansible_user=ubuntu
```

---

### `ansible.cfg`

Contains Ansible configuration such as inventory location and SSH private key.

Example:

```ini
[defaults]
inventory = ./inventory
private_key_file = /path/to/key.pem
host_key_checking = False
```

---

### `postgres.yml`

Main Ansible playbook responsible for:

- Gathering facts
- Installing PostgreSQL
- Starting PostgreSQL
- Configuring PostgreSQL
- Restarting PostgreSQL when required
- Checking PostgreSQL status
- Displaying the final status

---

### `templates/postgresql.conf.j2`

Jinja2 template used to dynamically configure PostgreSQL.

Example:

```jinja2
listen_addresses = '{{ postgres_listen_address }}'
port = {{ postgres_port }}
max_connections = {{ postgres_max_connections }}
```

---

# Ansible Concepts Used

## 1. Inventory

Inventory defines the servers that Ansible will manage.

```ini
[Database]
postgres ansible_host=YOUR_EC2_IP ansible_user=ubuntu
```

The `Database` group is used in the playbook:

```yaml
hosts: Database
```

---

## 2. Ansible Configuration

The `ansible.cfg` file tells Ansible where the inventory file is located and which SSH private key should be used.

```ini
[defaults]
inventory = ./inventory
private_key_file = /path/to/key.pem
host_key_checking = False
```

---

## 3. Facts

Ansible automatically gathers system information from the managed node.

In this project, the operating system is displayed using:

```yaml
- name: Display operating system
  ansible.builtin.debug:
    msg: "Operating System: {{ ansible_facts.distribution }}"
```

Output:

```text
Operating System: Ubuntu
```

---

## 4. Variables

Variables are used to avoid hardcoding PostgreSQL configuration values.

```yaml
vars:
  postgres_version: "16"
  postgres_port: 5432
  postgres_max_connections: 100
  postgres_listen_address: "localhost"
```

The PostgreSQL version variable is then reused:

```yaml
name: "postgresql-{{ postgres_version }}"
```

and:

```yaml
dest: "/etc/postgresql/{{ postgres_version }}/main/postgresql.conf"
```

This makes the playbook easier to maintain.

---

## 5. Apt Module

The `apt` module is used to install PostgreSQL on Ubuntu.

```yaml
- name: Install PostgreSQL
  ansible.builtin.apt:
    name: "postgresql-{{ postgres_version }}"
    state: present
    update_cache: true
```

---

## 6. Service Module

The PostgreSQL service is started and enabled using the service module.

```yaml
- name: Ensure PostgreSQL service is running
  ansible.builtin.service:
    name: postgresql
    state: started
    enabled: true
```

---

## 7. Template + Jinja2

The `template` module copies the Jinja2 template to the managed server.

```yaml
- name: Configure PostgreSQL
  ansible.builtin.template:
    src: templates/postgresql.conf.j2
    dest: "/etc/postgresql/{{ postgres_version }}/main/postgresql.conf"
  notify: Restart PostgreSQL
```

The Jinja2 variables are replaced with their actual values when Ansible processes the template.

---

## 8. Handler

A handler is used to restart PostgreSQL when the configuration changes.

The template task notifies the handler:

```yaml
notify: Restart PostgreSQL
```

Handler:

```yaml
handlers:

  - name: Restart PostgreSQL
    ansible.builtin.service:
      name: postgresql
      state: restarted
```

The handler runs only when the task that notified it reports a change.

---

## 9. Register

The `register` keyword stores the output of a task in a variable.

```yaml
- name: Check PostgreSQL service status
  ansible.builtin.command:
    cmd: systemctl is-active postgresql
  register: postgres_status
  changed_when: false
```

The output is stored inside:

```text
postgres_status
```

---

## 10. Debug

The Debug module is used to display the registered output.

```yaml
- name: Display PostgreSQL service status
  ansible.builtin.debug:
    var: postgres_status.stdout
```

Successful output:

```text
"postgres_status.stdout": "active"
```

---

# Playbook Execution

## Step 1: Test Ansible Connectivity

```bash
ansible Database -m ansible.builtin.ping
```

Expected:

```text
postgres | SUCCESS => {
    "ping": "pong"
}
```

---

## Step 2: Check Playbook Syntax

Before executing the playbook, syntax was verified using:

```bash
ansible-playbook postgres.yml --syntax-check
```

Successful result:

```text
playbook: postgres.yml
```

---

## Step 3: Run the Playbook

```bash
ansible-playbook postgres.yml
```

The playbook successfully completed all tasks.

Final Play Recap:

```text
postgres : ok=7 changed=0 unreachable=0 failed=0
```

---

# Successful Playbook Execution

The following screenshot shows the successful execution of the PostgreSQL Ansible playbook.

It includes:

- Gathering Facts
- Ubuntu operating system detection
- PostgreSQL installation
- PostgreSQL service management
- PostgreSQL configuration
- Service status checking
- Debug output
- Successful play recap

<img width="1917" height="841" alt="Screenshot 2026-09-05 204402" src="https://github.com/user-attachments/assets/47d41fc7-2a08-4aca-b3c1-22dfcb4da6cf" />


---

# PostgreSQL Service Verification

After running the playbook, PostgreSQL service status was verified using an Ansible ad-hoc command.

Command:

```bash
ansible Database -m ansible.builtin.command -a "systemctl is-active postgresql"
```

Successful output:

```text
postgres | CHANGED | rc=0 >>
active
```

The `active` status confirms that the PostgreSQL service is running on the Ubuntu managed node.

<img width="1537" height="112" alt="Screenshot 2026-09-05 204908" src="https://github.com/user-attachments/assets/c16e7982-3c20-43d2-8ff0-20759965c0d9" />


---

# Verification Commands

## Check Ansible Connectivity

```bash
ansible Database -m ansible.builtin.ping
```

## Check Inventory

```bash
ansible-inventory --graph
```

## Check Playbook Syntax

```bash
ansible-playbook postgres.yml --syntax-check
```

## Run Playbook

```bash
ansible-playbook postgres.yml
```

## Check PostgreSQL Service

```bash
ansible Database -m ansible.builtin.command -a "systemctl is-active postgresql"
```

## Check PostgreSQL Version

```bash
ansible Database -m ansible.builtin.command -a "psql --version"
```

---

# Learning Outcome

Through this project, the following Ansible concepts were implemented practically:

- Inventory
- Ad-hoc Commands
- Playbook
- Apt Module
- Service Module
- Variables
- Facts
- Template
- Jinja2
- Register
- Debug
- Handlers

The project demonstrates how a database server can be installed and managed automatically instead of performing each configuration step manually.

---

# Conclusion

The PostgreSQL automation project was successfully implemented using Ansible.

The playbook successfully:

1. Detected the Ubuntu operating system.
2. Installed PostgreSQL.
3. Started and enabled the PostgreSQL service.
4. Applied PostgreSQL configuration using a Jinja2 template.
5. Used a handler for PostgreSQL restart.
6. Registered the service status.
7. Displayed the service status using Debug.
8. Verified that PostgreSQL is in an `active` state.

This project provides practical experience with Ansible automation and demonstrates how multiple core Ansible concepts can be combined in a real-world PostgreSQL deployment.

---

## Author

**Shristy**
