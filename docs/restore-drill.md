# Restore Drill

Backups are only real after a restore has been tested.

Use this document to run and record scheduled restore drills. A drill can be done on a temporary VM, a spare machine, or an isolated directory when full machine recovery is not practical.

## Frequency

- Monthly: documentation review and config sanity check
- Quarterly: PostgreSQL restore drill
- Twice per year: full NAS rebuild drill on a spare host or VM
- After major changes: run the relevant partial drill

## Drill Types

### Documentation Drill

Goal: verify that a human can follow the docs.

Checklist:

- `README.md` still points to the correct recovery entry point.
- `docs/inventory.md` lists current services.
- `docs/disaster-recovery.md` has no stale commands.
- Secrets storage location is known.
- Offsite backup location is known.

### Config Drill

Goal: verify automation can still parse and run.

Commands:

```bash
ansible-playbook ansible/site.yml --syntax-check
```

If safe on the current host:

```bash
ansible-playbook ansible/site.yml --check --ask-vault-pass
```

### Database Drill

Goal: verify the latest PostgreSQL backup can restore.

Steps:

1. Create a temporary PostgreSQL instance.
2. Restore the latest backup.
3. Run app health checks against the restored database.
4. Destroy the temporary instance.

Record:

- backup file used
- restore duration
- errors
- missing data
- follow-up tasks

### Full Rebuild Drill

Goal: prove that a fresh host can become the NAS.

Steps:

1. Install minimal OS.
2. Install Git, Ansible, and ZFS prerequisites.
3. Clone `My-NAS-Config`.
4. Recover secrets.
5. Import or simulate ZFS storage.
6. Run Ansible.
7. Restore PostgreSQL and service data.
8. Start Nginx, Cloudflared, Docker services, and shares.
9. Run health checks.

Success criteria:

- Aide responds locally.
- Public access path works through Cloudflare.
- PostgreSQL data is present.
- Nginx routes the expected services.
- Backups can still run after restore.
- No undocumented manual step was required.

## Drill Log

Add one entry per drill.

### YYYY-MM-DD

- Drill type:
- Operator:
- Host:
- Backup used:
- Result:
- Time to restore:
- Problems found:
- Follow-up tasks:
