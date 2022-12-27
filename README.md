# lab-ansible

A series of playbooks to help me to automate things

---
## Requirements

This requires ansible collection `ansible.posix` as described in `requirements.yml`
Install w/ `ansible-galaxy collection install -r requirements.yml`

### tip:
  run `ansible all -m ping` when adding a new host in the inventory file to add the remote host id to known_hosts

---
## Roles included and usage
### chrony
- configure chrony to use `pool.ntp.org`
### dns
- configure `/etc/resolv.conf`

var | type | description | example values
--- | --- | --- | ---
dnsservers | list of strings | list of dns servers to use | [ "192.168.15.25", "192.168.15.35" ]
searchdomain | string | default search domains | "localdomain.lan"

### motd
- configure `/etc/profile.d/motd.sh`

### netowrking
- disable IPv6

var | type | description | example values
--- | --- | --- | ---
disable_ipv6 | boolean | set `true` to disable ipv6 | `true` or `false`

### ssh
- set the following options on `/etc/ssh/sshd_config`:
```properties
PermitRootLogin no
AllowUsers packer amsoares
Protocol 2
PasswordAuthentication yes
```
**NOTE:** `AllowUsers` settings come from the `user` dictionary

### users
- additional users to create
- example hash:
```yaml
users:
  user1:
    comment: "User 1 description"
    home: "/home/user1"
    uid: 1001
    sshkeys:
      - ssh-rsa (...) user1@host-A
      - ssh-rsa (...) user1@host-B
    state: present
    sudo: true
```
var | type | description | example values
--- | --- | --- | ---
key | string | username | user1
comment | string | description (gecos) | User 1 description
home | string | homedir | /home/user1
uid | number | uid | 1001
sshkeys | list of strings | public ssh keys to add | "ssh-rsa (...) user1@host"
state | string | state of the user, setting to `absent` will remove the user | `absent` or `present`
sudo | boolean | if set to `true` configures sudo | `true` or `false`

- additional set password in a separate list:
```yaml
# Set passwords here with 'ansible-vault encrypt_string'
# command: ansible-vault encrypt_string 'password' --name 'user'
users_passwords:
  user1: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          62626331663538343730613130613463323234366631336232333933393563633236326637383665
          3837653739346362353632363437656661373937636264390a376163316566316161623231336530
          64656532316365656532313239386430323032396238343261656233333565363336343735636362
          3331333936623166320a643565383738373964376531653261396161646134313930343663646634
          6532
```

### run a command ad-hoc on multiple servers
```
$ ansible all -a "uname -a"
$ ansible kubernetes -a "dnf update -y" --become
```
