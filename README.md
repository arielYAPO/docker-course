# DevOps — Ansible TP

## Question 3-1 — Inventory and Basic Commands

### Inventory

I created an Ansible inventory in `ansible/inventories/setup.yml`.

The inventory defines my remote Takima server in a group called `prod`.

It also contains connection variables:

- `ansible_user`: the SSH username (`admin`).
- `ansible_ssh_private_key_file`: the path to my SSH private key.

The inventory allows Ansible to identify the server and connect to it through SSH.

### Basic Commands

**Test the connection:**

`ansible all -i inventories/setup.yml -m ping`

The server responds with `pong`, confirming that Ansible can communicate with it.

**Retrieve server information:**

`ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"`

The `setup` module retrieves system facts. My server runs Debian 12.

**Remove Apache:**

`ansible all -i inventories/setup.yml -m apt -a "name=apache2 state=absent" --become`

The `apt` module manages packages, `state=absent` ensures Apache is removed, and `--become` enables administrator privileges.

Running the command again should not produce additional changes. This is called **idempotency**.

## Question 3-2 — Playbook and Docker Role

### Playbook

I created `ansible/playbook.yml` to automate Docker installation on my remote server.

The playbook uses:

- `hosts: all` to select the servers.
- `gather_facts: true` to retrieve server information.
- `become: true` to execute privileged operations.
- `roles: docker` to execute the Docker installation role.

### Docker Role

I created the Docker role using:

`ansible-galaxy init roles/docker`

The installation tasks are located in `ansible/roles/docker/tasks/main.yml`.

The role performs the following operations:

1. Installs required packages.
2. Adds Docker's official GPG key and APT repository.
3. Installs Docker Engine.
4. Creates a Python virtual environment.
5. Installs the Docker Python SDK.
6. Ensures the Docker service is running.

Using a role makes the playbook shorter, easier to maintain, and reusable.

### Verification

From the `ansible` directory, execute:

`ansible-playbook -i inventories/setup.yml playbook.yml`

To verify the Docker installation:

`ansible all -i inventories/setup.yml -a "docker --version"`

To verify the Docker service:

`ansible all -i inventories/setup.yml -a "systemctl is-active docker"`

The service returned `active`, confirming that Docker was running on the remote server.
