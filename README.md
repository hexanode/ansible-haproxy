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
haproxy_webstats_realm: 'HAProxy\ Statistics'
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
| comment          | A comment escaped for HAProxy   |      none     | String                              |
| bind             | Bind frontend with interface    |      none     | * (any), any IP Adress              |
| port             | Bind frontend with a port on if |      none     | any port number 1-65535             |
| mode             | Protocol used for the instance  |      tcp      | [Mode](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#4-mode) |
| options          | List of options                 |      none     | All options available for frontend  |
| default_backend  | Default backend used            |      none     | String                              |

**Note :**
- If you omit a value the default (if any) will be used.

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

| Option              | Role                                 | Default value  | Possible values                     |
|---------------------|--------------------------------------|:--------------:|-------------------------------------|
| name                | Frontend Name                        |      none      | String                              |
| cookie              | Cookie definition for backend        |      none      | See Backend Cookie Syntax guide     |
| mode                | Protocol used for the instance       |      tcp       | [Mode](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#4-mode) |
| balance             | Algorithm used to load balance       |   roundrobin   | [Balance](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#4.2-balance) |
| options             | List of options                      |      none      | All options available for backtend  |
| backend_server_list | List of backend servers              |      none      | See Backend Server Syntax guide     |

**Note :**
- If an option is omitted the default (if any) will be used.
- If the cookie option is defined this will automaticaly enable the cookie param for each backend server (See Backend Server Syntax guide)

The syntax is a bit flexible, per example you can use the following syntax.

```
- name: 'www'
  comment: 'HTTP Web backend for my app'
  mode: 'http'
  balance: 'roundrobin'
  options:
    - tcp-check
    - forwardfor
  cookie: { name: 'ServID', mode: 'insert', options: 'indirect' }
  backend_server_list:
    - { name: 'web1', address: '216.58.213.131', port: '80', weight: 10 }
    - { name: 'web2', address: '216.58.204.110', port: '80', maxconn: 256 }
```

This will be respectively translated to :

```
backend www
    # HTTP Web backend for my app
    mode http
    balance  roundrobin
    cookie ServID insert indirect
    option tcp-check
    option forwardfor
    server web1 216.58.213.131:80 weight 10
    server web2 216.58.204.110:80 maxconn 256
```


### Backend Cookie Syntax

Here is a recap of all possibles values for cookie

| Option        | Role                                | Default value | Possible values             |
|---------------|-------------------------------------|:-------------:|-----------------------------|
| name          | Cookie Name                         |      none     | String                      |
| mode          | Cookie Mode                         |      none     | rewrite \| insert \| prefix |
| options       | Option for cookie-based persistence |      none     | [Cookie](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#4-cookie) |


### Backend Server Syntax

Here is a recap of all possibles values for backend_server in backend_server_list in haproxy_backends configuration

| Option        | Role                                         | Default value       | Possible values  |
|---------------|----------------------------------------------|:-------------------:|------------------|
| name          | Backend server Name                          |        none         | String           |
| address       | IP Address of backend server                 |        none         | Single IP        |
| comment       | A comment escaped for HAProxy                |        none         | String           |
| cookie        | Customize value for backend cookie (When backend cookie is enabled) | backend server name | String |
| custom        | Any custom option not yet available with var |        none         | Any HAP option   |
| maxconn       | Maximum number of concurrent connections     |        none         | Number           |
| port          | Port number of backend server                |        none         | Port Number      |
| weight | Load is proportional to their weight relative to the sum of all weights | 1 | 0-256 [Weight](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#5.2-weight) |


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
