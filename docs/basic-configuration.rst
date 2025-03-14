.. _service-mariadb-basic-configuration:

Basic Configuration
###################

Creating A Database
*******************

.. code-block:: mariadb
  :caption: Create new DB user and database.

  mariadb -u root -p

  CREATE USER IF NOT EXISTS '{USER}'@'{DOMAIN}' IDENTIFIED BY '{PASS}';
  CREATE DATABASE IF NOT EXISTS {DB};

  GRANT ALL PRIVILEGES ON {DB}.* TO '{USER}'@'{DOMAIN}';
  FLUSH PRIVILEGES;

Import A Database
*****************

.. code-block:: mariadb
  :caption: Import DB and set appropriate DB permissions.

  mariadb -u {USER} -p {DATABASE} < database-dump.sql
  ALTER DATABASE {DB USER} OWNER TO {DB USER};
  GRANT ALL PRIVILEGES ON DATABASE {DB USER} TO {DB USER};

Database Backup
***************

Backup Entire Instance
======================
This will dump all databases, users and permissions. Remember to pull the data
from the instance or the data directory.

.. code-block:: bash

  mariadb-dump --user=root --password --lock-tables --all-databases > {DATABASES}.sql

Backup A Specific Database
==========================
Backup a specific database. Permissions will need to be restored with database.

.. code-block:: bash

  mariadb -u root -p {DATABASE} > {DATABASE}.sql
