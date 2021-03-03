# Description

This role deploys [Restic](https://restic.net/) and condigures timed backups.

# Configuration

```yaml
restic_repo_path: '/data/backups'
restic_repo_pass: 'super-secret-password'
restic_ssh_public_key: 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDA6mu...'

# Configuration of individual backups
restic_backups:
  - name: 'app-xyz-postgres'
    path: '/docker/app-xyz/db'
    frequency: 'daily'
    timeout: 120
```

# Management

You can check existing backups using:
```
TODO
```
