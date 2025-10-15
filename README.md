# MariaDB
Manage Maria databases.

## Requirements
[supported platforms](https://github.com/r-pufky/ansible_maria/blob/main/meta/main.yml)

## Role Variables
[defaults](https://github.com/r-pufky/ansible_maria/tree/main/defaults/main)

### Ports
All ports and protocols have been defined for the role.

[defaults/ports.yml](https://github.com/r-pufky/ansible_maria/blob/main/defaults/main/ports.yml)

## Dependencies
**galaxy-ng** roles cannot be used independently. Part of
[r_pufky.srv](https://github.com/r-pufky/ansible_collection_srv) collection.

## Example Playbook
Read defaults documentation. All options are supported.
maria-secure-installation is always run.

### Setup MariaDB, create user, create database, enable backups.
host_vars/db.example.com/vars/maria.yml
``` yaml
maria_srv_password: '{{ vault_local_root_password }}'
maria_cfg_databases_backups_enable: true
maria_cfg_users:
  - name: 'test1'
    host: '%'
    priv: '*.*:ALL'
    password: '{{ vault_db_test_password }}'
maria_cfg_databases:
  - name: 'skeleton'
    collation: 'utf8_unicode_520_ci'
    encoding: 'utf8'
    state: 'present'
```

site.yml
``` yaml
- name: 'db server'
  hosts: 'db.example.com'
  become: true
  roles:
     - 'r_pufky.srv.maria'
```

### Setup MariaDB and import database on database creation.
Import is idempotent -- database is only imported when the database is created.
Re-import will require manual database deletion.

host_vars/db.example.com/vars/maria.yml
``` yaml
maria_srv_password: '{{ vault_local_root_password }}'
maria_cfg_users:
  - name: 'test1'
    host: '%'
    priv: '*.*:ALL'
    password: '{{ vault_db_test_password }}'
maria_cfg_databases:
  - name: 'skeleton'
    collation: 'utf8_unicode_520_ci'
    encoding: 'utf8'
    state: 'present'
    extensions:
      db_import:
        enable: true
        source: 'host_vars/db.example.com/files/skeleton_default.sql'
```

site.yml
``` yaml
- name: 'db server'
  hosts: 'db.example.com'
  become: true
  roles:
     - 'r_pufky.srv.maria'
```

### Enable Encryption
Example minimum configuration to enable at-rest data encryption. Additional
settings are required per database. See `molecule/encryption/molecule.yml` for
additional examples.

host_vars/db.example.com/vars/maria.yml
``` yaml
maria_srv_password: '{{ vault_local_root_password }}'
maria_cfg_encryption_enable: true
maria_cfg_encryption_path: 'host_vars/db.example.com/files/encryption'
maria_cfg_server_mariadb:
  - version: 'all'
    config:
      pid_file: '/run/mysqld/mysqld.pid'
      basedir: '/usr'
      bind_address: '127.0.0.1'
      expire_logs_days: 10
      file_key_management_filename: '/etc/mysql/mariadb.conf.d/encryption/keyfile.enc'
      file_key_management_filekey: 'FILE:/etc/mysql/mariadb.conf.d/encryption/keyfile.key'
      file_key_management_encryption_algorithm: 'AES_CTR'
maria_cfg_users:
  - name: 'test1'
    host: '%'
    priv: '*.*:ALL'
    password: '{{ vault_db_test_password }}'
maria_cfg_databases:
  - name: 'skeleton'
    collation: 'utf8_unicode_520_ci'
    encoding: 'utf8'
    state: 'present'
    extensions:
      db_import:
        enable: true
        source: 'host_vars/db.example.com/files/skeleton_default.sql'
```

site.yml
``` yaml
- name: 'db server'
  hosts: 'db.example.com'
  become: true
  roles:
     - 'r_pufky.srv.maria'
```

## Development
Configure [environment](https://github.com/r-pufky/ansible_collection_docs/blob/main/ansible/environment.md)

Run all unit tests:
``` bash
molecule test --all
```

### Releases
Release format: **{OS}-{SERVICE}-{ROLE}**

Each type inherits the versioning system used; defaulting to schematic
versioning.

`12.0.0-2.0.3-1.0.0`

* 12.0.0 - Debian 12 (bookworm).
* 2.0.3 - Service/app version.
* 1.0.0 - Role version.

Releases are branched on Debian releases:

* **[13.x.x](https://github.com/r-pufky/ansible_maria)**: 13 Trixie.
* **[12.x.x](https://github.com/r-pufky/ansible_maria/tree/12.x)**: 12 Bookworm.

## Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License](https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0)
 [(direct link)](https://github.com/r-pufky/ansible_maria/blob/main/LICENSE)

## Author Information
PGP Fingerprint: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9](https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9)
| [github gist](https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22)
