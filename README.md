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
- ara_read_login_required: Sets ARA_READ_LOGIN_REQUIRED (default false).
- ara_write_login_required: Sets ARA_WRITE_LOGIN_REQUIRED (default false).
- ara_nginx_image: Nginx image (default nginx:1.27.0).
- ara_api_http_port: Host HTTP port for API endpoint (default 8088).
- ara_api_https_port: Host HTTPS port for API endpoint (default 8444).
- ara_web_http_port: Host HTTP port for WEB endpoint (default 8089).
- ara_web_https_port: Host HTTPS port for WEB endpoint (default 8445).
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
