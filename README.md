# [MariaDB][h]
Manage Maria databases.

## [Requirements][i]
Requires [r_pufky.srv][g] galaxy-ng collection. See
[additional documentation][m] and [reference documentation][h] for
troubleshooting and config variables.

Install size: ~60MB

> Tasks [potentially touching Network Mounted Filesystems][o] will be run as
> the task user and fallback to the service user. Manage these locations
> externally if these fail.

## Role Variables
Detailed variable use documented in defaults. See usage for role operation.

* [defaults][j] - User configurable options.

* [ports][k] - Ports are **not** managed (defined for external use).

## Usage
MariaDB managed with **root** via unix socket connections. Connection
information is written to **/root/.my.cnf**.

mariadb-secure-installation is always run to secure installation:

* Force local root password.
* Delete remote root users.
* Deletes all anonymous users.
* Delete test database & privileges.
* Flushes privileges.

Path                      | Usage
--------------------------|-------
/etc/mysql/mariadb.cnf    | maria_srv_cfg always deployed here.
/etc/mysql/mariadb.conf.d | maria_srv_conf_d always deployed here.
/etc/mysql/secure.conf.d  | maria_srv_secure_conf_d always deployed here.
/var/lib/mysql            | Default database location.

### Feature Flags
Tasks are gated by feature flags and executed in the following order.

  Step | Flag              | Notes
 ------|-------------------|-------
  1    | maria_flg_install | Install required packages, users, etc.
  2    | maria_flg_config  | Install user-defined config.
  3    | maria_flg_backup  | Create scheduled backups.

### Example Playbooks
All options can be used simultaneously.

#### New deployment
MariaDB will be ready to use, allowing logins with **test** which has all
privileges.

``` yaml
- name: 'Install MariaDB, enable scheduled backups.'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.maria'
  vars:
    maria_srv_password: '{{ vault_maria_root_password }}'
    # Test user created with all privs.
    maria_cfg_users:
      - name: 'test'
        host: '%'
        priv: '*.*:ALL'
        password: '{{ vault_db_test_password }}'
    maria_flg_backup: true
    # New database brand_new_db is created if non-existent.
    maria_cfg_dbs:
      - name: 'brand_new_db'
        collation: 'utf8_unicode_520_ci'
        encoding: 'utf8'
        state: 'present'
```

#### Create/Import Databases
Databases can be created automatically, optionally importing data from a SQL
backup if provided. Import is idempotent -- database is only imported when the
database is created. Re-import will require manual database deletion.

Use community.mysql.mysql_db parameters as dictionary keys with default values
matching specified parameters unless noted. Additional attributes are fully
defined. See [documentation][j].

Use community.mysql.mysql_user parameters as dictionary keys with default
values matching specified parameters unless noted. See [documentation][j].

``` yaml
- name: 'Create new MariaDB, creating test user.'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.maria'
  vars:
    maria_srv_password: '{{ vault_maria_root_password }}'
    maria_cfg_users:
      - name: 'test'
        host: '%'
        priv: '*.*:ALL'
        password: '{{ vault_db_test_password }}'
      - name: 'remove_this_user'
        host: '%'
        state: 'absent'
    maria_cfg_dbs:
      # Create 'empty' database.
      - name: 'empty'
        collation: 'utf8_unicode_520_ci'
        encoding: 'utf8'
        state: 'present'
      # Creates 'skeleton' database and import existing data.
      - name: 'skeleton'
        collation: 'utf8_unicode_520_ci'
        encoding: 'utf8'
        state: 'present'
        extensions:
          db_import:
            enable: true
            source: 'host_vars/maria.example.com/data/skeleton.sql'
```

#### Instance Configuration
Configure the MariaDB instance. mariadb.cnf, conf.d, secure.conf.d are all
sourced from the ansible controller and templated. Place private material in
secure.conf.d, which will automatically be locked down.

MariaDB configuration is complex and nuanced. See [sytem variables][p] keeping
in mind where the [role places files](#usage).

Example minimum configuration to enable at-rest data encryption. Additional
settings are required per database.

Example 50-server.cnf
``` ini
# Example setting up encrypted instance.
[mariadbd]
basedir=/usr
bind_address=127.0.0.1
expire_logs_days=10
file_key_management_encryption_algorithm=AES_CTR
file_key_management_filekey=FILE:/etc/mysql/secure.conf.d/keyfile.key
file_key_management_filename=/etc/mysql/secure.conf.d/keyfile.enc
pid_file=/run/mysqld/mysqld.pid
```

Example mariadb.cnf
``` ini
[client-server]
socket=/run/mysqld/mysqld.sock
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mariadb.conf.d/
!includedir /etc/mysql/secure.conf.d/

[mariadb]
plugin_load_add = file_key_management
aria-encrypt-tables
encrypt-binlog
encrypt-tmp-disk-tables
encrypt-tmp-files
loose-innodb-encrypt-log
loose-innodb-encrypt-tables
```

``` yaml
- name: 'Create new MariaDB, creating test user.'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.maria'
  vars:
    maria_srv_password: '{{ vault_maria_root_password }}'
    maria_srv_cfg: 'host_vars/maria.example.com/maria.cnf'
    maria_srv_conf_d: 'host_vars/maria.example.com/mariadb.conf.d'
    maria_srv_secure_conf_d: 'host_vars/maria.example.com/secure.conf.d'
    maria_cfg_users:
      - name: 'test1'
        host: '%'
        priv: '*.*:ALL'
        password: '{{ vault_db_test_password }}'
    maria_cfg_dbs:
      - name: 'skeleton'
        collation: 'utf8_unicode_520_ci'
        encoding: 'utf8'
        state: 'present'
        extensions:
          db_import:
            enable: true
            source: 'host_vars/db.example.com/files/skeleton_default.sql'
```

## Development
Configure [environment][a].

``` bash
# Run all tests.
molecule test --all
```

Testing variables:

  Variable            | type | Description
 ---------------------|------|-------------
  molecule_flg_inject | bool | Flag to inject files locally.

### [Releases][b]

  Release | Debian | Ansible | MariaDB | Notes
 ---------|--------|---------|---------|-------
  3.x.x   | 13     | 2.18    | v12.2   | Ansible 2.20, feature flags, and semantic versioning.
  2.x.x   | 12     | 2.18    | v11.x   | Migrate to Debian Trixie.
  1.x.x   | 12     | 2.18    | v11.x   | Implement data annotations.
  0.x.x   | 12     | 2.18    | v11.x   | Initial release.

## Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License][c] | [direct link][f]

## Author Information
PGP: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9][d] | [github gist][e]


[a]: https://r-pufky.github.io/ansible_docs
[b]: https://semver.org/spec/v2.0.0
[c]: https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0
[d]: https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9
[e]: https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22

[f]: https://github.com/r-pufky/ansible_maria/blob/main/LICENSE
[g]: https://github.com/r-pufky/ansible_collection_srv
[h]: https://mariadb.com/docs
[i]: https://github.com/r-pufky/ansible_maria/blob/main/meta/main.yml
[j]: https://github.com/r-pufky/ansible_maria/tree/main/defaults/main/main.yml
[k]: https://github.com/r-pufky/ansible_maria/blob/main/defaults/main/ports.yml
[m]: http://r-pufky.github.io/docs/service/mariadb
[o]: https://r-pufky.github.io/ansible_docs/best_practice/patterns/#network-mounts
[p]: https://mariadb.com/docs/server/reference/full-list-of-mariadb-options-system-and-status-variables