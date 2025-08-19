# ansible-pg-repmgr

This Ansible role installs and configures PostgreSQL with repmgr for high availability without keepalived. It sets up streaming replication between PostgreSQL servers and configures automatic failover using repmgr.

## Features

- Installs PostgreSQL and repmgr packages
- Configures streaming replication between PostgreSQL servers
- Sets up automatic failover with repmgr
- Supports both primary and standby node configurations
- Works with PostgreSQL 12, 13, 14, 15, and 16

## Requirements

- Debian/Ubuntu or RHEL/CentOS Linux
- Ansible 2.9 or higher
- Sudo access on target servers

## Role Variables

```yaml
# PostgreSQL version
postgresql_version: "16"

# Node role (primary or standby)
pg_role: "primary"

# PostgreSQL configuration
postgresql_data_dir: "/var/lib/postgresql/{{ postgresql_version }}/main"
postgresql_config_dir: "/etc/postgresql/{{ postgresql_version }}/main"
postgresql_bin_dir: "/usr/lib/postgresql/{{ postgresql_version }}/bin"
postgresql_service_name: "postgresql@{{ postgresql_version }}-main"

# PostgreSQL configuration parameters - Common for all nodes
postgresql_common_config_params:
  wal_level: replica
  max_wal_senders: 10
  max_replication_slots: 10
  hot_standby: on
  archive_mode: on
  archive_command: '/bin/true'

# PostgreSQL configuration parameters - Primary specific
postgresql_primary_config_params:
  wal_keep_segments: 128
  max_worker_processes: 8
  max_parallel_workers: 4

# PostgreSQL configuration parameters - Standby specific
postgresql_standby_config_params:
  hot_standby_feedback: on
  max_standby_streaming_delay: 30s
  max_worker_processes: 4
  max_parallel_workers: 2

# Replication settings
postgresql_replication_user: "repmgr"
postgresql_replication_password: "repmgr_password"  # Should be changed and stored securely
postgresql_replication_db: "repmgr"

# Node information
node_id: 1  # Should be unique for each node
node_name: "node1"  # Should be unique for each node
node_network_name: "{{ ansible_default_ipv4.address }}"  # Or specify a static IP

# Cluster information
cluster_name: "pg_cluster"
primary_node_id: 1  # ID of the primary node
primary_node_name: "node1"  # Name of the primary node
primary_node_address: "192.168.1.101"  # IP of the primary node

# PostgreSQL extensions
postgresql_extensions: []  # List of extensions to install and create
# Example:
# postgresql_extensions:
#   - name: "postgis"  # Extension name
#     db: "gisdb"      # Database to create extension in
#     schema: "public" # Optional schema
#     package: "postgresql-16-postgis-3" # Optional custom package name (if specified, package will be installed)
#   - name: "pg_stat_statements"  # Extension name only
#     db: "postgres"    # Only create extension in database, no package installation
```

## Dependencies

None.

## Example Playbook

### Primary Node Setup

```yaml
- hosts: pg_primary
  vars:
    pg_role: primary
    node_id: 1
    node_name: "node1"
    node_network_name: "192.168.1.101"
    primary_node_address: "192.168.1.101"
  roles:
    - ansible-pg-repmgr
```

### Standby Node Setup

```yaml
- hosts: pg_standby
  vars:
    pg_role: standby
    node_id: 2
    node_name: "node2"
    node_network_name: "192.168.1.102"
    primary_node_address: "192.168.1.101"
  roles:
    - ansible-pg-repmgr
```

## Verification

After deployment, you can verify the replication status:

```bash
# On primary node
sudo -u postgres repmgr cluster show

# Expected output
 ID | Name  | Role    | Status    | Upstream | Location | Priority | Timeline | Connection string
----+-------+---------+-----------+----------+----------+----------+----------+-------------------------------
 1  | node1 | primary | * running |          | default  | 100      | 1        | host=192.168.1.101 dbname=repmgr user=repmgr
 2  | node2 | standby |   running | node1    | default  | 100      | 1        | host=192.168.1.102 dbname=repmgr user=repmgr
```

## Failover Testing

To test automatic failover, you can stop PostgreSQL on the primary node:

```bash
# On primary node
sudo systemctl stop postgresql@16-main

# After a few moments, check the cluster status on the standby node
sudo -u postgres repmgr cluster show

# Expected output - node2 has been promoted to primary
 ID | Name  | Role    | Status    | Upstream | Location | Priority | Timeline | Connection string
----+-------+---------+-----------+----------+----------+----------+----------+-------------------------------
 1  | node1 | primary | ? failed  |          | default  | 100      | 1        | host=192.168.1.101 dbname=repmgr user=repmgr
 2  | node2 | primary | * running |          | default  | 100      | 2        | host=192.168.1.102 dbname=repmgr user=repmgr
```

## License

MIT

## Author Information

This role was created by the DevOps team. For more information, please contact devops@example.com.