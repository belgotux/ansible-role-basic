Basic
=====

Boilerplate to configure Debian/Ubuntu servers with useful tools and aliases.

Requirements
------------

- Ubuntu / Debian OS / WSL
- SSH key installed on the servers (root or a user with become)

The playbooks [`playbook-init-server.yml`](playbook-init-server.yml) can be use first to init a new server with public keys and install sudo for Debian. Only need to execute once : 
```
ansible-playbook -i inventories/prod roles/basic/playbook-init-server.yml -e 'myhost=node9 user=devops become_meth=sudo' -kK
``` 

Role Variables
--------------

- `bm` Install bare metal tools if defined
- `ssh_key_filename` SSH key generated for root (default id_rsa)
- `public_keys` list of public key files to copy from `files` folder

- `firstuser` defined 1st sudoer user
- `firstusergroup` defined 1st sudoer user's group
- `firstuserdir` directory of the 1st sudoer user
- `firstusershell` the shell of the 1st dusoer user

- `bash_alias_shared` enable shared alias (Installed alias in /usr/share only **with root user ONLY** via `remote_user` or `become` in your playbook) (default no to install only for `remote_user` in his homepath)
- `bash_alias_dir_share`: (default /usr/share)

Example Playbook
----------------
```yml
- hosts: [homeservers,vps]
  remote_user: root
  roles:
    - name: basic
      vars: 
        firstuser: belgotux
        firstusergroup: belgotux
        firstusershell: "/bin/zsh"
        firstuserdir: "{{('/home/'+ firstuser)}}"
        bash_alias_shared: yes
```

License
-------

[GPL v3](https://www.gnu.org/licenses/gpl-3.0.en.html)

Author Information
------------------

Belgotux
[MonLinux](https://www.monlinux.net)
