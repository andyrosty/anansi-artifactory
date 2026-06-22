# anansi-artifactory

Ansible automation for provisioning and running a self-hosted Nexus artifact repository server in a homelab.

## What this project does

- Configures an `artifact_repo` host group.
- Applies baseline system setup (`common` role): hostname, timezone, package updates, and base packages.
- Installs and enables Docker (`docker` role).
- Deploys Sonatype Nexus in Docker Compose (`nexus` role).
- Optionally runs a Cloudflare Tunnel sidecar for external access.

## Repository layout

```text
.
├── ansible.cfg
├── requirements.yml
├── playbooks/
│   └── artifact-repo.yml
├── inventory/homelab/
│   ├── hosts.yml
│   ├── group_vars/
│   │   ├── all.yml
│   │   └── artifact_repo/
│   │       ├── main.yml
│   │       └── vault.yml
│   └── host_vars/
│       └── artifact-repo-01.yml
└── roles/
    ├── common/
    ├── docker/
    └── nexus/
```

## Prerequisites

- Ansible installed on your control machine (2.14+ recommended).
- SSH access to the target host.
- Sudo privileges on the target host.
- Ansible Vault password for encrypted secrets in `inventory/homelab/group_vars/artifact_repo/vault.yml`.

## Setup

1. Install required collections:

   ```bash
   ansible-galaxy collection install -r requirements.yml
   ```

2. Update inventory and variables for your environment:

   - `inventory/homelab/hosts.yml`
   - `inventory/homelab/host_vars/artifact-repo-01.yml`
   - `inventory/homelab/group_vars/all.yml`
   - `inventory/homelab/group_vars/artifact_repo/main.yml`

3. Edit secrets in vault (for example, Cloudflare tunnel token):

   ```bash
   ansible-vault edit inventory/homelab/group_vars/artifact_repo/vault.yml
   ```

## Deploy

Run the playbook:

```bash
ansible-playbook playbooks/artifact-repo.yml --ask-vault-pass
```

If you use a vault password file:

```bash
ansible-playbook playbooks/artifact-repo.yml --vault-password-file .vault_pass
```

## Configuration highlights

- Nexus defaults are in `roles/nexus/defaults/main.yml`.
- Environment-specific Nexus and Cloudflare settings are in `inventory/homelab/group_vars/artifact_repo/main.yml`.
- Shared host settings are in `inventory/homelab/group_vars/all.yml`.
- Docker user/group behavior is in `roles/docker/defaults/main.yml`.

## Verify deployment

- Confirm playbook connectivity:

  ```bash
  ansible artifact_repo -m ping
  ```

- Verify Nexus is listening on the configured host port (default `8081`).
- Open `http://<artifact_repo_ip>:8081` in a browser.

## Notes

- `ansible.cfg` is preconfigured to use `inventory/homelab/hosts.yml` and `roles/`.
- Nexus data persists under `nexus_data_dir` (default `/opt/nexus/data`).
