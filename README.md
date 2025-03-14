# MariaDB
Manage Maria databases.

## Requirements
[supported platforms](https://github.com/r-pufky/ansible_maria/blob/main/meta/main.yml)

[collections/roles](https://github.com/r-pufky/ansible_maria/blob/main/meta/requirements.yml)

## Role Variables
[defaults](https://github.com/r-pufky/ansible_maria/tree/main/defaults/main)

### Ports
All ports and protocols have been defined for the role.

[defaults/ports.yml](https://github.com/r-pufky/ansible_maria/blob/main/defaults/main/ports.yml)

## Dependencies
Part of the [r_pufky.srv](https://github.com/r-pufky/ansible_collection_srv)
collection.

## Example Playbook
Read defaults documentation. All options are supported.
maria-secure-installation is always run.

### Setup MariaDB, create user, create database, enable backups.
host_vars/db.example.com/vars/maria.yml
``` yaml
maria_service_password: '{{ vault_local_root_password }}'
maria_databases_backups_enable: true
maria_users:
  - name: 'test1'
    host: '%'
    priv: '*.*:ALL'
    password: '{{ vault_db_test_password }}'
maria_databases:
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
     - 'r_pufky.db/maria'
```

### Setup MariaDB and import database on database creation.
Import is idempotent -- database is only imported when the database is created.
Re-import will require manual database deletion.

host_vars/db.example.com/vars/maria.yml
``` yaml
maria_service_password: '{{ vault_local_root_password }}'
maria_users:
  - name: 'test1'
    host: '%'
    priv: '*.*:ALL'
    password: '{{ vault_db_test_password }}'
maria_databases:
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
     - 'r_pufky.db/maria'
```

### Enable Encryption
Example minimum configuration to enable at-rest data encryption. Additional
settings are required per database. See `molecule/encryption/molecule.yml` for
additional examples.

host_vars/db.example.com/vars/maria.yml
``` yaml
maria_service_password: '{{ vault_local_root_password }}'
maria_encryption_enable: true
maria_encryption_path: 'host_vars/db.example.com/files/encryption'
maria_server_mariadb:
  - version: 'all'
    config:
      pid_file: '/run/mysqld/mysqld.pid'
      basedir: '/usr'
      bind_address: '127.0.0.1'
      expire_logs_days: 10
      file_key_management_filename: '/etc/mysql/mariadb.conf.d/encryption/keyfile.enc'
      file_key_management_filekey: 'FILE:/etc/mysql/mariadb.conf.d/encryption/keyfile.key'
      file_key_management_encryption_algorithm: 'AES_CTR'
maria_users:
  - name: 'test1'
    host: '%'
    priv: '*.*:ALL'
    password: '{{ vault_db_test_password }}'
maria_databases:
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
     - 'r_pufky.db/maria'
```

## Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License](https://github.com/r-pufky/ansible_maria/blob/main/LICENSE)

## Author Information
https://keybase.io/rpufky
