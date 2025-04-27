---

# Kubernetes Cluster Upgrade Automation

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

This Ansible playbook automates the process of upgrading Kubernetes clusters across **both Debian and RHEL/CentOS** systems. It follows Kubernetes upgrade best practices while handling distribution-specific package management.

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

## Variables Used in the Playbook

The following variables are used in the playbook to configure the behavior of the upgrade process. These variables can be customized as needed:

### Kubeadm Upgrade Configuration
```yaml
kubeadm_ignore_preflight_errors:
  - CoreDNSUnsupportedPlugins
  - CoreDNSMigration
  - CertExpired

kubeadm_force_upgrade: true
kubeadm_etcd_upgrade: true
kubeadm_skip_preflight: false
kubeadm_certificate_renewal: true
upgrade_timeout: 1800 # 30 minutes timeout for upgrade operations
```
- **`kubeadm_ignore_preflight_errors`**: List of preflight errors to ignore during the upgrade process.
- **`kubeadm_force_upgrade`**: Forces the upgrade even if preflight checks fail.
- **`kubeadm_etcd_upgrade`**: Ensures etcd is upgraded alongside the cluster.
- **`kubeadm_skip_preflight`**: Skips preflight checks if set to `true`.
- **`kubeadm_certificate_renewal`**: Enables automatic renewal of certificates during the upgrade.
- **`upgrade_timeout`**: Timeout (in seconds) for the entire upgrade operation.

---

### Kubernetes Repository Configuration
```yaml
kubernetes_repo_base_url: "https://pkgs.k8s.io/core:/stable"
kubernetes_deb_repo_format: "deb [signed-by={{ kubernetes_key_path }}] {{ kubernetes_repo_base_url }}:/v{{ next_minor_version.split('.')[0] }}.{{ next_minor_version.split('.')[1] }}/deb/ /"
kubernetes_rpm_repo_format: "{{ kubernetes_repo_base_url }}:/v{{ next_minor_version.split('.')[0] }}.{{ next_minor_version.split('.')[1] }}/rpm/"
```
- **`kubernetes_repo_base_url`**: Base URL for the Kubernetes package repository.
- **`kubernetes_deb_repo_format`**: Format for Debian-based systems to fetch Kubernetes packages.
- **`kubernetes_rpm_repo_format`**: Format for RPM-based systems to fetch Kubernetes packages.

---

### Keyring Configuration
```yaml
kubernetes_key_dir: "/usr/share/keyrings"
kubernetes_key_temp_path: "/tmp/kubernetes-apt-keyring.gpg"
kubernetes_key_path: "{{ kubernetes_key_dir }}/kubernetes-apt-keyring.gpg"
```
- **`kubernetes_key_dir`**: Directory where the GPG keyring is stored.
- **`kubernetes_key_temp_path`**: Temporary path for downloading the GPG key.
- **`kubernetes_key_path`**: Final path for the GPG keyring.

---

### Version Selection Behavior
```yaml
version_selection_strategy: "next_minor" # next_minor or latest
allow_downgrade: false
```
- **`version_selection_strategy`**: Determines whether to upgrade to the next minor version (`next_minor`) or the latest available version (`latest`).
- **`allow_downgrade`**: Allows downgrading the Kubernetes version if set to `true`.

---

### Timeout Settings
```yaml
repo_update_timeout: 30
package_install_timeout: 600 # 10 minutes for package operations
drain_timeout: 300 # seconds for drain operation
```
- **`repo_update_timeout`**: Timeout (in seconds) for updating package repositories.
- **`package_install_timeout`**: Timeout (in seconds) for package installation.
- **`drain_timeout`**: Timeout (in seconds) for draining nodes during the upgrade.

---

### Node Upgrade Configuration
```yaml
kube_config: "/home/ansible/.kube/config" # default kubeconfig location
# kube_config: /etc/kubernetes/admin.conf
```
- **`kube_config`**: Path to the kubeconfig file used to interact with the Kubernetes cluster. Defaults to `/home/ansible/.kube/config`.

---

### Backup Configuration
```yaml
backup_dir: "/opt/k8s-pre-upgrade-backup-{{ ansible_date_time.date }}"
etcd_cert_dir: "/etc/kubernetes/pki/etcd"
etcd_snapshot_prefix: "etcd-snapshot"
kubeadm_config_path: "/etc/kubernetes"
kubeconfig_path: "/etc/kubernetes/admin.conf"
etcd_backup_dir: "/opt/etcd-backups"
```
- **`backup_dir`**: Directory where pre-upgrade backups are stored, with a timestamp appended.
- **`etcd_cert_dir`**: Directory containing etcd certificates.
- **`etcd_snapshot_prefix`**: Prefix for etcd snapshot files.
- **`kubeadm_config_path`**: Path to the kubeadm configuration directory.
- **`kubeconfig_path`**: Path to the kubeconfig file for administrative access.
- **`etcd_backup_dir`**: Directory where etcd backups are stored.

---

### etcd Configuration
```yaml
etcd_version: "v3.5.12"
etcd_cert: "/etc/kubernetes/pki/etcd/server.crt"
etcd_key: "/etc/kubernetes/pki/etcd/server.key"
etcd_ca: "/etc/kubernetes/pki/etcd/ca.crt"
etcd_endpoints: "https://127.0.0.1:2379"
etcd_binary_url: "https://github.com/etcd-io/etcd/releases/download/{{ etcd_version }}/etcd-{{ etcd_version }}-linux-amd64.tar.gz"
```
- **`etcd_version`**: Version of etcd to use for backups.
- **`etcd_cert`, `etcd_key`, `etcd_ca`**: Paths to etcd certificates and keys.
- **`etcd_endpoints`**: Endpoint URL for etcd.
- **`etcd_binary_url`**: URL to download the etcd binary.

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

## Notes

- Ensure that SSH access is configured for all nodes in the inventory file.
- The `backup_k8s`, `rhel_upgrade`, and `debian_upgrade` roles must be defined in your Ansible project directory.
- Always verify the health of your cluster before and after running the playbook.

