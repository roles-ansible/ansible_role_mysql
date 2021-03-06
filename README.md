[![Ansible Galaxy](https://raw.githubusercontent.com/roles-ansible/ansible_role_mysql/main/.github/galaxy.svg?sanitize=true)](https://galaxy.ansible.com/do1jlr/mysql) [![MIT License](https://raw.githubusercontent.com/roles-ansible/ansible_role_mysql/main/.github/license.svg?sanitize=true)](https://github.com/roles-ansible/ansible_role_mysql/blob/main/LICENSE)

 ansible role MySQL
===================

Ansible role to install and configure a MySQL server for RHEL/CentOS, Debian/Ubuntu and Archlinux.

## Requirements

+ No special requirements
+ note that this role requires root access. *(``become: true``)*

## Role Variables

Available variables are listed below, along with default values *(see [defaults/main.yml](https://github.com/roles-ansible/ansible_role_mysql/blob/main/defaults/main.yml))*:

```yaml
mysql_user_home: /root
mysql_user_name: root
mysql_user_password: root
```
The home directory inside which Python MySQL settings will be stored, which Ansible will use when connecting to MySQL. This should be the home directory of the user which runs this Ansible role. The `mysql_user_name` and `mysql_user_password` can be set if you are running this role under a non-root user account and want to set a non-root user.

```yaml
mysql_root_home: /root
mysql_root_username: root
mysql_root_password: root
```
The MySQL root user account details.

```yaml
mysql_root_password_update: false
```
Whether to force update the MySQL root user's password. By default, this role will only change the root user's password when MySQL is first configured. You can force an update by setting this to `yes`.

> Note: If you get an error like `ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)` after a failed or interrupted playbook run, this usually means the root password wasn't originally updated to begin with. Try either removing  the `.my.cnf` file inside the configured `mysql_user_home` or updating it and setting `password=''` (the insecure default password). Run the playbook again, with `mysql_root_password_update` set to `yes`, and the setup should complete.

> Note: If you get an error like `ERROR 1698 (28000): Access denied for user 'root'@'localhost' (using password: YES)` when trying to log in from the CLI you might need to run as root or sudoer.

```yaml
mysql_enabled_on_startup: true
```
Whether MySQL should be enabled on startup.

```yaml
mysql_config_file: *default value depends on OS*
mysql_config_include_dir: *default value depends on OS*
```
The main my.cnf configuration file and include directory.

```yaml
overwrite_global_mycnf: true
```
Whether the global my.cnf should be overwritten each time this role is run. Setting this to `no` tells Ansible to only create the `my.cnf` file if it doesn't exist. This should be left at its default value (`yes`) if you'd like to use this role's variables to configure MySQL.

```yaml
mysql_config_include_files: []
```
A list of files that should override the default global my.cnf. Each item in the array requires a "src" parameter which is a path to a file. An optional "force" parameter can force the file to be updated each time ansible runs.

```yaml
mysql_databases: []
```
The MySQL databases to create. A database has the values `name`, `encoding` (defaults to `utf8`), `collation` (defaults to `utf8_general_ci`) and `replicate` (defaults to `1`, only used if replication is configured). The formats of these are the same as in the `mysql_db` module.

You can also delete a database (or ensure it's not on the server) by setting `state` to `absent` (defaults to `present`).

```yaml
mysql_users: []
```
The MySQL users and their privileges. A user has the values:

  - `name`
  - `host` (defaults to `localhost`)
  - `password` (can be plaintext or encrypted???if encrypted, set `encrypted: yes`)
  - `encrypted` (defaults to `no`)
  - `priv` (defaults to `*.*:USAGE`)
  - `append_privs` (defaults to `no`)
  - `state`  (defaults to `present`)

The formats of these are the same as in the `mysql_user` module.

```yaml
mysql_packages:
  - mysql
  - mysql-server
```
(OS-specific, RedHat/CentOS defaults listed here) Packages to be installed. In some situations, you may need to add additional packages, like `mysql-devel`.

```yaml
mysql_enablerepo: ""
```
(RedHat/CentOS only) If you have enabled any additional repositories (might I suggest geerlingguy.repo-epel or geerlingguy.repo-remi), those repositories can be listed under this variable (e.g. `remi,epel`). This can be handy, as an example, if you want to install later versions of MySQL.

```yaml
mysql_python_package_debian: python3-mysqldb
```
(Ubuntu/Debian only) If you need to explicitly override the MySQL Python package, you can set it here. Set this to `python-mysqldb` if using older distributions running Python 2.

```yaml
mysql_port: "3306"
mysql_bind_address: '0.0.0.0'
mysql_datadir: /var/lib/mysql
mysql_socket: *default value depends on OS*
mysql_pid_file: *default value depends on OS*
```
Default MySQL connection configuration.

```yaml
mysql_log_file_group: mysql *adm on Debian*
mysql_log: ""
mysql_log_error: *default value depends on OS*
mysql_syslog_tag: *default value depends on OS*
```
MySQL logging configuration. Setting `mysql_log` (the general query log) or `mysql_log_error` to `syslog` will make MySQL log to syslog using the `mysql_syslog_tag`.

```yaml
mysql_slow_query_log_enabled: false
mysql_slow_query_log_file: *default value depends on OS*
mysql_slow_query_time: 2
```
Slow query log settings. Note that the log file will be created by this role, but if you're running on a server with SELinux or AppArmor, you may need to add this path to the allowed paths for MySQL, or disable the mysql profile. For example, on Debian/Ubuntu, you can run `sudo ln -s /etc/apparmor.d/usr.sbin.mysqld /etc/apparmor.d/disable/usr.sbin.mysqld && sudo service apparmor restart`.

```yaml
mysql_key_buffer_size: "256M"
mysql_max_allowed_packet: "64M"
mysql_table_open_cache: "256"
[...]
```
The rest of the settings in `defaults/main.yml` control MySQL's memory usage and some other common settings. The default values are tuned for a server where MySQL can consume ~512 MB RAM, so you should consider adjusting them to suit your particular server better.

```yaml
mysql_server_id: "1"
mysql_max_binlog_size: "100M"
mysql_binlog_format: "ROW"
mysql_expire_logs_days: "10"
mysql_replication_role: ''
mysql_replication_master: ''
mysql_replication_user: {}
```
Replication settings. Set `mysql_server_id` and `mysql_replication_role` by server (e.g. the master would be ID `1`, with the `mysql_replication_role` of `master`, and the slave would be ID `2`, with the `mysql_replication_role` of `slave`). The `mysql_replication_user` uses the same keys as individual list items in `mysql_users`, and is created on master servers, and used to replicate on all the slaves.

`mysql_replication_master` needs to resolve to an IP or a hostname which is accessable to the Slaves (this could be a `/etc/hosts` injection or some other means), otherwise the slaves cannot communicate to the master.

### Later versions of MySQL on CentOS 7

If you want to install MySQL from the official repository instead of installing the system default MariaDB equivalents, you can add the following `pre_tasks` task in your playbook:

```yaml
  pre_tasks:
    - name: Install the MySQL repo.
      yum:
        name: http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
        state: present
      when: ansible_os_family == "RedHat"

    - name: Override variables for MySQL (RedHat).
      set_fact:
        mysql_daemon: mysqld
        mysql_packages: ['mysql-server']
        mysql_log_error: /var/log/mysqld.err
        mysql_syslog_tag: mysqld
        mysql_pid_file: /var/run/mysqld/mysqld.pid
        mysql_socket: /var/lib/mysql/mysql.sock
      when: ansible_os_family == "RedHat"
```

### MariaDB usage

This role works with either MySQL or a compatible version of MariaDB. On RHEL/CentOS 7+, the mariadb database engine was substituted as the default MySQL replacement package. No modifications are necessary though all of the variables still reference 'mysql' instead of mariadb.

#### Ubuntu 14.04 and 16.04 MariaDB configuration

On Ubuntu, the package names are named differently, so the `mysql_package` variable needs to be altered. Set the following variables (at a minimum):

```yaml
mysql_packages:
  - mariadb-client
  - mariadb-server
  - python-mysqldb
```

#### Optional Versionscheck

You can enable a optional versionscheck, that will prevent you from running a older version of this role at your systems by setting ``submodules_versioncheck`` to ``true``.

## Dependencies

None.

## Example Playbook

```yaml
- name: install mysql database to database servers
  hosts: db-servers
  vars_files:
    - vars/main.yml
  roles:
   - {role: do1jlr.mysql, tags: [mysql]}
```

*Inside `vars/main.yml`*:

```yaml
mysql_root_password: super-secure-password
_awesome_password_variable: awesome_secure_password
mysql_databases:
  - name: example_db
    encoding: latin1
    collation: latin1_general_ci
  - name: other_db
    encoding: utf8mb4
    collation: utf8mb4_general_ci
mysql_users:
  - name: example_user
    host: "%"
    password: similarly-secure-password
    priv: "example_db.*:ALL"
  - name: other_user
    host: 127.0.0.1
    password: "{{ _awesome_password_variable }}"
    priv: "other_db.*:ALL"
```

*Please note, that you can put a variable like ``_awesome_password_variable`` in your [ansible vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html).*

## License

MIT / BSD

## Author Information

+ This role was created in 2014 by [Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/).
+ This role was forked 2021 by [L3D](https://chaos.social/@l3d) and upgraded to the current version.
