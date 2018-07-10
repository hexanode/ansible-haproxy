# Ansible Role: HAProxy

An Ansible Role that install and configure HAProxy.

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
haproxy_user:  haproxy
haproxy_group: haproxy

# Web Stats
haproxy_webstats: false                         # Set to true in order to enable webstats. (default is false)
haproxy_webstats_ip: '0.0.0.0'                  # Listening IP for the webstats interface
haproxy_webstats_port: '8080'                   # Listening Port for the webstats interface
haproxy_webstats_maxconn: '10'                  # Maximum per-process number of concurrent connections
haproxy_webstats_refreshtime: '30s'             # Enable automatic refresh with a delay
haproxy_webstats_realm: 'HAProxy\ Statistics'   # Authentication Realm
haproxy_webstats_username: 'admin'              # Authentication Username
haproxy_webstats_password: 'password'           # Authentication Password
haproxy_webstats_uri: '/haproxy?stats'          # Authentication URL
haproxy_webstats_admin_opt: 'if FALSE'          # Manage admin level if/unless a condition is matched (ex: if TRUE / if LOCALHOST)
# haproxy_webstats_scope: '.'                   # Scope for this webstat interface

# Frontends
haproxy_frontends_list: []

# Backends
haproxy_backends_list: []
```

## Dependencies

none


## Frontends Syntax

Here is a recap of all possible values for haproxy_frontends in haproxy_frontends_list.

| Option          | Role                            | Required | Default value | Type                                |
|-----------------|---------------------------------|:--------:|:-------------:|-------------------------------------|
| name            | Frontend Name                   |   true   |      none     | String                              |
| comment         | A comment escaped for HAProxy   |   false  |      none     | String                              |
| bind            | Bind frontend with interface    |   true   |      none     | IP Address or * for any             |
| port            | Bind frontend with a port on if |   true   |      none     | Port number (1-65535)               |
| mode            | Protocol used for the instance  |   false  |      tcp      | [Mode](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#4-mode) |
| options         | List of options                 |   false  |      none     | List (All options available for frontend) |
| default_backend | Default backend used            |   true   |      none     | String                              |

**Note :**
- If you omit a value the default (if any) will be used.

The syntax is a bit flexible, per example you can use the following syntax.

```
- { name: 'default', bind: '*', port: '80', options: [], default_backend: 'www' },
- { name: 'example', bind: '93.184.216.34', port: '8080', options: [], default_backend: 'example_backend' }
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

| Option              | Role                                    | Required | Default value  | Type                               |
|---------------------|-----------------------------------------|:--------:|:--------------:|------------------------------------|
| name                | Frontend Name                           |   true   |      none      | String                             |
| check_all_servers   | Manage check by default for all servers |   false  |      none      | Boolean (true\|false)              |
| cookie              | Cookie definition for backend           |   false  |      none      | Dict [See Backend Cookie Syntax guide](#backend-cookie-syntax) |
| mode                | Protocol used for the instance          |   false  |      tcp       | [Mode](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#4-mode) |
| balance             | Algorithm used to load balance          |   false  |   roundrobin   | [Balance](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#4.2-balance) |
| options             | List of options                         |   false  |      none      | List (All options available for backend) |
| backend_server_list | List of backend servers                 |   true   |      none      | List [See Backend Server Syntax guide](#backend-server-syntax) |

**Note :**
- If an option is omitted the default (if any) will be used.
- If the cookie option is defined this will automaticaly enable the cookie param for each backend server (See Backend Server Syntax guide)

The syntax is a bit flexible, per example you can use the following syntax.

```
name: 'www'
comment: 'HTTP Web backend for my app'
mode: 'http'
balance: 'roundrobin'
options:
  - tcp-check
  - forwardfor
check_all_servers: true
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
    server web1 216.58.213.131:80 weight 10 cookie web1 check
    server web2 216.58.204.110:80 maxconn 256 cookie web2 check
```


### Backend Cookie Syntax

Here is a recap of all possibles values for cookie. The cookie variable for a backend is a list of dictionary

| Option        | Role                                | Required | Default value | Type                        |
|---------------|-------------------------------------|:--------:|:-------------:|-----------------------------|
| name          | Cookie Name                         |   true   |      none     | String                      |
| mode          | Cookie Mode                         |   true   |      none     | rewrite \| insert \| prefix |
| options       | Option for cookie-based persistence |   true   |      none     | [Cookie](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#4-cookie) |


### Backend Server Syntax

Here is a recap of all possibles values for backend_server in backend_server_list in haproxy_backends configuration

| Option        | Role                                         | Required | Default value       | Type             |
|---------------|----------------------------------------------|:--------:|:-------------------:|------------------|
| name          | Backend server Name                          |   true   |        none         | String           |
| address       | IP Address of backend server                 |   true   |        none         | Single IP Address |
| check         | Manage check for this server (This override the check_all_servers in backend) | false | none | Boolean (true\|false) |
| comment       | A comment escaped for HAProxy                |   false   |        none         | String           |
| cookie        | Customize value for backend cookie (When backend cookie is enabled) | false | backend server name | String |
| custom        | Any custom option not yet available with var |   false   |        none         | String Any HAP option |
| maxconn       | Maximum number of concurrent connections     |   false   |        none         | Number           |
| port          | Port number of backend server                |   true   |        none         | Port number (1-65535) |
| weight | Load is proportional to their weight relative to the sum of all weights | false | 1 | 0-256 [Weight](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#5.2-weight) |


## Example Playbook

How to use the role in your ansible playbook.

**Playbook**

```
- name: Playbook Task to install haproxy role
  hosts: all (Group of servers on which you want to install and configure haproxy)
  roles:
    - { role: haproxy, tags: [ 'haproxy' ] }
```

**Variables**

```
haproxy_webstats: true					# Enable webstats
haproxy_webstats_admin_opt: 'if TRUE'   # Enable admin level for webstats interface

haproxy_frontends_list:
  - { name: 'default', mode: 'http', bind: '*', port: '80', options: [], default_backend: 'www' }

haproxy_backends_list:
  - name: 'www'
    comment: 'HTTP Web backend for my app'
    mode: 'http'
    balance: 'roundrobin'
    options:
      - tcp-check
      - forwardfor
    check_all_servers: true
    cookie: { name: 'ServID', mode: 'insert', options: 'indirect' }
    backend_server_list:
      - { name: 'web1', address: '216.58.213.131', port: '80', weight: 10 }
      - { name: 'web2', address: '216.58.204.110', port: '80', maxconn: 256 }
```


## ToDo


## License

GPLv3


## Author Information

This role is maintained by maximiliend. Issues and Merge Request are welcome.
