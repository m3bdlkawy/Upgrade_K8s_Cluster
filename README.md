---

# Kubernetes Cluster Upgrade Automation

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

This Ansible playbook automates the process of upgrading Kubernetes clusters across **both Debian and RHEL/CentOS** systems. It follows Kubernetes upgrade best practices while handling distribution-specific package management.

---

## Table of Contents
1. [Official Documentation References](#official-documentation-references)
2. [Features](#features)
3. [Supported Platforms](#supported-platforms)
4. [Parameters](#parameters)
   - [Kubeadm Upgrade Configuration](#kubeadm-upgrade-configuration)
   - [Kubernetes Repository Configuration](#kubernetes-repository-configuration)
   - [Keyring Configuration](#keyring-configuration)
   - [Version Selection Behavior](#version-selection-behavior)
   - [Timeout Settings](#timeout-settings)
   - [Node Upgrade Configuration](#node-upgrade-configuration)
   - [Backup Configuration](#backup-configuration)
   - [etcd Configuration](#etcd-configuration)
5. [Inventory Example](#inventory-example)
6. [Usage Instructions](#usage-instructions)
7. [Customizing Parameters](#customizing-parameters)
8. [Troubleshooting](#troubleshooting)
9. [Notes](#notes)
10. [License](#license)

---

## 📚 Official Documentation References

### Ansible
- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible Playbooks](https://docs.ansible.com/ansible/latest/playbook_guide/index.html)

### Kubernetes
- [Official Kubernetes Upgrade Guide](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
- [kubeadm Upgrade Documentation](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-upgrade/)
- [Cluster Backup and Recovery](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)
- [Draining Nodes Safely](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/)

---

## Features

- **Cross-platform Support**: Works with both Debian/Ubuntu and RHEL/CentOS distributions  
- **K8s-Compliant Upgrades**: Follows [official Kubernetes upgrade procedures](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)  
- **Distro-aware Package Management**: Uses `apt` or `yum/dnf` based on OS detection  

---

## Supported Platforms

| OS Family      | Versions Tested          | Package Manager | K8s Version Compatibility |
|----------------|--------------------------|-----------------|---------------------------|
| Debian/Ubuntu  | 20.04+, Debian 10+       | apt             | v1.23+                    |
| RHEL/CentOS    | 7+, 8+, Rocky Linux 8+   | yum/dnf         | v1.22+                    |

---

## Parameters

### Kubeadm Upgrade Configuration

These parameters control the behavior of the `kubeadm` upgrade process.

| Name                                      | Description                                                                                   | Value                             |
|-------------------------------------------|-----------------------------------------------------------------------------------------------|-----------------------------------|
| `kubeadm_ignore_preflight_errors`         | List of preflight errors to ignore during the upgrade process                                 | `["CoreDNSUnsupportedPlugins", "CoreDNSMigration", "CertExpired"]` |
| `kubeadm_force_upgrade`                   | Forces the upgrade even if preflight checks fail                                              | `true`                            |
| `kubeadm_etcd_upgrade`                    | Ensures etcd is upgraded alongside the cluster                                                | `true`                            |
| `kubeadm_skip_preflight`                  | Skips preflight checks if set to `true`                                                       | `false`                           |
| `kubeadm_certificate_renewal`             | Enables automatic renewal of certificates during the upgrade                                  | `true`                            |
| `upgrade_timeout`                         | Timeout (in seconds) for the entire upgrade operation                                         | `1800` (30 minutes)               |

---

### Kubernetes Repository Configuration

These parameters define the package repositories for Kubernetes. Note that `kubernetes_deb_repo_format` dynamically uses `kubernetes_key_path`.

| Name                                      | Description                                                                                   | Value                             |
|-------------------------------------------|-----------------------------------------------------------------------------------------------|-----------------------------------|
| `kubernetes_repo_base_url`                | Base URL for the Kubernetes package repository                                                | `"https://pkgs.k8s.io/core:/stable"` |
| `kubernetes_deb_repo_format`              | Format for Debian-based systems to fetch Kubernetes packages                                  | `"deb [signed-by={{ kubernetes_key_path }}] {{ kubernetes_repo_base_url }}:/v{{ next_minor_version.split('.')[0] }}.{{ next_minor_version.split('.')[1] }}/deb/ /"` |
| `kubernetes_rpm_repo_format`              | Format for RPM-based systems to fetch Kubernetes packages                                     | `"{{ kubernetes_repo_base_url }}:/v{{ next_minor_version.split('.')[0] }}.{{ next_minor_version.split('.')[1] }}/rpm/"` |

---

### Keyring Configuration

These parameters configure the GPG keyring used for verifying Kubernetes packages.

| Name                                      | Description                                                                                   | Value                             |
|-------------------------------------------|-----------------------------------------------------------------------------------------------|-----------------------------------|
| `kubernetes_key_dir`                      | Directory where the GPG keyring is stored                                                     | `"/usr/share/keyrings"`           |
| `kubernetes_key_temp_path`                | Temporary path for downloading the GPG key                                                    | `"/tmp/kubernetes-apt-keyring.gpg"` |
| `kubernetes_key_path`                     | Final path for the GPG keyring                                                                | `"{{ kubernetes_key_dir }}/kubernetes-apt-keyring.gpg"` |

---

### Version Selection Behavior

These parameters control how the playbook selects the Kubernetes version to upgrade to.

| Name                                      | Description                                                                                   | Value                             |
|-------------------------------------------|-----------------------------------------------------------------------------------------------|-----------------------------------|
| `version_selection_strategy`              | Determines whether to upgrade to the next minor version (`next_minor`) or the latest available version (`latest`) | `"next_minor"`                   |
| `allow_downgrade`                         | Allows downgrading the Kubernetes version if set to `true`                                    | `false`                           |

---

### Timeout Settings

These parameters define timeouts for various operations during the upgrade process.

| Name                                      | Description                                                                                   | Value                             |
|-------------------------------------------|-----------------------------------------------------------------------------------------------|-----------------------------------|
| `repo_update_timeout`                     | Timeout (in seconds) for updating package repositories                                        | `30`                              |
| `package_install_timeout`                 | Timeout (in seconds) for package installation                                                 | `600` (10 minutes)                |
| `drain_timeout`                           | Timeout (in seconds) for draining nodes during the upgrade                                    | `300` (5 minutes)                 |

---

### Node Upgrade Configuration

These parameters configure node-specific settings for the upgrade process.

| Name                                      | Description                                                                                   | Value                             |
|-------------------------------------------|-----------------------------------------------------------------------------------------------|-----------------------------------|
| `kube_config`                             | Path to the kubeconfig file used to interact with the Kubernetes cluster                      | `"/home/ansible/.kube/config"`    |

---

### Backup Configuration

These parameters control backup-related settings.

| Name                                      | Description                                                                                   | Value                             |
|-------------------------------------------|-----------------------------------------------------------------------------------------------|-----------------------------------|
| `backup_dir`                              | Directory where pre-upgrade backups are stored, with a timestamp appended                     | `"/opt/k8s-pre-upgrade-backup-{{ ansible_date_time.date }}"` |
| `etcd_cert_dir`                           | Directory containing etcd certificates                                                        | `"/etc/kubernetes/pki/etcd"`      |
| `etcd_snapshot_prefix`                    | Prefix for etcd snapshot files                                                                | `"etcd-snapshot"`                 |
| `kubeadm_config_path`                     | Path to the kubeadm configuration directory                                                   | `"/etc/kubernetes"`               |
| `kubeconfig_path`                         | Path to the kubeconfig file for administrative access                                         | `"/etc/kubernetes/admin.conf"`    |
| `etcd_backup_dir`                         | Directory where etcd backups are stored                                                       | `"/opt/etcd-backups"`             |

---

### etcd Configuration

These parameters configure etcd-related settings.

| Name                                      | Description                                                                                   | Value                             |
|-------------------------------------------|-----------------------------------------------------------------------------------------------|-----------------------------------|
| `etcd_version`                            | Version of etcd to use for backups                                                            | `"v3.5.12"`                       |
| `etcd_cert`                               | Path to the etcd server certificate                                                           | `"/etc/kubernetes/pki/etcd/server.crt"` |
| `etcd_key`                                | Path to the etcd server key                                                                   | `"/etc/kubernetes/pki/etcd/server.key"` |
| `etcd_ca`                                 | Path to the etcd CA certificate                                                               | `"/etc/kubernetes/pki/etcd/ca.crt"` |
| `etcd_endpoints`                          | Endpoint URL for etcd                                                                         | `"https://127.0.0.1:2379"`        |
| `etcd_binary_url`                         | URL to download the etcd binary                                                               | `"https://github.com/etcd-io/etcd/releases/download/{{ etcd_version }}/etcd-{{ etcd_version }}-linux-amd64.tar.gz"` |

---

## Inventory Example

The inventory file defines the master and worker nodes in your Kubernetes cluster. Replace `<master-ip>` and `<worker-ip>` with the actual IP addresses of your nodes.

```ini
[masters]
master1 ansible_host=<master-ip>

[workers]
worker1 ansible_host=<worker-ip>
worker2 ansible_host=<worker-ip>
```

---

## Usage Instructions

1. **Navigate to the Project Directory**  
   Change to the directory containing your inventory and playbook files:
   ```bash
   cd /path/to/your/repo
   ```

2. **Run the Ansible Playbook**  
   Execute the playbook using the following command:
   ```bash
   ansible-playbook -i inventory playbook.yaml
   ```
   Replace `/path/to/your/repo` with the actual path to your repository.

---

## Customizing Parameters

You can customize the default values in several ways:

1. **Using `--extra-vars` with Ansible Playbook**
   Override variables directly from the command line:
   ```bash
   ansible-playbook -i inventory playbook.yaml --extra-vars "kube_config=/path/to/custom/kubeconfig"
   ```

2. **Editing Role Defaults**
   Modify the default values in the role's `defaults/main.yml` file:
   ```yaml
   kube_config: "/custom/path/to/kubeconfig"
   ```

3. **Using an External Variables File**
   Create a `vars.yaml` file and include it when running the playbook:
   ```bash
   ansible-playbook -i inventory playbook.yaml -e @vars.yaml
   ```


---

## Troubleshooting

### Issue: Preflight Checks Fail
If preflight checks fail, ensure that all nodes meet the prerequisites outlined in the [Kubernetes Upgrade Guide](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/).

### Issue: Timeout During Upgrade
Increase the `upgrade_timeout` value in the playbook if the upgrade process is timing out:
```yaml
upgrade_timeout: 3600 # Increase timeout to 1 hour
```

---

## Notes

- Ensure that SSH access is configured for all nodes in the inventory file.
- The `backup_k8s`, `rhel_upgrade`, and `debian_upgrade` roles must be defined in your Ansible project directory.
- Always verify the health of your cluster before and after running the playbook.
---
## Example Upgrade Output

### Successful Master Node Upgrade
<p align="center">
  <img src="https://github.com/m3bdlkawy/Upgrade_K8s_Cluster/raw/main/images/screenshot-2025-04-29-15-50-32.png" alt="Upgrade Summary" width="800">
</p>

This output shows a successful upgrade of a master node from Kubernetes v1.31.8 to v1.32.4, including:
- System architecture
- Current and target versions
- Repository used
- Package manager details

### Comprehensive Cluster Status After Upgrade
<p align="center">
  <img src="https://github.com/m3bdlkawy/Upgrade_K8s_Cluster/raw/main/images/screenshot-2025-04-29-16-03-42.png" alt="Cluster Status" width="800">
</p>

This output shows:
- Detailed upgrade status across the cluster
- Node versions before and after upgrade
- Drain/uncordon operations status
- Cluster-wide version consistency check
---