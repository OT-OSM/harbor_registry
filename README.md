Ansible Role: harbor_registry v
=========

[![Opstree Solutions][opstree_avatar]][opstree_homepage]<br/>[Opstree Solutions][opstree_homepage]

[opstree_homepage]: https://opstree.github.io/
[opstree_avatar]: https://img.cloudposse.com/150x150/https://github.com/opstree.png

An Ansible role to **install and configure Harbor Container Registry** using the official
Harbor installer.

---

Version History
---------------

| **Date** | **Version** | **Description** | **Changed By** |
|---------|-------------|-----------------|----------------|
| **Dec '30** | v1.0.0 | Initial production-ready release |  |

---

Salient Features
----------------

* Installs Harbor using the **official installer** (online or offline)
* Supports **external PostgreSQL and Redis**
* Multiple registry storage backends:
  * Filesystem
  * Amazon S3 / S3-compatible
  * Google Cloud Storage (GCS)
  * Azure Blob Storage
* systemd-based Harbor service management
* Secure secret handling using **Ansible Vault**
* Trivy vulnerability scanning
* Metrics, audit logging, and upload purging
* Idempotent and repeatable execution



Supported OS
------------

* Ubuntu: 20.04 / 22.04 / 24.04
* RHEL / Rocky Linux / AlmaLinux
* Any Linux distribution with:
  * systemd
  * Docker Engine
  * Docker Compose v2



Dependencies
------------

* Docker Engine
* Docker Compose v2
* Ansible ≥ 2.12
* External services (optional):
  * PostgreSQL
  * Redis
  * Object Storage (S3 / GCS / Azure)



## Repository Structure
```yaml
harbor_registry/
├── defaults/
│   └── main.yml          # Default (safe) configuration values
├── vars/
│   └── main.yml          # Internal constants (DO NOT override)
├── tasks/
│   ├── prereqs.yml       # Docker & compose validation
│   ├── install.yml       # Harbor installer download & extraction
│   ├── configure.yml    # harbor.yml rendering
│   ├── systemd.yml      # systemd service setup
│   └── main.yml
├── templates/
│   └── harbor.yml.j2     # Harbor configuration template
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
└── README.md
```


Role Variables
--------------
> ℹ️ All default values are defined in `defaults/main.yml`.  
> Override variables only when required using `group_vars` or inventory.


The following variables can be used to configure the **harbor_registry** role.

### Harbor Installer

These variables control how the Harbor installer is downloaded and installed.

| **Variable** | **Default Value** | **Description** | **Type** |
|-------------|------------------|-----------------|----------|
| `harbor_version` | `2.14.0` | Harbor version (without `v`) | Optional |
| `harbor_installer_type` | `offline` | Installer type (`online` or `offline`) | Optional |
| `harbor_install_dir` | `/opt` | Directory where Harbor is installed | Optional |
| `harbor_installer_name` | derived | Online installer filename | Internal |
| `harbor_offline_installer_name` | derived | Offline installer filename | Internal |
| `harbor_offline_installer_url` | derived | Offline installer download URL | Internal |
| `harbor_online_installer_url` | derived | Online installer download URL | Internal |

> ⚠️ Internal variables are derived automatically and should not be overridden.

### Core

| **Variable** | **Default Value** | **Description** | **Type** |
|-------------|------------------|-----------------|----------|
| `harbor_hostname` | — | Harbor hostname / FQDN | **Mandatory** |
| `harbor_data_dir` | `/data/harbor` | Base data directory for Harbor | Optional |
| `harbor_external_url` | — | External Harbor URL (if different from hostname) | Optional |
| `harbor_http_port` | `80` | HTTP port for Harbor | Optional |
| `harbor_https_enabled` | `false` | Enable HTTPS | Optional |
| `harbor_https_port` | `443` | HTTPS port | Optional |
| `harbor_ssl_cert_path` | — | TLS certificate path | Mandatory (if HTTPS enabled) |
| `harbor_ssl_key_path` | — | TLS private key path | Mandatory (if HTTPS enabled) |
| `harbor_log_level` | `info` | Harbor log level | Optional |
| `harbor_no_proxy` | `127.0.0.1,localhost,.local` | No-proxy addresses | Optional |



### Database (PostgreSQL)

| **Variable** | **Default Value** | **Description** | **Type** |
|-------------|------------------|-----------------|----------|
| `harbor_db_external` | `false` | Enable external PostgreSQL | Optional |
| `harbor_db_host` | — | PostgreSQL hostname | Mandatory (if external DB enabled) |
| `harbor_db_port` | `5432` | PostgreSQL port | Optional |
| `harbor_db_name` | `harbor_db` | Database name | Optional |
| `harbor_db_user` | — | Database username | Mandatory (if external DB enabled) |
| `harbor_db_sslmode` | `disable` | PostgreSQL SSL mode | Optional |

**Vault**

| **Secret Variable** | **Description** |
|-------------------|-----------------|
| `vault_harbor_db_password` | db password |


### Redis

| **Variable** | **Default Value** | **Description** | **Type** |
|-------------|------------------|-----------------|----------|
| `harbor_redis_external` | `false` | Enable external Redis | Optional |
| `harbor_redis_host` | — | Redis hostname | Mandatory (if external Redis enabled) |
| `harbor_redis_port` | `6379` | Redis port | Optional |
| `harbor_redis_db` | `0` | Redis DB index | Optional |

**Vault** (only if AUTH enabled)

| **Secret Variable** | **Description** |
|-------------------|-----------------|
| `vault_harbor_redis_password` | Redis AUTH password |

⚠️ **Internal Redis must not use authentication.**



### Registry Storage

| **Variable** | **Default Value** | **Description** | **Type** |
|-------------|------------------|-----------------|----------|
| `harbor_storage_type` | `filesystem` | Registry storage backend (`filesystem`, `s3`, `gcs`, `azure`) | Optional |
| `harbor_registry_storage_path` | `/data/harbor/registry` | Filesystem storage path | Mandatory (filesystem) |

### Amazon S3

| **Variable** | **Default Value** | **Description** | **Type** |
|-------------|------------------|-----------------|----------|
| `s3_region` | — | AWS region | Mandatory |
| `s3_bucket` | — | S3 bucket name | Mandatory |
| `s3_endpoint` | — | Custom S3 endpoint (MinIO, Ceph) | Optional |
| `s3_secure` | `true` | Use HTTPS | Optional |

#### Vault

| **Secret Variable** | **Description** |
|-------------------|-----------------|
| `s3_access_key` | S3 access key |
| `s3_secret_key` | S3 secret key |


### Google Cloud Storage (GCS)

| **Variable** | **Default Value** | **Description** | **Type** |
|-------------|------------------|-----------------|----------|
| `gcs_bucket` | — | GCS bucket name | Mandatory |

#### Vault

| **Secret Variable** | **Description** |
|-------------------|-----------------|
| `gcs_encoded_key` | Base64-encoded GCP service account JSON |


### Azure Blob Storage

| **Variable** | **Default Value** | **Description** | **Type** |
|-------------|------------------|-----------------|----------|
| `azure_account_name` | — | Azure storage account name | Mandatory |
| `azure_container` | — | Azure blob container name | Mandatory |

#### Vault

| **Secret Variable** | **Description** |
|-------------------|-----------------|
| `azure_account_key` | Azure storage account access key |


### Logging (MANDATORY ≥ Harbor 2.9)

| **Variable** | **Default Value** | **Description** | **Type** |
|-------------|------------------|-----------------|----------|
| `harbor_log_sweeper_duration` | `1` | Log cleanup interval (days) | Optional |
| `harbor_log_rotate_count` | `50` | Number of rotated log files | Optional |
| `harbor_log_rotate_size` | `200M` | Maximum size of a log file before rotation | Optional |
| `harbor_log_dir` | `/var/log/harbor` | Harbor log directory | Optional |


### Job Service (MANDATORY ≥ Harbor 2.8)

| **Variable** | **Default Value** | **Description** | **Type** |
|-------------|------------------|-----------------|----------|
| `harbor_job_workers` | `10` | Maximum number of job workers | Optional |
| `harbor_job_max_duration_hours` | `24` | Maximum job execution duration (hours) | Optional |
| `harbor_job_loggers` | `STD_OUTPUT, FILE` | Job log backends | Optional |
| `harbor_job_logger_sweeper_duration` | `1` | Job log cleanup interval (days) | Optional |


### Notification (MANDATORY ≥ Harbor 2.9)

| **Variable** | **Default Value** | **Description** | **Type** |
|-------------|------------------|-----------------|----------|
| `harbor_webhook_retry` | `10` | Maximum webhook retry attempts | Optional |
| `harbor_webhook_timeout` | `3` | Webhook HTTP timeout (seconds) | Optional |


### Registry (Non-storage Settings)

| **Variable** | **Default Value** | **Description** | **Type** |
|-------------|------------------|-----------------|----------|
| `harbor_registry_relativeurls` | `false` | Use relative URLs (behind proxy/LB) | Optional |
| `harbor_registry_cache_expire` | `168h` | Registry cache expiration duration | Optional |


### Trivy Scanner

| **Variable** | **Default Value** | **Description** | **Type** |
|-------------|------------------|-----------------|----------|
| `harbor_trivy_enabled` | `true` | Enable Trivy vulnerability scanning | Optional |
| `harbor_trivy_ignore_unfixed` | `false` | Ignore unfixed vulnerabilities | Optional |
| `harbor_trivy_severity` | `UNKNOWN,LOW,MEDIUM,HIGH,CRITICAL` | Severity levels to scan | Optional |
| `harbor_trivy_timeout` | `5m` | Trivy scan timeout | Optional |


### Metrics

| **Variable** | **Default Value** | **Description** | **Type** |
|-------------|------------------|-----------------|----------|
| `harbor_metrics_enabled` | `false` | Enable Harbor metrics | Optional |
| `harbor_metrics_port` | `9090` | Metrics service port | Optional |


### Upload Purging

| **Variable** | **Default Value** | **Description** | **Type** |
|-------------|------------------|-----------------|----------|
| `harbor_upload_purge_enabled` | `true` | Enable upload purging | Optional |
| `harbor_upload_purge_age` | `168h` | Age threshold for purging uploads | Optional |
| `harbor_upload_purge_interval` | `24h` | Purge execution interval | Optional |
| `harbor_upload_purge_dryrun` | `false` | Dry-run mode for purge | Optional |


### Proxy Configuration

| **Variable** | **Default Value** | **Description** | **Type** |
|-------------|------------------|-----------------|----------|
| `harbor_http_proxy` | — | HTTP proxy URL | Optional |
| `harbor_https_proxy` | — | HTTPS proxy URL | Optional |
| `harbor_no_proxy` | `127.0.0.1,localhost,.local` | No-proxy list | Optional |

### Internal TLS (Optional ≥ Harbor 2.9)

| **Variable** | **Default Value** | **Description** | **Type** |
|-------------|------------------|-----------------|----------|
| `harbor_internal_tls_enabled` | `false` | Enable internal TLS between Harbor components | Optional |
| `harbor_internal_tls_dir` | `/etc/harbor/tls/internal` | Internal TLS certificate directory | Optional |

### Harbor Version

| **Variable** | **Default Value** | **Description** | **Type** |
|-------------|------------------|-----------------|----------|
| `harbor_version` | `2.14.0` | Harbor version marker (DO NOT REMOVE) | Mandatory |



## Minimal Example (Consumer Project)

The following example shows a **minimal yet production-ready Harbor configuration**  
using **external PostgreSQL and Redis**, suitable for a consumer or application team.


### `group_vars/harbor/harbor.yml`

```yaml
# Core
harbor_hostname: harbor.example.com
harbor_data_dir: /data/harbor

# HTTPS / TLS
harbor_https_enabled: true
harbor_https_port: 443
harbor_ssl_cert_path: /etc/harbor/tls/tls.crt
harbor_ssl_key_path: /etc/harbor/tls/tls.key

# External PostgreSQL
harbor_db_external: true
harbor_db_host: postgres.internal.example.com
harbor_db_user: harbor
harbor_db_port: 5432
harbor_db_name: harbor_db
harbor_db_sslmode: disable

# External Redis
harbor_redis_external: true
harbor_redis_host: redis.internal.example.com
harbor_redis_port: 6379
harbor_redis_db: 0

# Registry Storage (Filesystem – minimal setup)
harbor_storage_type: filesystem
harbor_registry_storage_path: /data/harbor/registry

# Logging
harbor_log_level: info

# Trivy vulnerability scanning
harbor_trivy_enabled: true
harbor_trivy_ignore_unfixed: false
harbor_trivy_severity: "UNKNOWN,LOW,MEDIUM,HIGH,CRITICAL"

# Metrics (optional, disabled by default)
harbor_metrics_enabled: true

```

### `group_vars/harbor/vault.yml`
```yaml
vault_harbor_admin_password: "admin123"
vault_harbor_db_password: "dbpass"
vault_harbor_redis_password: "redispass"
```


## Example Playbook

Use the following Ansible playbook to install and configure **Harbor Container Registry**
using the `harbor_registry` role.

### `harbor.yml`

```yaml
- name: Install and Configure Harbor Registry
  hosts: harbor
  become: true
  vars_files:
    - group_vars/harbor/harbor.yml
    - group_vars/harbor/vault.yml
  roles:
    - harbor_registry
```

## References

### Source Code
- Harbor Official Repository: https://github.com/goharbor/harbor
- Harbor Documentation: https://goharbor.io/docs/

### Guide Followed
- Harbor Installation Guide (Official)
- Harbor Configuration Reference (`harbor.yml`)
- Ansible Best Practices for Roles & Vault

## License

This project is licensed under one of the following (as applicable):

- **MIT License**
- **BSD License**

See the `LICENSE` file for more details.


## Author Information

- **Shashi Prabha**


