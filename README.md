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
| custom          | A list of custom config lines   |   false  |      none     | List (Any frontend configuration not managed by this role) |
| bind            | Bind frontend with interface(s) |   true   |      none     | Dict or List [See Frontend Bind Syntax guide](#frontend-bind-syntax) |
| port            | Bind frontend with a port on if |   true   |      none     | Port number (1-65535)               |
| mode            | Protocol used for the instance  |   false  |      tcp      | [Mode](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#4-mode) |
| options         | List of options                 |   false  |      none     | List (All options available for frontend) |
| default_backend | Default backend used            |   true   |      none     | String                              |

**Note :**
- If you omit a value the default (if any) will be used.

The syntax is flexible, per example you can use the following syntax :

```
haproxy_frontends_list:
  - { name: 'default', bind: '*', port: '80', default_backend: 'www' },
  - { name: 'example', mode: tcp, bind: '93.184.216.34', port: '514', default_backend: 'app' }
```

This will be respectively translated to :

```
frontend default
    bind *:80
    mode http
    default_backend www

frontend example
    bind 93.184.216.34:514
    mode tcp
    default_backend app
```


### Frontend Bind syntax

You can define bind parameters for haproxy frontends in two way a single dictionary or a list of dictionary for multiples binds.

Here is a recap of all possible values for the bind dictionary :

| Option  | Role         | Required | Default value | Type                          |
|---------|--------------|:--------:|:-------------:|-------------------------------|
| address | Bind address |   false  |       *       | Single IP Address (* for any) |
| port    | Bind port    |   true   |      none     | Port number (1-65535)         |

Example :

```
# Single dictionary :
bind: { address: '*', port: '80' }

# List of dictionary :
bind:
  - { address: '*', port: '80' }
  - { port: '8080' }
```


## Backends Syntax

Here is a recap of all possible values for haproxy_backends in haproxy_backends_list.

| Option              | Role                                    | Required | Default value  | Type                               |
|---------------------|-----------------------------------------|:--------:|:--------------:|------------------------------------|
| name                | Frontend Name                           |   true   |      none      | String                             |
| check_all_servers   | Manage check by default for all servers |   false  |      none      | Boolean (true\|false)              |
| cookie              | Cookie definition for backend           |   false  |      none      | Dict [See Backend Cookie Syntax guide](#backend-cookie-syntax) |
| custom              | A list of custom config lines           |   false  |      none      | List (Any backend configuration not managed by this role) |
| mode                | Protocol used for the instance          |   false  |      tcp       | [Mode](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#4-mode) |
| balance             | Algorithm used to load balance          |   false  |   roundrobin   | [Balance](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#4.2-balance) |
| options             | List of options                         |   false  |      none      | List (All options available for backend) |
| backend_server_list | List of backend servers                 |   true   |      none      | List [See Backend Server Syntax guide](#backend-server-syntax) |

**Note :**
- If an option is omitted the default (if any) will be used.
- If the cookie option is defined this will automaticaly enable the cookie param for each backend server (See Backend Server Syntax guide)

The syntax is flexible, per example you can use the following syntax :

```
haproxy_backends_list:
  - name: 'www'
    comment: 'HTTP backend for my web app'
    mode: 'http'
    balance: 'roundrobin'
    options:
      - httpchk GET /
      - forwardfor
    check_all_servers: true
    cookie: { name: 'ServID', mode: 'insert', options: 'indirect' }
    backend_server_list:
      - { name: 'web1', address: '216.58.213.131', port: '80', maxconn: 256 }
      - { name: 'web2', address: '216.58.204.110', port: '80', maxconn: 256 }
      - { name: 'bkp1', address: '216.58.208.227', port: '80', cookie: 'web1', custom: 'backup' }
  - name: 'app'
    balance: 'roundrobin'
    options:
      - tcp-check
    custom:
      - stick-table type ip size 250k expire 30m
      - stick on src
    backend_server_list:
      - { name: 'app1', address: '10.0.10.1', port: '514', weight: 10 }
      - { name: 'app2', address: '10.0.10.2', port: '514', weight: 20 }
```

This will be respectively translated to :

```
backend www
    # HTTP backend for my web app
    mode http
    balance roundrobin
    cookie ServID insert indirect
    option httpchk GET /
    option forwardfor
    server web1 216.58.213.131:80 maxconn 256 cookie web1 check
    server web2 216.58.204.110:80 maxconn 256 cookie web2 check
    server bkp1 216.58.208.227:80 cookie web1 check backup

backend app
    mode tcp
    balance roundrobin
    option tcp-check
    stick-table type ip size 250k expire 30m
    stick on src
    server app1 10.0.10.1:514 weight 10
    server app2 10.0.10.2:514 weight 20
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

**Playbook example**

```
- name: Playbook Task to install haproxy role
  hosts: all (Group of servers on which you want to install and configure haproxy)
  roles:
    - { role: haproxy, tags: [ 'haproxy' ] }
```

**Variables example**

```
haproxy_webstats: true                  # Enable webstats
haproxy_webstats_admin_opt: 'if TRUE'   # Enable admin level for webstats interface

haproxy_frontends_list:
  - name: 'default'
    mode: 'http'
    bind: { port: '80' }
    port: '80'
    options: []
    default_backend: 'www'

haproxy_backends_list:
  - name: 'www'
    comment: 'HTTP backend for my web app'
    mode: 'http'
    balance: 'roundrobin'
    options:
      - httpchk GET /
      - forwardfor
    check_all_servers: true
    cookie: { name: 'ServID', mode: 'insert', options: 'indirect' }
    backend_server_list:
      - { name: 'web1', address: '216.58.213.131', port: '80', maxconn: 256 }
      - { name: 'web2', address: '216.58.204.110', port: '80', maxconn: 256 }
      - { name: 'bkp1', address: '216.58.208.227', port: '80', cookie: 'web1', custom: 'backup' }
  - name: 'app'
    balance: 'roundrobin'
    options:
      - tcp-check
    custom:
      - stick-table type ip size 250k expire 30m
      - stick on src
    backend_server_list:
      - { name: 'app1', address: '10.0.10.1', port: '514', weight: 10 }
      - { name: 'app2', address: '10.0.10.2', port: '514', weight: 20 }
```


## ToDo

- Manage SSL
- Manage use_backend
- Manage ACL

## License

GPLv3


## Author Information

This role is maintained by maximiliend. Issues and Merge Request are welcome.
