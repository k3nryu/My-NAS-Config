# Disaster Recovery

This document describes how to rebuild the NAS on a new machine or after OS disk failure.

The goal is not to repair the old system by hand. The goal is to recreate the system from:

- GitHub repositories
- recoverable secrets
- ZFS data disks or offsite backups
- documented restore steps

## Recovery Assumptions

Primary scenario:

- OS disk is lost or replaced.
- ZFS data disks are still physically healthy.
- GitHub is reachable.
- Secrets are recoverable.

Harder scenario:

- NAS hardware or data disks are lost.
- Restore must come from offsite backups.

Both scenarios should be tested with restore drills.

## Do Not Do These

- Do not format or initialize the data disks.
- Do not run destructive disk commands against `/dev/sdX` names.
- Do not blindly run automation before confirming whether the old ZFS pool is importable.
- Do not generate new secrets if the old secrets are needed to decrypt backups or keep tunnels/certificates stable.

## Recovery Inputs

Before starting, collect:

- GitHub access for `k3nryu/My-NAS-Config`
- Ansible Vault password or other secrets access
- Cloudflare account access
- Backup repository credentials
- List of ZFS disk IDs
- Current domain and DNS records
- SSH access to the new host

## Phase 1: Install Minimal OS

Install the same OS family used by the NAS.

Minimum packages:

```bash
dnf install -y git ansible-core rsync curl wget
```

Install ZFS according to the OS version before importing the pool.

## Phase 2: Clone Config Repo

```bash
cd /root
git clone https://github.com/k3nryu/My-NAS-Config.git
cd My-NAS-Config
```

Use SSH instead of HTTPS if SSH keys are already restored.

## Phase 3: Recover Secrets

Recover the secret source before starting services:

- Ansible Vault password
- Cloudflare token or tunnel credentials
- PostgreSQL passwords
- backup encryption password
- app `.env` files

Recommended future state:

- encrypted secrets in Git via Ansible Vault or `sops`
- one offline emergency copy of the decryption key

## Phase 4: Import Existing ZFS Pool

This phase is critical.

Check available pools:

```bash
zpool import
```

Import the existing pool by name:

```bash
zpool import tank
```

Confirm mountpoints:

```bash
zpool status tank
zfs list
```

Only if this is a brand-new NAS with blank disks should the pool creation path be used.

Current caution: the `zfs` role can create `tank` when `zpool list tank` fails. During real disaster recovery with old data disks, import the existing pool first so automation does not treat an unimported pool as missing.

## Phase 5: Run Ansible

After ZFS is imported and secrets are available:

```bash
ansible-playbook ansible/site.yml --ask-vault-pass
```

The playbook should be safe to rerun.

If a referenced role is not implemented yet, either implement it before relying on full automation or temporarily disable that service flag in `ansible/group_vars/nas.yml`.

## Phase 6: Restore Application Data

Restore data in priority order:

1. PostgreSQL
2. Nginx and Cloudflare access path
3. ACME certificate state
4. Docker services and app `.env` files
5. File shares and WebDAV

For PostgreSQL, prefer a tested restore command such as:

```bash
pg_restore --clean --if-exists --dbname "$DATABASE_URL" /path/to/latest.dump
```

Exact backup paths and commands should be added after the backup system is finalized.

## Phase 7: Start Services

Start the managed services:

```bash
systemctl status sshd
systemctl status docker
systemctl status nginx
systemctl status cloudflared
```

For Docker workloads, run the compose or deploy command defined in `docker/`.

## Phase 8: Verify Access

Verify local access:

```bash
curl -f http://127.0.0.1:8000/
```

Verify reverse proxy access:

```bash
curl -I https://example.com
```

Verify Cloudflare tunnel health from the Cloudflare dashboard.

Verify shares from a LAN client:

- Samba
- NFS
- WebDAV, if enabled

## Phase 9: Record The Recovery

After recovery, update:

- `docs/restore-drill.md`
- `docs/inventory.md`
- `docs/recovery-checklist.md`

Record what worked, what failed, and what needs automation.
