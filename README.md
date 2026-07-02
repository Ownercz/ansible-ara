# sw-docker-ansible-ara

Deploys Ansible ARA with Docker Compose at `/opt/docker/ansible-ara/` and exposes it through an Nginx frontend container.

## Features

- Deploys `recordsansible/ara-api` with persistent storage.
- Fronts ARA with Nginx (HTTP and optional HTTPS).
- Uses idempotent template-driven configuration.
- Supports zero-downtime style config updates through handler-based container recreation.

## Main Variables

- `ara_server_name`: Primary public hostname for ARA.
- `ara_public_aliases`: Additional hostnames for the same endpoint.
- `ara_root_dir`: Base deployment directory (default `/opt/docker/ansible-ara`).
- `ara_image`: ARA image (default `docker.io/recordsansible/ara-api:latest`).
- `ara_nginx_image`: Nginx image (default `nginx:1.27.0`).
- `ara_enable_tls`: Enable HTTPS listener and certificate usage.
- `ara_tls_cert_path`: TLS certificate path on host.
- `ara_tls_key_path`: TLS private key path on host.

See `defaults/main.yml` for complete defaults.

## Example Playbook

```yaml
---
- name: Deploy Ansible ARA
	hosts: ara_servers
	become: true
	roles:
		- role: sw-docker-ansible-ara
			vars:
				ara_server_name: ara.lipovcan.cz
				ara_public_aliases:
					- ara-alt.lipovcan.cz
				ara_enable_tls: true
				ara_tls_cert_path: /opt/ssl/cert.pem
				ara_tls_key_path: /opt/ssl/cert.key
```

The default deployment path is `/opt/docker/ansible-ara/` and persistent ARA data is stored in `/opt/docker/ansible-ara/data/server`.
