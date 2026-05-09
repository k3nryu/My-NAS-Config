# NAS Inventory

This file is the human-readable inventory for rebuilding the NAS.

Keep it boring and current. If a service is important enough to run on the NAS, it should appear here with its data path, port, domain, backup status, and restore priority.

## Host

- Hostname: `rhel_nas`
- Ansible connection: local
- Timezone: `Asia/Tokyo`
- Primary config repo: `https://github.com/k3nryu/My-NAS-Config`
- Main ZFS pool: `tank`
- ZFS mountpoint: `/share`

## Storage

Configured ZFS devices:

- `/dev/disk/by-id/wwn-0x5000c500f22d731f`
- `/dev/disk/by-id/wwn-0x5000c500f7307499`

Current intended layout:

```text
/share
├── NAS/
│   ├── aide/
│   ├── archive/
│   ├── backups/
│   ├── docker/
│   ├── programs/
│   └── scripts/
```

Record exact datasets here after they are finalized:

| Dataset | Mountpoint | Purpose | Backup required |
| --- | --- | --- | --- |
| `tank` | `/share` | Main NAS storage | Yes |
| `tank/NAS` | `/share/NAS` | Shared service and user data | Yes |

## Managed By Ansible

Defined in `ansible/site.yml`:

- `base`
- `zfs`
- `firewalld`
- `samba`
- `nfs`
- `docker`
- `nginx`
- `apache_webdav`
- `cloudflared`

Roles currently implemented:

- `base`
- `zfs`

Roles referenced but not yet implemented:

- `firewalld`
- `samba`
- `nfs`
- `docker`
- `nginx`
- `apache_webdav`
- `cloudflared`

## Services

Fill this table as each service becomes managed by this repo.

| Service | Domain | Local port | Data path | Backup | Restore priority | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| Aide | TBD | `127.0.0.1:8000` | `/share/NAS/aide` | Database required | High | App code lives in `k3nryu/aide` |
| PostgreSQL | Internal | `5432` | TBD | Yes | Critical | Backup with `pg_dump` and/or WAL |
| Nginx | Internal/public edge | `80`, `443` | TBD | Config only | Critical | Reverse proxy for local services |
| Cloudflared | Public access | Tunnel | TBD | Config/token only | Critical | Cloudflare Tunnel for external access |
| ACME | Certificate automation | N/A | TBD | Account/config required | High | Prefer DNS-01 through Cloudflare |
| Samba | LAN | TBD | `/share` | Data via ZFS/backup | Medium | Enabled in group vars |
| NFS | LAN | TBD | `/share` | Data via ZFS/backup | Medium | Enabled in group vars |
| Apache WebDAV | LAN/public TBD | TBD | TBD | Config/data TBD | Medium | Enabled in group vars |

## External Dependencies

- GitHub: stores infrastructure and application repositories.
- Cloudflare: DNS, proxy, and/or Tunnel.
- ACME provider: certificate issuance.
- Offsite backup target: TBD.
- Secrets storage: TBD.

## Secrets Not Stored In Git

Keep these out of plain Git:

- Cloudflare API token
- Cloudflare Tunnel token or credentials file
- ACME account key
- PostgreSQL passwords
- Application `.env` files
- Backup repository password
- SSH private keys
- Ansible Vault password

Store recoverable secrets in one controlled place, such as Ansible Vault, `sops` with `age`, or a password manager.
