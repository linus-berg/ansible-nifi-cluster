# Ansible NiFi Cluster Deployment

This project provides a set of Ansible roles to deploy and manage an Apache NiFi cluster using its built-in (embedded) ZooKeeper. It includes automated configuration for clustering, state management, and user/authorization management.

## Features

- **Clustered Deployment**: Automatically configures NiFi nodes to form a cluster.
- **Embedded ZooKeeper**: Configures the built-in ZooKeeper on all nodes, eliminating the need for an external ZK cluster.
- **User Management**: Manages NiFi users and initial admin/node identities directly from Ansible variables.
- **Service Management**: Deploys a `systemd` service for reliable NiFi process management.
- **Dynamic Configuration**: Derives ZooKeeper connection strings and node identities dynamically from the Ansible inventory.

## Prerequisites

- **Ansible**: 2.9 or higher installed on your control machine.
- **Target Nodes**:
  - Ubuntu/Debian or CentOS/RedHat.
  - SSH access with `sudo` privileges.
  - At least 3 nodes are recommended for a production-like cluster.

## Directory Structure

```text
.
├── group_vars/
│   └── all.yml           # Central configuration for version, users, and cluster settings
├── inventory/
│   └── hosts.ini         # Define your NiFi cluster nodes here
├── roles/
│   ├── common/           # Installs Java and required utilities
│   ├── nifi/             # Downloads, extracts, and configures NiFi and ZK
│   └── nifi_security/    # Manages users.xml and authorizers.xml
├── playbook.yml          # Main orchestration playbook
└── README.md             # You are here
```

## Quick Start

### 1. Configure the Inventory
Edit `inventory/hosts.ini` and add your NiFi node IP addresses or hostnames:

```ini
[nifi]
node1 ansible_host=192.168.1.10
node2 ansible_host=192.168.1.11
node3 ansible_host=192.168.1.12
```

### 2. Customize Variables
Update `group_vars/all.yml` to match your requirements:
- **`nifi_version`**: Specify the Apache NiFi version.
- **`nifi_initial_admin`**: The identity of the initial administrator (e.g., a username or CN).
- **`nifi_users`**: A list of users to pre-populate in `users.xml`.

### 3. Run the Playbook
Execute the following command to deploy the cluster:

```bash
ansible-playbook -i inventory/hosts.ini playbook.yml
```

## User Management

Users are managed via the `nifi_security` role, which templates `users.xml` and `authorizers.xml`. 

- **Static Management**: Adding a user to `nifi_users` in `group_vars/all.yml` and re-running the playbook will update the `users.xml` on all nodes.
- **Initial Admin**: The `nifi_initial_admin` is granted full access in `authorizers.xml` upon initial deployment.
- **Cluster Nodes**: The playbook automatically adds all nodes in the `nifi` inventory group as "Node Identities" in `authorizers.xml`, which is required for secure cluster communication.

## Important Notes

- **Security**: This example uses HTTP for simplicity. For production environments, it is **highly recommended** to enable HTTPS by providing keystores/truststores in `nifi.properties.j2`.
- **Sensitive Key**: Ensure you change the `nifi_sensitive_props_key` in `group_vars/all.yml` to a secure, unique string.
- **Restarting**: Changing NiFi or ZooKeeper configuration via the playbook will trigger a restart of the NiFi service on the target nodes.
