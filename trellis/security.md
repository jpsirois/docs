---
ID: 6151
post_title: Security
author: Ben Word
post_date: 2015-09-03 17:49:35
post_excerpt: ""
layout: doc
permalink: https://roots.io/trellis/docs/security/
published: true
docs_project:
  - 'a:1:{i:0;s:2:"19";}'
publish_to_discourse:
  - 'a:1:{i:0;s:1:"0";}'
---
## Locking down root

The `sshd` role heightens your server's security by providing better SSH defaults, disabling password authentication for SSH access, and optionally disabling SSH `root` login. To disable `root` login:

* Set `sshd_permit_root_login: false` in `group_vars/all/security.yml`
* Set a sudoer password for the `admin_user` user (see below)
* Run the `server.yml` playbook (see note about `--ask-become-pass` in "Admin User" section below)

You may toggle `sshd_permit_root_login` between `true` or `false` on a server that is already provisioned.

## Admin user

When you set `sshd_permit_root_login: false` and run the `server.yml` playbook, it will connect as `root` one final time and disable `root` login. On subsequent runs, `server.yml` will connect as the `admin_user` defined in `group_vars/all/users.yml` (default `admin`).

With `root` login disabled, the `admin_user` will need to run commands using `sudo` with a password, so you will need to add the option [`--ask-become-pass`](http://docs.ansible.com/ansible/become.html#new-ansible-variables) when running `server.yml`.
```
ansible-playbook server.yml -e env=production --ask-become-pass
```
This prompts you to enter the sudoer password described in the "Admin User Sudoer Password" section below. See the [SSH Keys docs](https://roots.io/trellis/docs/ssh-keys/) for more information about Trellis SSH users.

## Admin user sudoer password

While `server.yml` provisions your server as the `admin_user`, it will perform some operations using `sudo` with a password. You will need to set the sudoer password for `admin` in the list of `vault_sudoer_passwords` defined in `group_vars/<environment>/vault.yml`. Here is an example:

```yaml
vault_sudoer_passwords:
  admin: $6$rounds=100000$JUkj1d3hCa6uFp6R$3rZ8jImyCpTP40e4I5APx7SbBvDCM8fB6GP/IGOrsk/GEUTUhl1i/Q2JNOpj9ashLpkgaCxqMqbFKdZdmAh26/
  another_user: $6$rounds=100000$r3ZZsk/uc31cAxQT$YHMkmKrwgXr3u1YgrSvg0wHZg5IM6MLEzqOraIXqh5o7aWshxD.QaNeCcUX3KInqzTqaqN3qzo9nvc/QI0M1C.
```

The passwords were generated using the python command [found here](http://docs.ansible.com/faq.html#how-do-i-generate-crypted-passwords-for-the-user-module). The passwords generated here are `example_password` and `another_password`, respectively. The ansible user module doesn't handle any encryption and passwords must be encrypted beforehand. It's also recommended `group_vars/<environment>/vault.yml` be encrypted using [Ansible Vault](https://roots.io/trellis/docs/vault/).