root-keys
=========

Configure root user.

This role configures the following.

* Changes the file `/root/.ssh/authorized_keys`.
  * Where user access from.
  * What abilities can be performed.
  * Who of a set of administrators can access.
* Set root user password.
* Disable password login from SSH.

Version
-------

* `3.0.1` --- bug fix, ansible-linting
* `3.0.0` --- update to ansible 2.12.0
* `2.4.0` --- added RHEL9 and CentOS8 support
* `2.3.2` --- Fixed setting PermitRootLogin when it is not previously set
* `2.3.1` --- Bugfix
* `2.3.0` --- Added support for Jammy, removed centos8 support.
* `2.2.0` --- Added support for RHEL8, removed centos6 from testng.
* `2.1.4` --- removed ubuntu precise from testing
* `2.1.3` --- added tests for ubuntu focal, 20.04
* `2.1.2` --- tested with Ansible 2.9.11
* `2.1.1` --- prepare for github
* `2.1.0` --- fixed non working windows, added `root_ssh_config_path` and `administrator_password`
* `2.0.1` --- updated readme
* `2.0.0` --- prohibit SSH login with password as default and allow changing root password
* `1.0.1` --- updated readme
* `1.0.0` --- first production version
* `master` --- latest development version

Requirements
------------

This role is limited to:

* Ubuntu 16.04
* Ubuntu 18.04
* Ubuntu 20.04
* Ubuntu 22.04
* CentOS 7
* CentOS 8
* RHEL 8
* RHEL 9
* Windows

Role Variables
--------------

* `root_keys_allow_ips` --- list of source ip addresses, default `[]`
* `root_keys_restrict` --- comma separated string with restrictions for login, default `restrict,pty`. Available restrictions
  * `restrict` --- limit all, always use first
  * `pty` --- allow tty
  * `port-forwarding` --- allow port forwarding
  * `X11-forwarding` --- allow X forwarding
  * `agent-forwarding` --- allow forward SSH agent
  * See other options with `man sshd`
* `root_keys_users` --- list of dicts with all root users, default `[]`. Dict list elements is defined as following
   * `user` --- linux username, __required__
   * `key` --- file with one ssh key on each line, default `''`
   * `allow_ips` --- list of source ip addresses, default `root_keys_allow_ips`
   * `restrict` --- subset of restrictions, default `root_keys_restrictions`
   * `enabled` --- is the user enabled or not, default `false`
* `root_keys_users_limit` --- limit to only a list of root users, default `[]`
* `root_keys_users_always` --- always add this list of root users, overrides `root_keys_users_limit`, default `['ansible']`
* `root_keys_authorized_keys_file` --- path to the authorized_keys file of root, default `/root/.ssh/authorized_keys`
* `root_remote_password_login` --- allow SSH root login with password, default `false`
* `root_password` --- set root password hash, default not set and lock account
* `root_ssh_config_path` --- path for `sshd_config`, default `/etc/ssh/sshd_config`
* `administrator_password` --- set windows local administrator password in clear text, default not set

Dependencies
------------

None

Example Playbook
----------------

Variables are kept in the `host_vars` or `group_vars` folder usually. Defining everything in playbook is not recommended. This is just an example.

    - hosts: servers
      vars:
      roles:
        - role: root-keys
          root_keys_allow_ips:
            - 10.0.0.0/8
            - 172.16.0.0/24
          root_keys_restrict: restrict,pty
          root_keys_users:
            - user: user1
              key: |
                ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHkCVF05JvfkrfOOESivOxV4N8+A/EMEkF7/nCQMRoQg
              enabled: true
            - user: user2
              key: |
                ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDXBWyAhgHwuJEpIsqS/8Tl3yD4p8Mu9SR31lnM7/PKm
              allow_ips:
                - 192.168.0.0/24
              restrict: restrict,pty,agent-forwarding
              enabled: true
          root_keys_users_limit:
            - user1
            - user3
          root_keys_users_always:
            - user2

### Add custom values in recipe

Append to default lists in `group_vars` or `host_vars`.

#### Append to list element

* In `group_vars` or `host_vars`.
    ```
    root_keys_users_custom:
      - user: user3
        key: |
          ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCyLO8A5/7+Ckyl5REO5RpVg1Hpm4t9oM+Rum6k9W8fr6MqgpbJuDJ9mmb5AsoaL80J62mvZQZqXqYCtAfL7ZXMcZZiFwbhpdwRehFfR8vDjae4P4AooV7Vk99I1jvumes1z/FQMvue27lromgMejq/GcEAndlY9smYuQXLy3ajUw3dV+w4rm4L3+Qbps1lft8PZU/G0FMMqqTIPPdcLOLgD/d/625RhVtwqx0C/z3fykylFJ4RkWYQK5p0F/BO8Rr1un7AWYl2ITnOpxe99B9YfE2GI9sTvFAp221cEiozRPBIGJIa1JoZl+tDzb4XbF27Zjs3+6ywbSC5nrIW5gArlgavwKd+fL6vyDJXD+NM86QcGZvlhnMr4wanfHo6JVnekgbxWVaag/JY+yVO3RVZ/VdN+4nF1PXyjmdaLGebUrTDCh2JiPmSZ5Q7y9oyH5/ayJnGxVGCN/d8RK1So2UUhkWV+Z4J22yc2PEc1RD7HyLEI1t9WCT6nciY/t5it2E=
        enabled: true
        allow_ips:
        restrict:
    ```
* In playbook, merge defaults with your custom values.
    ```
    pre_tasks:
      - name: add custom root_keys_users
        set_fact:
          root_keys_users: '{{ root_keys_users + root_keys_users_custom|default([]) }}'
    ```

Testing
-------

Testing the role with Vagrant running on VirtualBox.

    cd tests
    vagrant up

Rerun tests.

    vagrant provision

Remove test VMs.

    vagrant destroy -f

License
-------

GPLv2

Author Information
------------------

* Arnulf Heimsbakk <aheimsbakk@met.no>
* Audun Marius Gangstø <audunmg@met.no>

###### set vim: spell spelllang=en:
