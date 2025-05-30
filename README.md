# Description

This role deploys [Restic](https://restic.net/) and configures timed backups.

# Configuration

```yaml
restic_repo_path: '/data/backups'
restic_repo_master_pass: 'super-secret-password'
restic_ssh_public_key: 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDA6mu...'

# Configuration of individual backups
restic_backups:
  - name: 'app-xyz-postgres'
    tags: ['pgdumpdir']
    path: '/docker/app-xyz/db/backup'
    docs: 'https://xyz.app/docs'
    excludes: ['^postmaster.pid$']
    requires: 'dump-xyz-db.service'
    after: 'dump-xyz-db.service'
    frequency: 'daily'
    restart: 'no'
    timeout: 120
    enabled: true
```
The `restic_repo_master_pass` is used to generate a per-host password using [machine ID](https://man7.org/linux/man-pages/man5/machine-id.5.html) as a salt.

Password changing is disabled by default, but can be enabled with `restic_repo_pass_allow_change: true`.

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
The `-i` flag sources `/home/restic/.bashrc` which holds the repo settings.

# Pruning

By default a timer runs daily which cleans up backups to keep only the most recent 30.

You can modify this by setting `restic_pruning_retention`.

# Security

The repository is encrypted using a password derived from the Restic master password hashed with SHA512 using a slice of SHA1 of the hostname as salt:
```yaml
restic_repo_pass: >
  {{ restic_repo_master_pass | mandatory
  | password_hash('sha512', (inventory_hostname|hash('sha1'))[0:16])
  | string }}
```
This way the password remains deterministic based on hostname, but is different for each host.

The danger with that is that if the hostname changes the password will also change.
In that case the role will fail with an error before it changes the password.
A manual intervention might be required, like initializing a new Restic repository.
