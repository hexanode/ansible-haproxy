# Ansible Role: HAProxy

An Ansible Role that install and configure HAProxy.

We want this role as simple, configurable and interoperable as possible.

## Role behavior

This ansible role install and configure HAProxy.

This software can manage, global configurations, statistic panel, frontends, backends with all common and custom configuration.

## Requirements

Compliant with :
- Debian 8 (Jessie)
- Debian 9 (Stretch)
- Debian 10 (Buster)

Other:
- Fact gathering should be allowed in ansible-playbook (By default)
- The role require the superuser privileges. The task should be used with remote_user root or with a sudo/su grant
- openssl package should be present on the remote host if using the self signed ssl certificate

## Role Variables

Available variables with defaults values are defined in `defaults/main.yml`.

Modifiables variables and possible values are listed below :

```
# Security
haproxy_chroot: '/var/lib/haproxy'

# SSL
haproxy_generate_self_signed_cert: true         # Set to false in order to disable self signed ssl certificate generation in /etc/haproxy/ssl/combined.pem

# User & Group for HAProxy
haproxy_user:  haproxy
haproxy_group: haproxy

# Defaults
haproxy_timeout_connect: 6000                   # Timeout connect
haproxy_timeout_client:  60000                  # Timeout client
haproxy_timeout_server:  60000                  # Timeout server
haproxy_defaultdhparam:  2048                   # Default ssl dh parameter
haproxy_master_worker_mode: true                # Enable Master-worker mode. Only available with HAProxy 1.8 (on Debian 10)

# Logging
haproxy_logrotate_period: daily                 # Logrotate rotation period, you can use all logrotate configuration (daily, weekly, ...)
haproxy_logrotate_amount: 21                    # Logrotate amount of rotation to keep
haproxy_logrotate_maxsize: '500M'               # Logrotate maximum size for log file before rotating

# Custom errors pages
haproxy_custom_errors_pages: true               # Set to false to disable copy of custom error pages

# Web Stats
haproxy_webstats: false                         # Set to true in order to enable webstats. (default is false)
haproxy_webstats_ip: '0.0.0.0'                  # Listening IP for the webstats interface
haproxy_webstats_port: '8080'                   # Listening Port for the webstats interface
haproxy_webstats_maxconn: '10'                  # Maximum per-process number of concurrent connections
haproxy_webstats_refreshtime: '30s'             # Enable automatic refresh with a delay
haproxy_webstats_realm: 'HAProxy\ Statistics'   # Authentication Realm
haproxy_webstats_username: 'admin'              # Authentication Username
haproxy_webstats_password: 'password'           # Authentication Password
haproxy_webstats_uri: '/haproxy?stats'          # Authentication URL
haproxy_webstats_admin_opt: 'if FALSE'          # Manage admin level if/unless a condition is matched (ex: if TRUE / if LOCALHOST)
# haproxy_webstats_scope: '.'                   # Scope for this webstat interface
# haproxy_webstats_crt: '/path/to/crt.pem'      # Enable SSL by providing an absolute path to the combined certificate and key pem file

# Frontends
haproxy_frontends_list: []

# Backends
haproxy_backends_list: []

# Certificates list
haproxy_crt_list: []
```

## Dependencies

none

## Frontends Syntax

Here is a recap of all possible values for haproxy_frontends in haproxy_frontends_list.

| Option          | Role                            | Required | Default value | Type                                |
|-----------------|---------------------------------|:--------:|:-------------:|-------------------------------------|
| name            | Frontend Name                   |   true   |      none     | String                              |
| bind            | Bind frontend with interface(s) |   true   |      none     | Dict or List [See Frontend Bind Syntax guide](#frontend-bind-syntax) |
| comment         | A comment escaped for HAProxy   |   false  |      none     | String                              |
| custom          | A list of custom config lines   |   false  |      none     | List (Any frontend configuration not managed by this role) |
| acl | A list of acl conf lines | false | none | [ACL](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#4.2-acl) |
| redirect | A list of redirect conf lines | false | none | [ACL](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#4.2-acl) |
| use_backend | A list of use_backend conf lines | false | none | [ACL](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#4.2-acl) |
| mode | Protocol used for the instance | false | tcp | [Mode](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#4-mode) |
| options         | List of options                 |   false  |      none     | List (All options available for frontend) |
| timeouts        | List of timeouts                |   false  |      none     | List (All timeouts available for frontend) |
| default_backend | Default backend used            |   true   |      none     | String                              |

**Note :**
- If you omit a value the default (if any) will be used.

The syntax is flexible, per example you can use the following syntax :

```
haproxy_frontends_list:
  - name: 'default'
    bind: { address: '*', port: '80' }
    default_backend: 'www'
  - name: 'example'
    mode: tcp
    bind: { address: '93.184.216.34', port: '514' }
    default_backend: 'app'
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
| crt     | Enable SSL for this bind by defining crt | false | none | Absolute path to pem combined (certificate + key) file [crt](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#crt) |
| crt_list | Enable SSL for this bind by defining crt_list | false | none | Absolute path to a certificate list file [crt-list](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#crt-list) Ansible generated is '/etc/haproxy/crt-list.txt' |
| disable_http2 | Disable HTTP/2 for this bind | false | false | Boolean |

**Note :**
- On Debian 10, HAProxy 1.8 is used. So HTTP/2 is automaticaly deployed when port 443 is binded. If you want to disable this default parameter, you can set true disable_http2

Example :

```
# Single dictionary :
bind: { address: '*', port: '80' }

# List of dictionary :
bind:
  - { address: '*', port: '80' }
  - { port: '443', crt: '/etc/haproxy/ssl/combined.pem' }
  # or
  - { port: '443', crt_list: '/etc/haproxy/crt-list.txt' }
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
| mode          | Cookie Mode                         |   true   |      none     | rewrite \| insert \| prefix |
| options       | Option for cookie-based persistence |   true   |      none     | [Cookie](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#4-cookie) |


### Backend Server Syntax

Here is a recap of all possibles values for backend_server in backend_server_list in haproxy_backends configuration

| Option        | Role                                         | Required | Default value       | Type             |
|---------------|----------------------------------------------|:--------:|:-------------------:|------------------|
| name          | Backend server Name                          |   true   |        none         | String           |
| address       | IP Address of backend server                 |   true   |        none         | Single IP Address |
| check         | Manage check for this server (This override the check_all_servers in backend) | false | none | Boolean (true\|false) |
| comment       | A comment escaped for HAProxy                |   false   |        none         | String           |
| cookie        | Customize value for backend cookie (When backend cookie is enabled) | false | backend server name | String |
| custom        | Any custom option not yet available with var |   false   |        none         | String Any HAP option |
| maxconn       | Maximum number of concurrent connections     |   false   |        none         | Number           |
| port          | Port number of backend server                |   true   |        none         | Port number (1-65535) |
| weight | Load is proportional to their weight relative to the sum of all weights | false | 1 | 0-256 [Weight](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#5.2-weight) |


### Certificates list Syntax

Here is a recap of all possibles values for haproxy_crt_list list. The haproxy_crt_list variable is a list of dictionary

| Option    | Role                               | Required | Default value | Type   |
|-----------|------------------------------------|:--------:|:-------------:|-----------------------------|
| snifilter | sni filter                         |   true   |      none     | String |
| path      | Path of certificate on remote host |   false  |      none     | [SNI Filter](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#5.1-crt-list) |

Example :

```
haproxy_crt_list:
  - { snifilter: 'app.example.com', path: '/etc/haproxy/ssl/example.com.pem' }
  - { path: '/etc/haproxy/ssl/selfsigned.pem'}
```

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
    bind:
      - { address: '*', port: '80' }
      - { port: '443', crt: '/etc/haproxy/ssl/combined.pem' }
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


## Miscellaneous

This role generate by default a self signed certificate (for testing purposes or waiting for a real ssl certificate).
    - Files are generated in `/etc/haproxy/ssl/`
    - With selfsigned.crt as the certificate, selfsigned.key as the key and selfsigned as the combined file, wich is usefull for haproxy
    - You can disable this generation by setting the variable haproxy_generate_self_signed_cert to false `haproxy_generate_self_signed_cert: false`

Some customization can be done by overriding some variables present in defaults/main.yml like :
- `haproxy_timeout_connect`, `haproxy_logrotate_period`, etc.

## License

GPLv3


## Author Information

This role is maintained by maximiliend. Issues and Merge Request are welcome.
