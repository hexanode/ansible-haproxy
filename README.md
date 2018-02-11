# Ansible Role: Firewall

An Ansible Role that installs iptables and configure HAProxy.

We want this role as simple, configurable and interoperable as possible.

## Requirements

Compliant with :
- Debian 9 (Stretch)

Other:
- Fact gathering should be allowed in ansible-playbook (By default)
- The role require the superuser privileges. The task should be used with remote_user root or with a sudo/su grant

## Role behavior


## Role Variables

Available variables with defaults values are defined in `defaults/main.yml`.

Modifiables variables and possible values are listed below :

```
# Security
haproxy_chroot: '/var/lib/haproxy'

# User & Group for HAProxy
haproxy_user: haproxy
haproxy_group: haproxy

# Web Stats

# Set to true in order to enable webstats. (default is false)
haproxy_webstats: false
haproxy_webstats_ip: '0.0.0.0'
haproxy_webstats_port: '8080'
haproxy_webstats_maxconn: '10'
haproxy_webstats_refreshtime: '30s'
haproxy_webstats_username: 'admin'
haproxy_webstats_password: 'password'
haproxy_webstats_uri: '/haproxy?stats'

# Frontends
haproxy_frontends_list: []

# Backends
haproxy_backends_list: []


```

## Dependencies

none


## Example Playbook

How to use the role in your ansible playbook.

```
- name: Playbook Task to install haproxy role
  hosts: all (Group of servers on which you want to install and configure haproxy)
  roles:
    - { role: haproxy, tags: [ 'haproxy' ] }
```

## ToDo


## License

GPLv3


## Author Information

This role is maintained by maximiliend. Issues and Merge Request are welcome.
