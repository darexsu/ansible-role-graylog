# Ansible role Graylog
[![CI Molecule](https://github.com/darexsu/ansible-role-graylog/actions/workflows/ci.yml/badge.svg)](https://github.com/darexsu/ansible-role-graylog/actions/workflows/ci.yml)&emsp;![](https://img.shields.io/static/v1?label=idempotence&message=ok&color=success)&emsp;![Ansible Role](https://img.shields.io/ansible/role/d/58619?color=blue&label=downloads)


  - Role:
      - [platforms](#platforms)
      - [install](#install)
      - [requirements](#requirements)
      - [FAQ](#faq)
      - [merge behaviour](#merge-behaviour)      
  - Playbooks (merge version):
      - [install and configure: Graylog v4.3, Elasticsearch v7.10, MongoDB v4.4, FirewallD](#install-and-configure-graylog-elasticsearch-mongodb-firewalld-merge-version)
          - [install: Graylog](#install-graylog-merge-version)
          - [configure: Graylog](#configure-graylog-merge-version)
  - Playbooks (full version):
      - [install and configure: Graylog v4.3, Elasticsearch v7.10, MongoDB v4.4, FirewallD](#install-and-configure-graylog-elasticsearch-mongodb-firewalld-full-version)
          - [install: Graylog ](#install-graylog-full-version)
          - [configure: Graylog](#configure-graylog-full-version)

### Platforms

|  Testing         | repo: graylog      |
| :--------------: | :----------------: |
| Debian 11        |  graylog.com       |
| Debian 10        |  graylog.com       |
| Ubuntu 20.04     |  graylog.com       |
| Ubuntu 18.04     |  graylog.com       |
| Oracle Linux 8   |  graylog.com       |
| Rocky Linux 8    |  graylog.com       |
### Install

```
ansible-galaxy install darexsu.graylog --force
```
### Requirements

roles: [Elasticsearch](https://github.com/darexsu/ansible-role-elasticsearch), [MongoDB](https://github.com/darexsu/ansible-role-mongodb), [FirewallD](https://github.com/darexsu/ansible-role-firewalld) (will automatically be installed)

### FAQ

- Q: Playbooks (merge version)
  - A: Some users prefer that variables that are hashes (aka ‘dictionaries’ in Python terms) are merged. This setting is called ‘merge’. You can use both version - see [merge behaviour](#merge-behaviour). Don't turn on "hash_behaviour" in ansible.cfg

- Q: Easy way to deploy. Ready for use
  - A: Use playbook  - [install and configure: Graylog v4.3, Elasticsearch v7.10, MongoDB v4.4, FirewallD](#install-and-configure-graylog-elasticsearch-mongodb-firewalld-full-version)

### Merge behaviour

Replace or Merge dictionaries (with "hash_behaviour=replace" in ansible.cfg):
```
# Replace             # Merge
---                   ---
  vars:                 vars:
    dict:                 merge:
      a: "value"            dict: 
      b: "value"              a: "value" 
                              b: "value"

# How does merge work?:
Your vars [host_vars]  -->  default vars [current role] --> default vars [include role]
  
  dict:          dict:              dict:
    a: "1" -->     a: "1"    -->      a: "1"
                   b: "2"    -->      b: "2"
                                      c: "3"
    
```
##### Install and configure: Graylog, Elasticsearch, MongoDB, FirewallD (merge version)
```yaml
---
- hosts: all
  become: true

  vars:
    merge:
      # Graylog
      graylog:
        enabled: true
        version: "4.3"
        repo: "graylog"
        service:
          enabled: true
          state: "started"
      # Graylog -> install
      graylog_install:
        enabled: true
        packages: ["graylog-server", "graylog-integrations-plugins"]
      # Graylog -> config
      graylog_config:
        enabled: true
        file: "server.conf"
        src: "server_conf.j2"
        backup: true
        data:
          is_leader: "true"
          node_id_file: "/etc/graylog/server/node-id"
          password_secret: "RObxb6DltymSD5LQtUuRUEqJPHQW"  # Change me -> generate password_secret (pwgen -N 1 -s 28)
          root_username: "admin"
          root_password_sha2: "96e061a52a96166e6fec9013c3c219781bf6d511062250fa542241cb951b2c17"  # Change me -> generate sha256 (echo -n ${password_secret} | sha256sum)
          bin_dir: "/usr/share/graylog-server/bin"
          data_dir: "/var/lib/graylog-server"
          plugin_dir: "/usr/share/graylog-server/plugin"
          http_bind_address: "0.0.0.0:9000"
          http_publish_uri: "http://0.0.0.0:9000/"
          http_enable_gzip: "false"
          elasticsearch_hosts: "http://localhost:9200"
          rotation_strategy: "count"
          elasticsearch_max_docs_per_index: "20000000"
          elasticsearch_max_number_of_indices: "20"
          retention_strategy: "delete"
          elasticsearch_shards: "4"
          elasticsearch_replicas: "0"
          elasticsearch_index_prefix: "graylog"
          allow_leading_wildcard_searches: "false"
          allow_highlighting: "false"
          elasticsearch_analyzer: "standard"
          output_batch_size: "500"
          output_flush_interval: "1"
          output_fault_count_threshold: "5"
          output_fault_penalty_seconds: "30"
          processbuffer_processors: "5"
          outputbuffer_processors: "3"
          processor_wait_strategy: "blocking"
          ring_size: "65536"
          inputbuffer_ring_size: "65536"
          inputbuffer_processors: "2"
          inputbuffer_wait_strategy: "blocking"
          message_journal_enabled: "true"
          message_journal_dir: "/var/lib/graylog-server/journal"
          lb_recognition_period_seconds: "3"
          mongodb_uri: "mongodb://localhost/graylog"
          mongodb_max_connections: "1000"
          mongodb_threads_allowed_to_block_multiplier: "5"
          proxied_requests_thread_pool_size: "32"

      # MongoDB
      mongodb:
        enabled: true
        version: "4.4"
        repo: "mongodb"
        service:
          enabled: true
          state: "started"
      # MongoDB -> install
      mongodb_install:
        enabled: true
        packages: ["mongodb-org"]

      # ElasticSearch
      elasticsearch:
        enabled: true
        version: "7.x"
        repo: "elastic"
        service:
          enabled: true
          state: "stopped"
      # ElasticSearch -> install
      elasticsearch_install:
        enabled: true
        packages: ["{{ graylog_const[ansible_os_family]['elasticsearch_packages'] }}"]
      # ElasticSearch -> config -> elasticsearch.yml
      elasticsearch_yml:
        enabled: true
        file: "elasticsearch.yml"
        src: "elasticsearch_yml.j2"
        backup: true
        data: |
          cluster.name: graylog
          path.data: /var/lib/elasticsearch
          path.logs: /var/log/elasticsearch

      # FirewallD
      firewalld:
        enabled: true
        service:
          enabled: true
          state: "started"
      # FirewallD -> install
      firewalld_install:
        enabled: false
        packages: [firewalld]
      # FirewallD -> rules
      firewalld_rules:
        graylog_gui_port:
          enabled: true
          zone: "public"
          state: "enabled"
          port: "9000/tcp"
          permanent: true
          immediate: true

  tasks:
    - name: role darexsu.graylog
      include_role:
        name: darexsu.graylog

```
##### Install: Graylog (merge version)
```yaml
---
- hosts: all
  become: true

  vars:
    merge:
      # Graylog
      graylog:
        enabled: true
        version: "4.3"
      # Graylog -> install
      graylog_install:
        enabled: true


  tasks:
    - name: role darexsu.graylog
      include_role:
        name: darexsu.graylog

```
##### Configure: Graylog (merge version)
```yaml
---
- hosts: all
  become: true

  vars:
    merge:
      # Graylog
      graylog:
        enabled: true
      # Graylog -> config
      graylog_config:
        enabled: true
        data:
          is_leader: "true"
          node_id_file: "/etc/graylog/server/node-id"
          password_secret: "RObxb6DltymSD5LQtUuRUEqJPHQW"  # Change me -> generate password_secret (pwgen -N 1 -s 28)
          root_username: "admin"
          root_password_sha2: "96e061a52a96166e6fec9013c3c219781bf6d511062250fa542241cb951b2c17"  # Change me -> generate sha256 (echo -n ${password_secret} | sha256sum)
          bin_dir: "/usr/share/graylog-server/bin"
          data_dir: "/var/lib/graylog-server"
          plugin_dir: "/usr/share/graylog-server/plugin"
          http_bind_address: "0.0.0.0:9000"
          http_publish_uri: "http://0.0.0.0:9000/"
          http_enable_gzip: "false"
          elasticsearch_hosts: "http://localhost:9200"
          rotation_strategy: "count"
          elasticsearch_max_docs_per_index: "20000000"
          elasticsearch_max_number_of_indices: "20"
          retention_strategy: "delete"
          elasticsearch_shards: "4"
          elasticsearch_replicas: "0"
          elasticsearch_index_prefix: "graylog"
          allow_leading_wildcard_searches: "false"
          allow_highlighting: "false"
          elasticsearch_analyzer: "standard"
          output_batch_size: "500"
          output_flush_interval: "1"
          output_fault_count_threshold: "5"
          output_fault_penalty_seconds: "30"
          processbuffer_processors: "5"
          outputbuffer_processors: "3"
          processor_wait_strategy: "blocking"
          ring_size: "65536"
          inputbuffer_ring_size: "65536"
          inputbuffer_processors: "2"
          inputbuffer_wait_strategy: "blocking"
          message_journal_enabled: "true"
          message_journal_dir: "/var/lib/graylog-server/journal"
          lb_recognition_period_seconds: "3"
          mongodb_uri: "mongodb://localhost/graylog"
          mongodb_max_connections: "1000"
          mongodb_threads_allowed_to_block_multiplier: "5"
          proxied_requests_thread_pool_size: "32"

  tasks:
    - name: role darexsu.graylog
      include_role:
        name: darexsu.graylog

```
##### Install and configure: Graylog, Elasticsearch, MongoDB, FirewallD (full version)
```yaml
---
- hosts: all
  become: true

  vars:
    # Graylog
    graylog:
      enabled: true
      version: "4.3"
      repo: "graylog"
      service:
        enabled: true
        state: "started"
    # Graylog -> install
    graylog_install:
      enabled: true
      packages: ["graylog-server", "graylog-integrations-plugins"]
    # Graylog -> config
    graylog_config:
      enabled: true
      file: "server.conf"
      src: "server_conf.j2"
      backup: true
      data:
        is_leader: "true"
        node_id_file: "/etc/graylog/server/node-id"
        password_secret: "RObxb6DltymSD5LQtUuRUEqJPHQW"  # Change me -> generate password_secret (pwgen -N 1 -s 28)
        root_username: "admin"
        root_password_sha2: "96e061a52a96166e6fec9013c3c219781bf6d511062250fa542241cb951b2c17"  # Change me -> generate sha256 (echo -n ${password_secret} | sha256sum)
        bin_dir: "/usr/share/graylog-server/bin"
        data_dir: "/var/lib/graylog-server"
        plugin_dir: "/usr/share/graylog-server/plugin"
        http_bind_address: "0.0.0.0:9000"
        http_publish_uri: "http://0.0.0.0:9000/"
        http_enable_gzip: "false"
        elasticsearch_hosts: "http://localhost:9200"
        rotation_strategy: "count"
        elasticsearch_max_docs_per_index: "20000000"
        elasticsearch_max_number_of_indices: "20"
        retention_strategy: "delete"
        elasticsearch_shards: "4"
        elasticsearch_replicas: "0"
        elasticsearch_index_prefix: "graylog"
        allow_leading_wildcard_searches: "false"
        allow_highlighting: "false"
        elasticsearch_analyzer: "standard"
        output_batch_size: "500"
        output_flush_interval: "1"
        output_fault_count_threshold: "5"
        output_fault_penalty_seconds: "30"
        processbuffer_processors: "5"
        outputbuffer_processors: "3"
        processor_wait_strategy: "blocking"
        ring_size: "65536"
        inputbuffer_ring_size: "65536"
        inputbuffer_processors: "2"
        inputbuffer_wait_strategy: "blocking"
        message_journal_enabled: "true"
        message_journal_dir: "/var/lib/graylog-server/journal"
        lb_recognition_period_seconds: "3"
        mongodb_uri: "mongodb://localhost/graylog"
        mongodb_max_connections: "1000"
        mongodb_threads_allowed_to_block_multiplier: "5"
        proxied_requests_thread_pool_size: "32"

    # MongoDB
    mongodb:
      enabled: true
      version: "4.4"
      repo: "mongodb"
      service:
        enabled: true
        state: "started"
    # MongoDB -> install
    mongodb_install:
      enabled: true
      packages: ["mongodb-org"]

    # ElasticSearch
    elasticsearch:
      enabled: true
      version: "7.x"
      repo: "elastic"
      service:
        enabled: true
        state: "stopped"
    # ElasticSearch -> install
    elasticsearch_install:
      enabled: true
      packages: ["{{ graylog_const[ansible_os_family]['elasticsearch_packages'] }}"]
    # ElasticSearch -> config -> elasticsearch.yml
    elasticsearch_yml:
      enabled: true
      file: "elasticsearch.yml"
      src: "elasticsearch_yml.j2"
      backup: true
      data: |
        cluster.name: graylog
        path.data: /var/lib/elasticsearch
        path.logs: /var/log/elasticsearch

    # FirewallD
    firewalld:
      enabled: true
      service:
        enabled: true
        state: "started"
    # FirewallD -> install
    firewalld_install:
      enabled: false
      packages: [firewalld]
    # FirewallD -> rules
    firewalld_rules:
      graylog_gui_port:
        enabled: true
        zone: "public"
        state: "enabled"
        port: "9000/tcp"
        permanent: true
        immediate: true

  tasks:
    - name: role darexsu.graylog
      include_role:
        name: darexsu.graylog

```
##### Install: Graylog (full version)
```yaml
---
- hosts: all
  become: true

  vars:
    # Graylog
    graylog:
      enabled: true
      version: "4.3"
      repo: "graylog"
      service:
        enabled: true
        state: "started"
    # Graylog -> install
    graylog_install:
      enabled: true
      packages: ["graylog-server", "graylog-integrations-plugins"]

  tasks:
    - name: role darexsu.graylog
      include_role:
        name: darexsu.graylog

```
##### Configure: Graylog (full version)
```yaml
---
- hosts: all
  become: true

  vars:
    # Graylog
    graylog:
      enabled: true
      version: "4.3"
      repo: "graylog"
      service:
        enabled: true
        state: "started"
    # Graylog -> config
    graylog_config:
      enabled: true
      file: "server.conf"
      src: "server_conf.j2"
      backup: true
      data:
        is_leader: "true"
        node_id_file: "/etc/graylog/server/node-id"
        password_secret: "RObxb6DltymSD5LQtUuRUEqJPHQW"  # Change me -> generate password_secret (pwgen -N 1 -s 28)
        root_username: "admin"
        root_password_sha2: "96e061a52a96166e6fec9013c3c219781bf6d511062250fa542241cb951b2c17"  # Change me -> generate sha256 (echo -n ${password_secret} | sha256sum)
        bin_dir: "/usr/share/graylog-server/bin"
        data_dir: "/var/lib/graylog-server"
        plugin_dir: "/usr/share/graylog-server/plugin"
        http_bind_address: "0.0.0.0:9000"
        http_publish_uri: "http://0.0.0.0:9000/"
        http_enable_gzip: "false"
        elasticsearch_hosts: "http://localhost:9200"
        rotation_strategy: "count"
        elasticsearch_max_docs_per_index: "20000000"
        elasticsearch_max_number_of_indices: "20"
        retention_strategy: "delete"
        elasticsearch_shards: "4"
        elasticsearch_replicas: "0"
        elasticsearch_index_prefix: "graylog"
        allow_leading_wildcard_searches: "false"
        allow_highlighting: "false"
        elasticsearch_analyzer: "standard"
        output_batch_size: "500"
        output_flush_interval: "1"
        output_fault_count_threshold: "5"
        output_fault_penalty_seconds: "30"
        processbuffer_processors: "5"
        outputbuffer_processors: "3"
        processor_wait_strategy: "blocking"
        ring_size: "65536"
        inputbuffer_ring_size: "65536"
        inputbuffer_processors: "2"
        inputbuffer_wait_strategy: "blocking"
        message_journal_enabled: "true"
        message_journal_dir: "/var/lib/graylog-server/journal"
        lb_recognition_period_seconds: "3"
        mongodb_uri: "mongodb://localhost/graylog"
        mongodb_max_connections: "1000"
        mongodb_threads_allowed_to_block_multiplier: "5"
        proxied_requests_thread_pool_size: "32"

  tasks:
    - name: role darexsu.graylog
      include_role:
        name: darexsu.graylog

```