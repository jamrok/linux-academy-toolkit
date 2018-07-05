# change_linux_academy_server_password

  For all linux academy servers:
  - Change any specified user's current SSH password to a new password (and for `root` if default user is given)
  - Setup ssh keys for specified user (and for `root` if default user is given)
  - Create or update static inventory file `la_hosts.yml` for use with ansible

## Task Summary
  - Ensure Linux Academy username is provided
  - Determine whether to use regular hostname or alternative names
  - Dynamically add to ansible hosts list from the local machine
  - Obtain SSH Username, Current Password and New Password
  - Prepare to change password for provided user.
    - If default user (`user`) is given, also change creds for `root` user
  - Get IP for each host
  - Get Remote Host Key (ignore security implications, assume host is safe)
  - Add Remote Host Key to Known Hosts File
  - Ensure SSH Private Key Exists
  - Change Password on Remote Servers

## Contributors
  - Author: Jamrok
  - Maintainer: Jamrok

## Supporting Docs
  - [Introduction To Ansible Ad-Hoc Commands](https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html)
  - [Manual for `ansible-playbook`](https://docs.ansible.com/ansible/latest/cli/ansible-playbook.html)
  - [Manual for `ansible`](https://docs.ansible.com/ansible/latest/cli/ansible.html)

## Assumptions
  - dig, expect and ssh-keyscan is available on the local system

## Precautions
  - N/A

## Rollback
  - None

## Requirements
  - **Ansible**: <= 2.5.5
    - Minimum verified version: 2.4.0.0
    - A bug in 2.6.0 is breaking functionality (if old password is incorrect).
      - Workaround:
        Update your `ansible/plugins/connection/ssh.py` as follows:
```
          -        except (OSError, IOError):
          -            raise AnsibleConnectionFailure('SSH Error: data could not be sent to remote host "%s". Make sure this host can be reached over ssh' % self.host)
          +        except (OSError, IOError) as e:
          +            if (e.errno != errno.EPIPE):
          +                raise AnsibleConnectionFailure('SSH Error: data could not be sent to remote host "%s". Make sure this host can be reached over ssh' % self.host)
```
  - Requires local root: no

## Compatibility
  - OS: CentOS 7 / Ubuntu 16.04
  - Idempotent: Yes
  - Check Mode: No

## Variables
  - `user` - Your Linux Academy Username
    - permitted values: your username (not your email)
    - type: string
    - default: undefined
    - required: yes

  - `suffix` - Use alternate server name (eg. username1b.mylabserver.com)
    - permitted values: y/yes/n/no
    - type: bool
    - default: no
    - required: yes

  - `default_key` - Use default ssh key `~/.ssh/id_rsa`? If not, use separate key file `./id_rsa_la_servers`
    - permitted values: y/yes/n/no
    - type: bool
    - default: yes
    - required: yes

  - `ssh_user` - SSH username whose password will be changed. Also changes the `root` user's creds when `user` is given.
    - permitted values: user, root, etc
    - type: string
    - default: 'user'
    - required: yes

  - `ssh_pass` - Current password for SSH user
    - permitted values: any_password
    - type: string
    - default: '123456'
    - required: yes

  - `new_ssh_pass` - New password for SSH user
    - permitted values: any_password
    - type: string
    - default: undefined
    - required: yes

  - `new_ssh_pass2` - Confirmation of new password for SSH user
    - permitted values: any_password
    - type: string
    - default: undefined
    - required: yes

## Examples

### Basic usage (you will be prompted of more information)

  ```bash
  ansible-playbook change_linux_academy_server_password.yml
```

### Basic usage with some verbosity

  ```bash
  ansible-playbook change_linux_academy_server_password.yml -vv --diff
```

### Basic usage with some verbosity + limit to a couple hosts (la-username1 and la-username3)

  ```bash
  ansible-playbook change_linux_academy_server_password.yml -vv --diff -l "localhost,la-username[13]*"
```
Note: Always include `localhost` because hosts are added dynamically from the `localhost`

### Basic usage with some verbosity and all field provided (no prompts, please change new_ssh_pass)

  ```bash
  ansible-playbook change_linux_academy_server_password.yml -vv --diff -e "user=la-username suffix=n default_key=y ssh_user=user ssh_pass=123456 new_ssh_pass2={{new_ssh_pass}}" -e new_ssh_pass= #<--remove_space_set_pass
```

### After everything runs successfully, verify you can run ansible as regular user and as root

  ```bash
  ansible all -i la_hosts.yml -m shell -a whoami
  la-username6.mylabserver.com | SUCCESS | rc=0 >>
  user
  la-username2.mylabserver.com | SUCCESS | rc=0 >>
  user
  la-username1.mylabserver.com | SUCCESS | rc=0 >>
  user
  la-username4.mylabserver.com | SUCCESS | rc=0 >>
  user
  la-username5.mylabserver.com | SUCCESS | rc=0 >>
  user
  la-username3.mylabserver.com | SUCCESS | rc=0 >>
  user
```

  ```bash
  ansible all -i la_hosts.yml -m shell -a whoami -b
  la-username5.mylabserver.com | SUCCESS | rc=0 >>
  root
  la-username6.mylabserver.com | SUCCESS | rc=0 >>
  root
  la-username1.mylabserver.com | SUCCESS | rc=0 >>
  root
  la-username4.mylabserver.com | SUCCESS | rc=0 >>
  root
  la-username2.mylabserver.com | SUCCESS | rc=0 >>
  root
  la-username3.mylabserver.com | SUCCESS | rc=0 >>
  root
```

### Profit!
