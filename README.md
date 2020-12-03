# Ansible Role: Web3Signer 

### Description
Ansible role that will install, configure and runs [Web3Signer](https://docs.web3signer.consensys.net/): an open-source signing service

### Table of Contents
  - [Supported Platforms](#supported-platforms)
  - [Dependencies](#dependencies)
  - [Role Variables](#role-variables)
  - [Configure Signing Keys](#configure-signing-keys)
  - [Example Playbook](#example-playbook)
  - [License](#license)
  - [Author Information](#author-information)

### Supported Platforms
```
* Debian
* Ubuntu
* Redhat(CentOS/Fedora)
* Amazon
```

### Dependencies

* JDK 11 or greater
* PostgreSQL database if slashing protection enabled for eth2


### Role Variables:

All variables which can be overridden are stored in [defaults/main.yml](defaults/main.yml) file. 
By and large these variables are configuration options. 
Please refer to the Web3Signer [docs](https://docs.web3signer.consensys.net/en/latest/Reference/CLI/CLI-Syntax/) for more information

| Name                             | Default Value | Description                        |
| -------------------------------- | ------------- | -----------------------------------|
| web3signer_version               | `develop`      | Version to install                 |
| web3signer_user                  | `web3signer`    | OS user to create and run web3signer |
| web3signer_group                 | `web3signer`    | OS group |
| web3signer_app_home              | `/opt/web3signer` | App install location |
| web3signer_config_path           | `/etc/web3signer` | Config file location |
| web3signer_log_path              | `/var/log/web3signer` | Log file location |
| web3signer_log_filename          | `web3signer.log` | Log file name. Location will be web3signer_log_path |
| web3signer_data_home             | `/data/web3signer` | Location to store any persistent data. Keys stored in a sub folder keys |
| web3signer_db_verify_connection  | `True`           | Enables checking database available |
| web3signer_db_host               |                | PostgreSQL database instance host |
| web3signer_db_name               |                | PostgreSQL database name |
| web3signer_db_username           |                | PostgreSQL database user |
| web3signer_db_password           |                | PostgreSQL database user password |
| web3signer_flyway_version        | `7.3.0`          | Flyway CLI version to download for flyway migration |
| web3signer_service_name          | `web3signer`     | Systemd service name |
| web3signer_config_filename       | `web3signer.yml` | Configuration file name. Location will be `web3signer_config_path`|
| web3signer_command               | `eth2`           | Web3signer command. Supported `eth1`,`eth2`,`filecoin` |
| web3signer_keys                  | `[]`               | Specify keys to configure. Please see notes below for more details|
| web3signer_logging               | `INFO`           | Logging verbosity levels: OFF, FATAL, WARN, INFO,DEBUG, TRACE, ALL|
| web3signer_http_listen_host      | `127.0.0.1`       | Host for HTTP to listen on |
| web3signer_http_listen_port      | `9000`            | Port for HTTP to listen on |
| web3signer_http_host_allowlist   | `127.0.0.1`        | Comma separated list of hostnames to allow for http access, or * to accept any host |
| web3signer_metrics_enabled       | `False`           | Set to start the metrics exporter |
| web3signer_metrics_host          | `127.0.0.1`       | Host for the metrics exporter to listen on |
| web3signer_metrics_port          | `9001`            | Port for the metrics exporter to listen on |
| web3signer_metrics_categories    | `[HTTP,SIGNING,FILECOIN,JVM,PROCESS,ETH2_SLASHING_PROTECTION]` | Comma separated list of categories to track metrics for |
| web3signer_host_allowlist        | `127.0.0.1`       | Comma separated list of hostnames to allow for metrics access, or * to accept any host |
| web3signer_idle_connection_timeout_seconds | `30`    | Number of seconds after which an idle connection will be terminated |
| web3signer_swagger_ui_enabled    | `False`           | Enable swagger UI |
| web3signer_tls_keystore_file     |                   | Path to a PKCS#12 formatted keystore used to enable TLS on inbound connections |
| web3signer_tls_keystore_password_file |              | Path to a file containing the password used to decrypt the keystore |
| web3signer_tls_allow_any_client  |                   | If defined, any client may connect, regardless of presented certificate. This cannot be set if either a whitelist or CA clients have been enabled |
| web3signer_tls_known_client_file |                   | Path to a file containing the fingerprints of authorized clients |
| web3signer_tls_allow_ca_clients  |                   | If defined, allows clients authorized by the CA to connect to EthSigner |
                               

### Configure Signing Keys:

Signing keys to be configured on the filesystem must be provided via parameter `web3signer_keys` as a map. Key of the map will be used as the name
of the key file. Each key creates a key file. Currently support unencrypted keys and support other type of keys will be added soon.
```yaml
web3signer_keys:
  key1:
    type: 'file-raw'
    keyType: 'BLS'
    privateKey: '0x6eeb32dd0fe010051825e3ef402b1a7c66fd6daa9c61eb351c5d760684de8e6a'
```

### Example Playbook:

Example playbook to install PostgreSQL, Java and Web3Signer on a single VM

```yaml
- name: Web3Signer Installation
  hosts: web3signer
  remote_user: vagrant
  vars:
    web3signer_http_listen_host: '0.0.0.0'
    web3signer_db_host: 'localhost'
    web3signer_db_name: 'web3signer'
    web3signer_db_username: 'web3signer'
    web3signer_db_password: 'somepassword'
    web3signer_keys:
      key1:
        type: 'file-raw'
        keyType: 'BLS'
        privateKey: '0x6eeb32dd0fe010051825e3ef402b1a7c66fd6daa9c61eb351c5d760684de8e6a'
    postgresql_hba_entries:
      - { type: local, database: all, user: postgres, auth_method: trust }
      - { type: host, database: web3signer, user: web3signer, address: localhost, auth_method: password }
    postgresql_users:
      - name: 'web3signer'
        password: 'somepassword'
    postgresql_databases:
      - name: 'web3signer'

  roles:
    - role: geerlingguy.postgresql
      become: True
    - role: lean_delivery.java
      become: True
    - role: consensys.web3signer
```

### License

Apache


### Author Information

Consensys, 2020