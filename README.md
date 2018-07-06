# Ansible Role: HAProxy

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


## Frontends Syntax

Here is a recap of all possible values for haproxy_frontends in haproxy_frontends_list.

| Option           | Role                            | Default value | Possible values                     |
|------------------|---------------------------------|:-------------:|-------------------------------------|
| name             | Frontend Name                   |      none     | String                              |
| bind             | Bind frontend with interface    |      none     | * (any), any IP Adress              |
| port             | Bind frontend with a port on if |      none     | any port number 1-65535             |
| mode             | Protocol used for the instance  |      http     | [Mode](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#4-mode) |
| options          | List of options                 |      none     | All options available for frontend  |
| default_backend  | Default backend used            |      none     | String                              |

**Note :**
- If you omit an option the default will be used.

The syntax is a bit flexible, per example you can use the following syntax.

```
[
    { name: 'default', bind: '*', port: '80', options: [], default_backend: 'www' },
    { name: 'example', bind: '93.184.216.34', port: '8080', options: [], default_backend: 'example_backend' }
]
```

This will be respectively translated to :

```
frontend default
    bind *:80
    mode http
    default_backend www

frontend example
    bind 93.184.216.34:8080
    mode http
    default_backend example_backend
```


## Backends Syntax

Here is a recap of all possible values for haproxy_backends in haproxy_backends_list.

| Option              | Role                                  | Default value | Possible values                     |
|---------------------|---------------------------------------|:-------------:|-------------------------------------|
| name                | Frontend Name                         |      none     | String                              |
| mode                | Protocol used for the instance        |      http     | [Mode](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#4-mode) |
| balance             | Algorithm used to load balance        |      none     | [Balance](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#4.2-balance) |
| options             | List of options                       |      none     | All backend options (name, value)   |
| noforwardfor        | Don't add http header X-Forwarded-For |      false    | boolean                             |
| backend_server_list | List of backend servers               |      none     | See backend_server syntax guide     |

**Note :**
- If you omit an option the default will be used.

The syntax is a bit flexible, per example you can use the following syntax.

```
[
    { name: 'default_backend', balance: 'roundrobin', options: [ {name: 'tcp-check', value: '' } ], backend_server_list: [] }
]
```

This will be respectively translated to :

```
backend default_backend
    mode http
    balance roundrobin
    cookie SERVERID insert indirect
    option tcp-check
    option forwardfor
```


## Backend Server Syntax

Here is a recap of all possibles values for backend_server in backend_server_list in haproxy_backends configuration

| Option              | Role                                  | Default value | Possible values                     |
|---------------------|---------------------------------------|:-------------:|-------------------------------------|
| name                | Frontend Name                         |      none     | String                              |
| address             | IP Address of backend server          |      none     | Single IP                           |
| port                | Port number of backend server         |      none     | Port Number                         |


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
