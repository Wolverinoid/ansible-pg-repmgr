# ansible-pg-repmgr
# Ansible Role: role_name

A brief description of the role goes here.

## Requirements

Any pre-requisites that may not be covered by Ansible itself or the role should be mentioned here. For instance, if the role uses the EC2 module, it may be a good idea to mention in this section that the boto package is required.

## Role Variables

A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

```yaml
# defaults file for role_name
postgresql_version: "16"  # PostgreSQL version to install
```

## Dependencies

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

## Example Playbook

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```yaml
- hosts: servers
  roles:
    - { role: username.role_name, postgresql_version: "15" }  # Override default PostgreSQL version
```

## License

BSD

## Author Information

An optional section for the role authors to include contact information, or a website (HTML is not allowed).# Ansible Role: PostgreSQL with repmgr

This Ansible role installs and configures PostgreSQL with repmgr for high availability without keepalived. It sets up streaming replication between PostgreSQL servers and configures automatic failover using repmgr.

## Requirements

- Debian/Ubuntu or RHEL/CentOS Linux
- Ansible 2.9 or higher

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
    - postgresql-repmgr
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
    - postgresql-repmgr
```

## License

MIT

## Author Information

An optional section for the role authors to include contact information, or a website.