# Ansible Role: EDB

[![Build Status](https://travis-ci.org/geerlingguy/ansible-role-edb.svg?branch=master)](https://travis-ci.org/geerlingguy/ansible-role-edb)

Installs and configures EDB server on RHEL/CentOS or Debian/Ubuntu servers.

## Requirements

In order to install the EDB PostgreSQL Advanced Server (EPAS) on the RedHat family of distros, you will need the EPEL to satisfy the package requirements.

You will also need EDB repository credentials. See the example below for how to include those creds, or take a look at molecule/default/converge.yml

Note that this role requires root access, so either run it in a playbook with a global `become: yes`, or invoke the role in your playbook like:

    - hosts: database
      tasks:
        - include_vars:
            file: repo-creds.yml
        - import_role:
            name: terradatum.edb
          become: yes

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

    edb_repo_username: ""
    edb_repo_password: ""

You must set these credentials in order to download the EDB packages. The recommendation is passing them in through the command line, or add them to your vault.

    edb_restarted_state: "restarted"

Set the state of the service when configuration changes are made. Recommended values are `restarted` or `reloaded`.

    edb_python_library: python-psycopg2

Library used by Ansible to communicate with EDB. If you are using Python 3 (e.g. set via `ansible_python_interpreter`), you should change this to `python3-psycopg2`.

    edb_user: enterprisedb
    edb_group: enterprisedb

The user and group under which EDB will run.

    edb_unix_socket_directories:
      - /var/run/edb-as

The directories (usually one, but can be multiple) where EDB's socket will be created.

    edb_service_state: started
    edb_service_enabled: true

Control the state of the edb service and whether it should start at boot time.

    edb_global_config_options:
      - option: unix_socket_directories
        value: '{{ edb_unix_socket_directories | join(",") }}'

Global configuration options that will be set in `postgresql.conf`. Note that for RHEL/CentOS 6 (or very old versions of EDB), you need to at least override this variable and set the `option` to `unix_socket_directory`.

    edb_hba_entries:
      - { type: local, database: all, user: enterprisedb, auth_method: peer }
      - { type: local, database: all, user: all, auth_method: peer }
      - { type: host, database: all, user: all, address: '127.0.0.1/32', auth_method: md5 }
      - { type: host, database: all, user: all, address: '::1/128', auth_method: md5 }

Configure [host based authentication](https://www.edb.org/docs/current/static/auth-pg-hba-conf.html) entries to be set in the `pg_hba.conf`. Options for entries include:

  - `type` (required)
  - `database` (required)
  - `user` (required)
  - `address` (one of this or the following two are required)
  - `ip_address`
  - `ip_mask`
  - `auth_method` (required)
  - `auth_options` (optional)

If overriding, make sure you copy all of the existing entries from `defaults/main.yml` if you need to preserve existing entries.

    edb_locales:
      - 'en_US.UTF-8'

(Debian/Ubuntu only) Used to generate the locales used by EDB databases.

    edb_databases:
      - name: exampledb # required; the rest are optional
        lc_collate: # defaults to 'en_US.UTF-8'
        lc_ctype: # defaults to 'en_US.UTF-8'
        encoding: # defaults to 'UTF-8'
        template: # defaults to 'template0'
        login_host: # defaults to 'localhost'
        login_password: # defaults to not set
        login_user: # defaults to 'edb_user'
        login_unix_socket: # defaults to 1st of edb_unix_socket_directories
        port: # defaults to 5444
        owner: # defaults to edb_user
        state: # defaults to 'present'

A list of databases to ensure exist on the server. Only the `name` is required; all other properties are optional.

    edb_users:
      - name: jdoe #required; the rest are optional
        password: # defaults to not set
        encrypted: # defaults to not set
        priv: # defaults to not set
        role_attr_flags: # defaults to not set
        db: # defaults to not set
        login_host: # defaults to 'localhost'
        login_password: # defaults to not set
        login_user: # defaults to '{{ edb_user }}'
        login_unix_socket: # defaults to 1st of edb_unix_socket_directories
        port: # defaults to 5444
        state: # defaults to 'present'

A list of users to ensure exist on the server. Only the `name` is required; all other properties are optional.

    edb_users_no_log: true

Whether to output user data (which may contain sensitive information, like passwords) when managing users.

    edb_version: [OS-specific]
    edb_data_dir: [OS-specific]
    edb_bin_path: [OS-specific]
    edb_config_path: [OS-specific]
    edb_daemon: [OS-specific]
    edb_packages: [OS-specific]

OS-specific variables that are set by include files in this role's `vars` directory. These shouldn't be overridden unless you're using a version of EDB that wasn't installed using system packages.

## Dependencies

When installing EDB PostgreSQL Advanced Server on RHEL/CentOS, you must have the EPEL repo installed and enabled.

## Example Playbook

    - hosts: database
      become: yes
      vars_files:
        - vars/main.yml
      roles:
        - geerlingguy.repo-epel
        - terradatum.edb

*Inside `vars/main.yml`*:

    edb_databases:
      - name: example_db
    edb_users:
      - name: example_user
        password: supersecure

## License

MIT / BSD

## Author Information

This role was created in 2016 by [Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/), and subsequently forked by [G. Richard Bellamy](https://github.com/rbellamy) for use by [Terradatum](https://terradatum.com).
