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

## 🔄 Workflow (K8s-Compliant)

1. **Pre-Flight Checks**  
   - Verifies cluster state meets [Kubernetes upgrade prerequisites](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#before-you-begin)  
   - Checks etcd health using `kubectl get pods -n kube-system`

2. **Backup Phase**  
   - Creates etcd snapshot (following [official backup guide](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster))  
   - Backs up `/etc/kubernetes` and manifests

3. **Upgrade Execution**  
   - Masters first, then workers ([per K8s upgrade sequence](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#upgrade-control-plane))  
   - Uses `kubeadm upgrade` commands  

---

## Behavior of the Playbook

This playbook is designed to upgrade your Kubernetes cluster **one minor version at a time**, as recommended by Kubernetes best practices. For example:
- If your current Kubernetes version is `v1.23.x`, running the playbook will upgrade it to `v1.24.x`.
- If your current Kubernetes version is `v1.24.x`, running the playbook will upgrade it to `v1.25.x`.

If the cluster is already at the latest stable version available in the repositories, the playbook will **skip the upgrade process** and exit without making any changes.

This ensures that:
1. The cluster remains stable and compliant with Kubernetes' "one minor version at a time" upgrade policy.
2. You avoid unnecessary upgrades or errors when the cluster is already up-to-date.

---

## Important Notes for Users

### Default `kube_config` Location
The playbook uses the default `kubeconfig` location (`/home/ansible/.kube/config`) to interact with the Kubernetes cluster. This is defined in the `playbook.yaml` file under the `kube_config` variable:

```yaml
vars:
  kube_config: "/home/ansible/.kube/config"
```

If your `kubeconfig` file is located elsewhere, you can override this variable by passing it as an extra argument when running the playbook:

```bash
ansible-playbook -i inventory playbook.yaml --extra-vars "kube_config=/path/to/your/kubeconfig"
```

### Customizing Role Options
Each role in this playbook (`backup_k8s`, `rhel_upgrade`, `debian_upgrade`) has its own default variables defined in the `roles/<role_name>/defaults/main.yaml` file. If you need to customize these options (e.g., changing backup paths, enabling/disabling certain features), you can modify the respective `main.yaml` file for each role.

For example:
- To customize the backup directory for the `backup_k8s` role, edit `roles/backup_k8s/defaults/main.yaml` and update the `backup_dir` variable.
- To adjust package manager options for the `rhel_upgrade` or `debian_upgrade` roles, modify their respective `defaults/main.yaml` files.

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

## Playbook (`playbook.yaml`)

Below is the structure of the Ansible playbook that performs the Kubernetes cluster upgrade:

```yaml
---
- name: Backup Kubernetes Components
  hosts: all
  gather_facts: true
  become: true
  roles:
  - role: backup_k8s

- name: Upgrade the whole Kubernetes cluster server by server
  hosts: all
  gather_facts: true
  become: true
  serial: 1
  tasks:
  - name: Include OS-specific upgrade role
    include_role:
      name: "{{ 'rhel_upgrade' if ansible_os_family == 'RedHat' else 'debian_upgrade' }}"
    vars:
      kube_config: "/home/ansible/.kube/config"
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
---
## Example Upgrade Output

### Successful Master Node Upgrade
![Master Node Upgrade Summary](/home/mohamed/Pictures/Screenshots/Screenshot_from_2025-04-29_15-50-32.png)

This output shows a successful upgrade of a master node from Kubernetes v1.31.8 to v1.32.4, including:
- System architecture
- Current and target versions
- Repository used
- Package manager details

### Comprehensive Cluster Status After Upgrade
![Cluster Upgrade Summary](/home/mohamed/Pictures/Screenshots/Screenshot_from_2025-04-29_16-03-42.png)

This output shows:
- Detailed upgrade status across the cluster
- Node versions before and after upgrade
- Drain/uncordon operations status
- Cluster-wide version consistency check
---