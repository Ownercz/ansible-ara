# sw-docker-ansible-ara

Deploys Ansible ARA with Docker Compose at /opt/docker/ansible-ara and exposes it through an Nginx frontend container.

## Ansible Galaxy

Role name in Galaxy metadata: sw_docker_ansible_ara

Install from Git:

		ansible-galaxy role install git+https://github.com/Ownercz/ansible-ara.git

Install from requirements.yml:

		---
		- src: git+https://github.com/Ownercz/ansible-ara.git
			name: sw_docker_ansible_ara

## Features

- Deploys recordsansible/ara-api with persistent storage.
- Fronts ARA with Nginx (HTTP and optional HTTPS).
- Uses separate ara-api.conf and ara-web.conf vhosts in ara-nginx.
- Adds dedicated ara-prometheus.conf vhost for metrics endpoint with optional basic auth.
- Publishes API and WEB through separate nginx containers to avoid host port conflicts.
- Uses idempotent template-driven configuration.
- Recreates the stack after configuration changes; changes to the local Prometheus Dockerfile, entrypoint, or exporter also rebuild its image.

## Main Variables

| Variable | Default | Description |
| --- | --- | --- |
| `ara_root_dir` | `/opt/docker/ansible-ara` | Base deployment directory. |
| `ara_compose_dir` | `{{ ara_root_dir }}/compose` | Directory containing the Compose project and local build context. |
| `ara_image` | `docker.io/recordsansible/ara-api:latest` | ARA API image. |
| `ara_nginx_image` | `nginx:1.27.0` | Nginx image. |
| `ara_api_server_name` | `airflow.lipovcan.cz` | Public API hostname. |
| `ara_api_public_aliases` | `[]` | Additional API hostnames. |
| `ara_web_server_name` | `{{ ara_server_name }}` | Public web hostname. |
| `ara_web_public_aliases` | `{{ ara_public_aliases }}` | Additional web hostnames. |
| `ara_server_name` | `ara.lipovcan.cz` | Legacy default for `ara_web_server_name`. |
| `ara_public_aliases` | `[]` | Legacy default for `ara_web_public_aliases`. |
| `ara_prometheus_server_name` | `{{ ara_web_server_name }}` | Public Prometheus metrics hostname. |
| `ara_prometheus_public_aliases` | `[]` | Additional metrics hostnames. |
| `ara_api_http_port` / `ara_api_https_port` | `8088` / `8444` | Published API HTTP / HTTPS ports. |
| `ara_web_http_port` / `ara_web_https_port` | `8089` / `8445` | Published web HTTP / HTTPS ports. |
| `ara_prometheus_http_port` / `ara_prometheus_https_port` | `8090` / `8446` | Published metrics HTTP / HTTPS ports. |
| `ara_external_auth` | `true` | Enable ARA external authentication. |
| `ara_read_login_required` / `ara_write_login_required` | `false` / `false` | ARA login requirements for reading / writing. |
| `ara_auth_dir` | `{{ ara_root_dir }}/data/auth` | Directory for persisted basic auth files. |
| `ara_enable_api_basic_auth` / `ara_enable_web_basic_auth` | `true` / `true` | Enable Nginx basic auth on the API / web endpoint. |
| `ara_api_basic_auth_username` / `ara_api_basic_auth_password` | `ansible` / `demo` | API basic auth credentials. |
| `ara_web_basic_auth_username` / `ara_web_basic_auth_password` | `web` / `demo` | Web basic auth credentials. |
| `ara_auto_configure_host_security` | `true` | Derive ARA allowed hosts and trusted origins from public endpoints. |
| `ara_extra_allowed_hosts` | `[]` | Additional ARA allowed hosts. |
| `ara_extra_csrf_trusted_origins` | `[]` | Additional CSRF trusted origins. |
| `ara_extra_cors_origin_whitelist` | `[]` | Additional CORS allowed origins. |
| `ara_prometheus_enabled` / `ara_prometheus_build_locally` | `true` / `true` | Enable metrics and build the Prometheus image locally. |
| `ara_prometheus_patch_exporter` | `true` | Include the extended exporter in the local image. |
| `ara_prometheus_ara_version` | `""` | Pin the ARA version in the local image; empty uses the latest release. |
| `ara_prometheus_playbook_breakdown` / `ara_prometheus_task_breakdown` | `true` / `true` | Emit the wide `ara_playbook_runs` / `ara_task_runs` series and counters. |
| `ara_prometheus_tags_breakdown` | `true` | Emit list-valued tag breakdowns (`ara_playbooks_by_tag` / `_by_skip_tag` / `ara_tasks_by_tag`). |
| `ara_prometheus_count_limited_runs` | `false` | Include `--limit` partial runs in the wide run metrics. |
| `ara_prometheus_task_name_breakdown` | `false` | Enable high-cardinality per-task-name metrics. |
| `ara_prometheus_task_name_allowlist` | `[]` | Restrict task-name metrics to specific names. |
| `ara_enable_tls` | `true` | Enable HTTPS listeners and certificates. |
| `ara_nginx_listen_ssl_only` | `true` | Publish only HTTPS when TLS is enabled. |
| `ara_tls_cert_path` / `ara_tls_key_path` | `/opt/ssl/cert.pem` / `/opt/ssl/cert.key` | TLS certificate / private key paths on the host. |

Note: Internal ARA container hostnames (for example ara-server) are always appended to ARA_ALLOWED_HOSTS so internal services such as ara-prometheus can query the API directly.
- ara_nginx_image: Nginx image (default nginx:1.27.0).
- ara_prometheus_server_name: Public hostname for Prometheus metrics endpoint.
- ara_prometheus_public_aliases: Additional hostnames for Prometheus metrics endpoint.
- ara_api_http_port: Host HTTP port for API endpoint (default 8088).
- ara_api_https_port: Host HTTPS port for API endpoint (default 8444).
- ara_web_http_port: Host HTTP port for WEB endpoint (default 8089).
- ara_web_https_port: Host HTTPS port for WEB endpoint (default 8445).
- ara_prometheus_http_port: Host HTTP port for Prometheus metrics endpoint (default 8090).
- ara_prometheus_https_port: Host HTTPS port for Prometheus metrics endpoint (default 8446).
- ara_prometheus_patch_exporter: Overwrite the pip-installed exporter with the extended one in files/ (default true).
- ara_prometheus_ara_version: Pin the ara version the exporter image is built on for reproducible builds (default "" = latest).
- ara_prometheus_playbook_breakdown: Emit the wide ara_playbook_runs{playbook,inventory,status} series + counter (default true).
- ara_prometheus_task_breakdown: Emit the wide ara_task_runs{role,action,status} series + counter (default true).
- ara_prometheus_tags_breakdown: Emit list-valued tag breakdowns (ara_playbooks_by_tag / _by_skip_tag / ara_tasks_by_tag) (default true).
- ara_prometheus_count_limited_runs: Include --limit partial runs in ara_playbook_runs; off by default as partial runs skew run-over-run comparisons.
- ara_prometheus_task_name_breakdown: Per-task-name breakdown; high cardinality, off by default. Bound it with ara_prometheus_task_name_allowlist.
- ara_enable_tls: Enable HTTPS listener and certificate usage.
- ara_nginx_listen_ssl_only: When true and TLS is enabled, expose/listen only SSL (default true).
- ara_tls_cert_path: TLS certificate path on host.
- ara_tls_key_path: TLS private key path on host.

See [defaults/main.yml](defaults/main.yml) for all available settings.

Changes to Compose or Nginx configuration recreate the stack. When a local Prometheus build input (Dockerfile, entrypoint, or exporter) changes, the same handler runs `docker compose up -d --force-recreate --build --remove-orphans`. Without build input changes, it omits `--build`.

## Example Playbook

		---
		- name: Deploy Ansible ARA
			hosts: ara_servers
			become: true
			roles:
				- role: sw_docker_ansible_ara
					vars:
						ara_api_server_name: airflow.lipovcan.cz
						ara_web_server_name: airflow.lipovcan.cz
						ara_api_public_aliases: []
						ara_web_public_aliases:
							- airflow.lipovcan.cz
						ara_api_http_port: 8088
						ara_api_https_port: 8444
						ara_web_http_port: 8089
						ara_web_https_port: 8445
						ara_nginx_listen_ssl_only: true
						ara_read_login_required: false
						ara_write_login_required: false
						ara_enable_tls: true
						ara_tls_cert_path: /opt/ssl/cert.pem
						ara_tls_key_path: /opt/ssl/cert.key

The default deployment path is /opt/docker/ansible-ara and persistent ARA data is stored in /opt/docker/ansible-ara/data/server.
By default, SSL-only mode is enabled and ports are separated: API on 8444, WEB on 8445.
By default, two endpoint hostnames are configured: airflow.lipovcan.cz (API) and airflow.lipovcan.cz (web).

## Authentication

- ARA server runs with external authentication enabled (ARA_EXTERNAL_AUTH=true).
- Nginx protects endpoints with basic auth by default:
  - API: ansible / demo
  - WEB: web / demo
- Credentials are persisted on host in /opt/docker/ansible-ara/data/auth and mounted read-only into nginx containers.

## How To Send Ansible Runs To ARA

1. Install ARA on the Ansible control node:

		pip3 install ara==1.8.0

2. Enable ARA plugin paths:

		export ANSIBLE_CALLBACK_PLUGINS="$(python3 -m ara.setup.callback_plugins)"
		export ANSIBLE_ACTION_PLUGINS="$(python3 -m ara.setup.action_plugins)"
		export ANSIBLE_LOOKUP_PLUGINS="$(python3 -m ara.setup.lookup_plugins)"

3. Configure ARA HTTP client target and API credentials:

		export ARA_API_CLIENT=http
		export ARA_API_SERVER="https://airflow.lipovcan.cz:8444"
		export ARA_API_USERNAME="ansible"
		export ARA_API_PASSWORD="demo"

4. Run playbooks normally:

		ansible-playbook site.yml

Equivalent ansible.cfg snippet:

		[defaults]
		callback_plugins = /home/ownercz/.venv/lib/python3.12/site-packages/ara/plugins/callback
		action_plugins = /home/ownercz/.venv/lib/python3.12/site-packages/ara/plugins/action
		lookup_plugins = /home/ownercz/.venv/lib/python3.12/site-packages/ara/plugins/lookup

		[ara]
		api_client = http
		api_server = https://airflow.lipovcan.cz:8444
		api_username = ansible
		api_password = demo
