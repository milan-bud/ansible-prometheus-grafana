# ansible-prometheus-grafana
Ansible playbook for Prometheus, Grafana and applicaition exporters (Redis,PostgreSQL,RabbitMQ) setup

# Infrastructure Monitoring & Exporters Deployment
# (Ansible playbook for Prometheus, Grafana and applicaition exporters (Redis,PostgreSQL,RabbitMQ) setup)

This Ansible project automates the installation, configuration, and management of Prometheus & Grafana monitoring stack across multiple Ubuntu nodes.

## What is Handled

* **Prometheus Server:** Deployed on single node (:9090).
* **Grafana Server:** Deployed on ingle node (:3000).
* **Node Exporter:** System-level OS metrics (CPU, RAM, Disk, Network) deployed on all target nodes (:9100).
* **Application Exporters (on specified nodes):**
  * PostgreSQL Exporter (:9187)
  * Redis Exporter (:9121)
  * RabbitMQ Exporter (Native Plugin, :15692)

---

## How It Works

1. **Version:** Every exporter role checks the currently installed binary version (`<binary> --version`) before downloading. If the version matches `group_vars/all.yaml`, download and extract tasks are skipped (`changed=0`).
2. **Dynamic exporter components:** Application exporters are conditionally installed on nodes based on the `exporters` array variable defined in `inventory.ini`.
3. **Templates** `roles/prometheus/templates/prometheus.yaml.j2` contains Prometheus scrape config 

---

## Repository Structure

```text
.
├── inventory.ini           # Node definitions, IP addresses, and host variables
├── site.yaml               # Master playbook orchestrating all roles
├── group_vars/
│   └── all.yaml            # Global software versions (e.g., node_exporter_version: 1.8.2)
└── roles/
    ├── node_exporter/      # OS metrics collector role
    ├── postgres_exporter/  # PostgreSQL metrics collector role
    ├── redis_exporter/     # Redis metrics collector role
    ├── rabbitmq_exporter/  # RabbitMQ metrics collector role
    ├── prometheus/         # Prometheus server role + config template
    └── grafana/            # Grafana server role
```


Configuration & What to Specify
1. Define Hosts (inventory.ini)

Specify your target host IP addresses and assign exported components:

[prometheus_server]
prometheus-node ansible_host=10.203.212.78

[grafana_server]
grafana-node ansible_host=10.203.212.78

[node_exporter_nodes]
node-1 ansible_host=10.203.212.78
node-2 ansible_host=10.203.212.97 exporters="['postgres', 'redis', 'rabbitmq']"

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/id_ed25519
ansible_python_interpreter=/usr/bin/python3


2. Set Binary Versions (group_vars/all.yaml)

Update software versions in a single place to trigger automated upgrades:

node_exporter_version: "1.8.2"
prometheus_version: "2.54.1"
postgres_exporter_version: "0.15.0"
redis_exporter_version: "1.61.0"


How to Use
Prerequisites

1.Copy your SSH public key to all target nodes:

cat ~/.ssh/id_ed25519.pub | multipass exec <node-name> -- bash -c 'cat >> ~/.ssh/authorized_keys'

2.Ensure target applications (PostgreSQL, Redis, RabbitMQ) are running on target hosts.


Execution

Run the main playbook from your control host:
# Run full deployment
ansible-playbook -i inventory.ini site.yaml

# Run syntax check
ansible-playbook -i inventory.ini site.yaml --syntax-check



Verification

After the playbook finishes with failed=0:

    Prometheus Targets: Visit http://<prometheus-ip>:9090/targets to verify all 5 endpoints show status UP.

    Grafana Dashboards: Visit http://<grafana-ip>:3000 (Default: admin / admin), add Prometheus as a data source, and import community dashboard IDs (1860, 9628, 763, 10991).
