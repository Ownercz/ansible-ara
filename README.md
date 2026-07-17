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
- Supports handler-based stack recreation after config changes.

## Main Variables

- ara_api_server_name: Public hostname for API endpoint.
- ara_api_public_aliases: Additional hostnames for API endpoint.
- ara_web_server_name: Public hostname for web endpoint.
- ara_web_public_aliases: Additional hostnames for web endpoint.
- ara_server_name: Backward compatibility alias mapped to ara_web_server_name.
- ara_public_aliases: Backward compatibility alias mapped to ara_web_public_aliases.
- ara_root_dir: Base deployment directory (default /opt/docker/ansible-ara).
- ara_image: ARA image (default docker.io/recordsansible/ara-api:latest).
- ara_external_auth: Enables ARA external auth mode (default true).
- ara_read_login_required: Sets ARA_READ_LOGIN_REQUIRED (default false).
- ara_write_login_required: Sets ARA_WRITE_LOGIN_REQUIRED (default false).
- ara_auth_dir: Directory for persisted auth files (default /opt/docker/ansible-ara/data/auth).
- ara_enable_api_basic_auth: Enables API basic auth on nginx (default true).
- ara_api_basic_auth_username: API basic auth username (default ansible).
- ara_api_basic_auth_password: API basic auth password (default demo).
- ara_enable_web_basic_auth: Enables WEB basic auth on nginx (default true).
- ara_web_basic_auth_username: WEB basic auth username (default web).
- ara_web_basic_auth_password: WEB basic auth password (default demo).
- ara_auto_configure_host_security: Auto-generates ARA_ALLOWED_HOSTS and origin lists from endpoint hostnames.
- ara_extra_allowed_hosts: Extra hostnames/IPs appended to ARA_ALLOWED_HOSTS.
- ara_extra_csrf_trusted_origins: Extra origins appended to ARA_CSRF_TRUSTED_ORIGINS.
- ara_extra_cors_origin_whitelist: Extra origins appended to ARA_CORS_ORIGIN_WHITELIST.

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
- ara_enable_tls: Enable HTTPS listener and certificate usage.
- ara_nginx_listen_ssl_only: When true and TLS is enabled, expose/listen only SSL (default true).
- ara_tls_cert_path: TLS certificate path on host.
- ara_tls_key_path: TLS private key path on host.

See defaults/main.yml for complete defaults.

## Example Playbook

		---
		- name: Deploy Ansible ARA
			hosts: ara_servers
			become: true
			roles:
				- role: sw_docker_ansible_ara
					vars:
						ara_api_server_name: ara-api.lipovcan.cz
						ara_web_server_name: ara.lipovcan.cz
						ara_api_public_aliases: []
						ara_web_public_aliases:
							- ara-alt.lipovcan.cz
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
By default, two endpoint hostnames are configured: ara-api.lipovcan.cz (API) and ara.lipovcan.cz (web).

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
