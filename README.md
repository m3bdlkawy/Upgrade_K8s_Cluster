# Kubernetes Cluster Upgrade Automation

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

This Ansible playbook automates the process of upgrading Kubernetes clusters across **both Debian and RHEL/CentOS** systems. It follows Kubernetes upgrade best practices while handling distribution-specific package management.

---

## 📚 Official Documentation References

### Ansible
- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible Playbooks](https://docs.ansible.com/ansible/latest/playbook_guide/index.html)
- [Ansible Modules for Kubernetes](https://docs.ansible.com/ansible/latest/collections/community/kubernetes/k8s_module.html)

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

## Usage Example

```bash
ansible-playbook -i inventory playbook.yml 
