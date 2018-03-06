# ansible-postgresql-slave-init
Prepare a postgresql slave to join into a postgresql cluster

# example of a master-slave configuration
```
- name: postgresql master
  become: true
  become_method: sudo
  hosts: all
  roles:
    - role: entercloudsuite.postgres
      postgresql_global_config_options:
          - option: listen_addresses
            value: '*'
          - option: wal_level
            value: hot_standby
          - option: archive_mode
            value: on
          - option: archive_command
            value: 'cp -i %p /var/lib/postgresql/9.5/main/archive/%f'
          - option: max_wal_senders
            value: 3
          - option: wal_keep_segments
            value: 8
      postgresql_hba_entries:
          - { type: local, database: all, user: postgres, auth_method: peer }
          - { type: local, database: all, user: all, auth_method: peer }
          - { type: host, database: all, user: all, address: '127.0.0.1/32', auth_method: md5 }
          - { type: host, database: all, user: all, address: '::1/128', auth_method: md5 }
          - { type: host, database: replication, user: replica, address: '10.2.0.0/16', auth_method: md5 }
      postgresql_users:
          - name: replica
            password: yoursupersecretpassword
            role_attr_flags: REPLICATION

- name: prepare postgresql slave
  hosts: all
  become: true
  become_method: sudo
  become_user: root
  roles:
    - role: jsecchiero.postgresql-slave-init
      postgresql_master: postgres.service.automium.consul
      postgresql_replica_user: replica
      postgresql_replica_password: yoursupersecretpassword

- name: postgresql slave
  hosts: all
  become: true
  become_method: sudo
  roles:
    - role: entercloudsuite.postgres
      postgresql_global_config_options:
          - option: listen_addresses
            value: '*'
          - option: wal_level
            value: hot_standby
          - option: archive_mode
            value: on
          - option: archive_command
            value: 'cp -i %p /var/lib/postgresql/9.5/main/archive/%f'
          - option: max_wal_senders
            value: 3
          - option: wal_keep_segments
            value: 8
          - option: hot_standby
            value: 'on'
      postgresql_hba_entries:
          - { type: local, database: all, user: postgres, auth_method: peer }
          - { type: local, database: all, user: all, auth_method: peer }
          - { type: host, database: all, user: all, address: '127.0.0.1/32', auth_method: md5 }
          - { type: host, database: all, user: all, address: '::1/128', auth_method: md5 }
          - { type: host, database: replication, user: replica, address: '10.2.0.0/16', auth_method: md5 }
```
