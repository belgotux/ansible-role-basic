Basic
=====

Boilerplate to configure Debian/Ubuntu servers with useful tools and aliases.

Requirements
------------

- Ubuntu / Debian OS / WSL
- SSH key installed on the servers (root or a user with become)

The playbooks [`playbook-init-server.yml`](playbook-init-server.yml) can be use first to init a new server with public keys and install sudo for Debian. Only need to execute once : 
```
ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventories/prod roles/basic/playbook-init-server.yml -e 'mykey=~/.ssh/id_ed25519.pub myhost=node9 become_meth=su' --become-method=su -kK
``` 

- Copy [`basic.aliases.example`](files/basic.aliases.example) to `basic.aliases` to add your aliaseses

Role Variables
--------------

- `bm` Install bare metal tools if defined
- `ssh_key_filename` SSH key generated for root (default id_rsa)

- `basic_list_users` list of users and ssh keys :
  - `name` username
  - `primarygroup` user's default group
  - `groups` users's other groups
  - `pubkeys` list of public keys, filename in the `files` folder 
  - `home` user home (default `/home/username`)
  - `shell` user's shell (default `/bin/bash`)
  - `create_home` if you want to create home directory (default `true`)
  - `append` append or replace groups (default `true` to append)
  - `passwd` password hash of the user 
    - `mkpasswd --method=sha-512` rresult in `$6$`
    - `docker run --rm -it ulikoehler/mkpasswd` result in yescript `$y$` for ubuntu >22.04
  - `generate_ssh_key` generate ssh keys pair for user (default `false`)
- `root_public_keys` list of public key files to copy from `files` folder to root

- `bash_alias_shared` enable shared alias (Installed alias in /usr/share only **with root user ONLY** via `remote_user` or `become` in your playbook) (default no to install only for `remote_user` in his homepath)
- `bash_alias_dir_share` (default /usr/share)

### optional
- `basic_auto_upgrade` Configure inattended-upgrades (default false)
- `basic_packages_default` List of common packages to installed
- `basic_packages_extra` List of others packages for specific group or hosts
- `staff_directories` list of directory that can be modified by staff group
- `basic_custom_scripts_common` list of local scripts to put to /usr/local/bin (to use in groups)
- `basic_custom_scripts_local` list of additionnal scripts to put to /usr/local/bin (to use in hosts)
- `basic_udev_rules` list of `[name,dest,value]` to put content into file in `/etc/udev/rules.d`
- `basic_custom_systemd_common` and `basic_custom_systemd_local` list of `[name,type,dest,src|value]` to put file into directory `/etc/systemd/{{type}}`
- ``basic_ufw` install ufw and configure minimal ssh allow and default IN policy to deny (default false)

Copy cron
---------
You can copy cron files into /etc/cron.d/ based on group name. Just put files into directory `files/cron/YOUR_GROUP/` to copy them.

Example Playbook
----------------

### Playboot to to initiate a fresh raspberry pi `init-new-host.yml` : 
```yml
- hosts: all
  roles:
    - name: basic
      vars: 
        basic_list_users:
          - name: belgotux
            groups: sudo,users,staff,adm
            passwd: $6$xxxx
            pubkeys:
              - xxx.pub
              - yyy.pub
        bash_alias_shared: true
```
Usage : 
```
ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventories/prod init-new-host.yml -u pi --limit octoprint -kK
```


### Example in use of a large playbook with other roles
```yml
- hosts: [homeservers,vps]
  roles:
    - name: basic
      vars: 
        basic_list_users:
          - name: belgotux
            groups: sudo,users,staff,adm
            shell: "/bin/zsh"
            passwd: $6$xxxx
            pubkeys:
              - xxx.pub
              - yyy.pub
        basic_sudo_passwordless: true
        bash_alias_shared: true
        basic_users_and_path_alias_list:
          - user: root
            path: /root
          - user: root
            path: /etc/skel
          - user: pi
            path: /home/pi
          - user: belgotux
            path: /home/belgotux
      tags: basic
    - role: viasite-ansible.zsh
      tags: zsh
      become: true
    - role: postfix-client
      tags: postfix
```

License
-------

[GPL v3](https://www.gnu.org/licenses/gpl-3.0.en.html)

Author Information
------------------

Belgotux
[MonLinux](https://www.monlinux.net)
