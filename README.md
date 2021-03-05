# Description

This role deploys [Restic](https://restic.net/) and condigures timed backups.

# Configuration

```yaml
restic_repo_path: '/data/backups'
restic_repo_master_pass: 'super-secret-password'
restic_ssh_public_key: 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDA6mu...'

# Configuration of individual backups
restic_backups:
  - name: 'app-xyz-postgres'
    tags: ['postgres']
    path: '/docker/app-xyz/db'
    frequency: 'daily'
    timeout: 120
```
The `restic_repo_master_pass` is used to generate a per-host password using hostname as a salt.

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

# Security

The repository is encrypted using a password derived from the Resitc master password hashed with sha512 using the hostname as salt:
```yaml
restic_repo_pass: >
  {{ restic_repo_master_pass | mandatory
  | password_hash('sha512', 65534 | random(seed=inventory_hostname)
  | string) }}
```
This way the password remains deterministic based on hostname, but is different for each host.

The danger with that is that if the hostname changes the password will also change.
In that case the role will fail with an error before it changes the password.
A manual intervention might be required, like initializing a new Restic repository.
