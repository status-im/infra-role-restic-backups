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
    tags: ['postgres']
    path: '/docker/app-xyz/db'
    frequency: 'daily'
    timeout: 120
```

# Management

You can check existing backups using:
```
 > sudo -iu restic restic -q snapshots
ID        Time                 Host                 Tags        Paths
---------------------------------------------------------------------------------
cd7c53ee  2021-03-03 18:48:42  node-01.example.org  mongodb     /docker/wekan/db
9c380d18  2021-03-03 18:49:09  node-01.example.org  postgres    /docker/hackmd/db
---------------------------------------------------------------------------------
2 snapshots
```
