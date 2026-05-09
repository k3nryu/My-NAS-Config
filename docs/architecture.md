# Architecture

`My-NAS-Config` is the source of truth for rebuilding the NAS.

The design goal is simple: the NAS should be reproducible from Git, recoverable secrets, and restorable data backups.

## Layers

```text
GitHub repositories
        |
        v
My-NAS-Config
        |
        v
Ansible + Docker + service configs
        |
        v
RHEL NAS host
        |
        v
ZFS storage + network services
```

## Responsibilities

`My-NAS-Config` owns:

- host bootstrap expectations
- Ansible inventory and roles
- ZFS pool and mountpoint intent
- Docker deployment scripts
- recovery documentation
- service inventory

Application repositories own application code. For example, `aide` owns the FastAPI app, while this repo should own how Aide is run on the NAS.

## Current Automation

Ansible inventory:

- `ansible/inventory/nas.yml`

Host group variables:

- `ansible/group_vars/nas.yml`

Main playbook:

- `ansible/site.yml`

Implemented roles:

- `base`
- `zfs`

Referenced future roles:

- `firewalld`
- `samba`
- `nfs`
- `docker`
- `nginx`
- `apache_webdav`
- `cloudflared`

## Recovery Model

The NAS should be recoverable from:

- GitHub repo checkout
- Ansible Vault or another recoverable secrets system
- existing ZFS disks or offsite backups
- documented Cloudflare and ACME configuration
- PostgreSQL backup artifacts

The recovery process should be rehearsed. See `docs/restore-drill.md`.

## Important ZFS Note

During recovery with existing data disks, import the old pool before running full automation.

The current `zfs` role can create a pool when `zpool list tank` fails. That is useful for brand-new blank disks, but during disaster recovery an unimported pool is not the same thing as a missing pool.
