# Ansible Role: Harbor Registry

<p align="left">
  <img src="https://github.com/user-attachments/assets/e20ece47-881d-4a9d-b3af-fc7209330939" width="550" alt="Harbor Logo"/>
</p>

A lightweight and reusable Ansible role to install and configure the [Harbor](https://goharbor.io) container registry with SSL support, custom admin credentials, and flexible configuration.

---

## Requirements

- Docker must be installed on the target system.
- Docker Compose must be installed.
- System Resource Requirements:

| Resource | Minimum     | Recommended |
|----------|-------------|-------------|
| CPU      | 2 vCPU      | 4 vCPU      |
| Memory   | 4 GB RAM    | 8 GB RAM    |
| Disk     | 40 GB Disk  | 160 GB Disk |

> For full details, refer to [Harbor Installation Prerequisites](https://goharbor.io/docs/2.13.0/install-config/installation-prereqs/)

---

## Role Variables

| Variable | Description | Default / Example |
|---------|-------------|-------------------|
| `harbor_hostname` | The domain or IP for accessing Harbor UI and registry. Must be reachable externally (do not use `localhost` or `127.0.0.1`). | `"registry.example.com"` |
| `harbor_admin_password` | Initial password for the Harbor admin user. Used only during the first installation. | `"Harbor12345"` |
| `harbor_certificate_path` | Path to the SSL certificate file (used by Nginx for HTTPS). | `"/data/cert/harbor.crt"` |
| `harbor_private_key_path` | Path to the SSL private key file. | `"/data/cert/harbor.key"` |

You can override these values in your playbook or inventory to customize your Harbor deployment.

---

## Dependencies

None.

---

## Example Playbook

```yaml
- name: Install and configure Harbor
  hosts: harbor_servers
  become: true
  roles:
    - role: harbor_registry
      harbor_hostname: "registry.ldc.opstree.dev"
      harbor_admin_password: "Harbor12345"
      harbor_certificate_path: "/data/cert/harbor.crt"
      harbor_private_key_path: "/data/cert/harbor.key" 
```

## License

BSD License

## Author

Sharvari Khamkar
