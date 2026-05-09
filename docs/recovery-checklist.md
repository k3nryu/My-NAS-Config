# Recovery Checklist

Use this as the short version during an incident. Details live in `docs/disaster-recovery.md`.

## First Five Minutes

- Confirm what failed: OS disk, data disks, network, power, or service config.
- Do not format or initialize disks.
- Photograph or record disk layout if hardware changed.
- Confirm access to GitHub, Cloudflare, and secrets storage.
- Decide whether this is a ZFS import recovery or an offsite backup recovery.

## ZFS Data Disks Survived

- Install minimal OS.
- Install Git, Ansible, and ZFS prerequisites.
- Clone `k3nryu/My-NAS-Config`.
- Recover secrets.
- Run `zpool import` and confirm `tank` is visible.
- Import `tank`.
- Confirm `zpool status tank` and `zfs list`.
- Run `ansible-playbook ansible/site.yml --ask-vault-pass`.
- Restore app `.env` files and service secrets.
- Start Docker, Nginx, Cloudflared, and sharing services.
- Run health checks.

## Data Disks Lost

- Install minimal OS.
- Clone `k3nryu/My-NAS-Config`.
- Recover secrets.
- Create or attach replacement storage.
- Restore ZFS datasets or service data from offsite backup.
- Restore PostgreSQL backups.
- Run Ansible.
- Start services.
- Validate local and public access.

## Health Checks

- `zpool status tank`
- `zfs list`
- `systemctl status docker`
- `systemctl status nginx`
- `systemctl status cloudflared`
- Aide local health: `curl -f http://127.0.0.1:8000/`
- Public domain check through Cloudflare
- PostgreSQL restore verification
- Samba/NFS/WebDAV client check, if enabled

## After Recovery

- Record the incident in `docs/restore-drill.md`.
- Update stale inventory entries.
- Add automation for every manual step that was required.
