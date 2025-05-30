Ansible Role: Netdata-docker
============================

Install Netdata Docker Compose project.

Netdata is a distributed real-time, health monitoring platform for systems, hardware, containers & applications, collecting metrics.

- https://www.netdata.cloud/
- https://learn.netdata.cloud/docs/netdata-agent/installation/docker

Requirements
------------

Requires the following to be installed:
- docker
- docker compose

Role Variables
--------------

Common Docker projects variables:

```yaml
# Base directory for Docker projects
docker_projects_path: # /var/apps
```

Available role variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
# Docker project variables

netdata_project_name: netdata

# Docker project dynamic vars (uses `docker_project_name` prefix, adapt if overridden)

# Port targeted by Traefik router
netdata_traefik_loadbalancer_server_port: "{{ netdata_port }}"

# Scheme targeted by Traefik router
netdata_traefik_loadbalancer_server_scheme: "{{ netdata_api_use_ssl | ternary('https', 'http') }}"

# Traefik router middlewares
netdata_traefik_middlewares: []
#  - "internal-access@file" # see djuuu.traefik_docker templates/dynamic-conf/middlewares/internal-access.yml.j2
#  - "basic-auth@file"      # see djuuu.traefik_docker files/dynamic-conf/middlewares/basic-auth.yml

# Main service additional docker-compose options (ex: cpu_shares, deploy, ...)
netdata_service_additional_options: "" # |
#  cpu_shares: 20
#  deploy:
#    resources:
#      limits:
#        memory: 1024M
```

```yaml
# Netdata project variables

# netdata/netdata container version
netdata_version: stable

netdata_extra_deb_packages: []
#  - fail2ban

netdata_extra_docker_volumes: []
#  - comment: https://learn.netdata.cloud/docs/collecting-metrics/authentication-and-authorization/fail2ban
#    volume: /var/run:/host/var/run:ro
#  - comment: systemd units monitoring
#    volume: /run/dbus:/run/dbus:ro
#  - /other/example:/host/other/example:ro

# https://learn.netdata.cloud/docs/netdata-agent/configuration/securing-agents/web-server-reference#enable-httpstls-support
netdata_api_use_ssl: false

# Only useful when netdata_api_use_ssl is true
# Note: traefik_domains is generated by djuuu.docker_project role, for djuuu.traefik_docker integration
# See:
#   - https://github.com/Djuuu/ansible-role-docker-project?tab=readme-ov-file#generated-variables
#   - https://github.com/Djuuu/ansible-role-docker-project?tab=readme-ov-file#dynamic-variables
#   - https://github.com/Djuuu/ansible-role-traefik-docker
netdata_api_healthcheck_domain: "{{ traefik_domains[0] | default(none) }}"

# Connect Agent to Cloud (optional)
netdata_claim_token:
netdata_claim_rooms: []
#  - 12345678-9abc-def0-1234-456789abcdef # All nodes
#  - 23456789-abcd-ef01-2344-56789abcdef0 # Servers
#  - ...

# Is node ephemeral?
netdata_is_ephemeral_node: false

# Local directory for Netdata configuration backup 
netdata_config_local_path: "{{ playbook_dir }}/config/netdata/{{ inventory_hostname }}"
```

```yaml
## Streaming
# https://learn.netdata.cloud/docs/observability-centralization-points/metrics-centralization-points/configuring-metrics-centralization-points

# Streaming API key (parent) (generate with uuidgen)
netdata_stream_api_key:
# Allowed ips (set to false or empty to disable streaming to host)
netdata_stream_allow_from: "*"

# Streaming target
netdata_stream_to_destination:
netdata_stream_to_api_key:
```

Config files
------------

Files in the following locations will be copied in the project's config directory:

- Netdata config:  
  `config/netdata/{{ inventory_hostname }}/config/netdata.conf`
- Plugins config:  
  `config/netdata/{{ inventory_hostname }}/config/go.d/*.conf`  
  (and other plugin types: `charts.d`, `custom-plugins.d`, `health.d`, `python.d`, `statsd.d`)

Dependencies
------------

This role depends on :
- [djuuu.docker_project](https://github.com/Djuuu/ansible-role-docker-project)

Some variables allow integration with:
- [djuuu.traefik_docker](https://github.com/Djuuu/ansible-role-traefik-docker)

Example Playbooks
-----------------

```yaml
- hosts: all
  gather_facts: false

  roles:
    - djuuu.netdata_docker
```

```yaml
- hosts: all
  gather_facts: false

  tasks:
    - name: Backup Netdata config
      ansible.builtin.include_role:
        name: djuuu.netdata_docker
        tasks_from: get-config
```

```yaml
- hosts: all
  gather_facts: false

  tasks:
    - name: Get Netdata info
      ansible.builtin.include_role:
        name: djuuu.netdata_docker
        tasks_from: get-info
      tags: [always]
```

License
-------

Beerware License
