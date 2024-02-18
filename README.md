# Ansible Role For Nginx Setting

[![CI](https://github.com/Asya-kawai/ansible-role-nginx-with-naxsi/actions/workflows/ci.yml/badge.svg)](https://github.com/Asya-kawai/ansible-role-nginx-with-naxsi/actions/workflows?query=workflow%3ACI)

Setup nginx.

# Examples

Directory tree.

```
.
├── README.md
├── ansible.cfg
├── etc
│   └── nginx
│       ├── centos
│       │   ├── conf.d
│       │   │   ├── your-domain-example.com.conf
│       │   ├── html
│       │   │   └── index.html
│       │   ├── mime.types
│       │   └── nginx.conf
│       └── ubuntu
│           ├── conf.d
│           │   └── your-domain-example.com.conf
│           ├── html
│           │   └── index.html
│           ├── mime.types
│           └── nginx.conf
├── group_vars
│   ├── all.yml
│   ├── webserver_centos.yml
│   └── webserver_ubuntu.yml
├── inventory
└── webservers.yml
```

## Inventory file

```
[webservers:children]
webserver_ubuntu
webserver_centos

[webserver_ubuntu]
ubuntu ansible_host=10.10.10.10 ansible_port=22

[webserver_centos]
centos ansible_host=10.10.10.11 ansible_port=22
```

## Group Vars / Common settings(all.yml)

`all.yml` sets common variables.

```
# Common settings
become: yes
ansible_user: root

# Private_key is saved in local host only!
ansible_ssh_private_key_file: ""

# The path of nginx configuraiton files
nginx_conf_src_path: etc/nginx/ubuntu
```

## Group Vars / Ubuntu(webserver_ubuntu.yml)

`webserver_ubuntu.yml` is `webservers` host's children.
This role does NOT need to define any variables.

## Group Vars / CentOS(webserver_centos.yml)

`webserver_ubuntu.yml` is `webservers` host's children.
This role does NOT need to define any variables.

## Playbook / Webservers(webservers.yml)

```
- hosts: webservers
  become: yes
  module_defaults:
    apt:
      cache_valid_time: 86400
  roles:
    - user
```

## Nginx conf files

This role copy some config files(`nginx.conf`, `mime.types`, `conf.d/*` and `html/*`) in `/usr/loca/etc/nginx/`.

So when you want to copy these files to `ubuntu`(ubuntu is inventory's host name),
you should make `./etc/nginx/ubuntu/{nginx.conf, mime.types, conf.d/, html/}`.

```
etc/nginx/ubuntu/
├── conf.d
│   ├── your-domain-example.com.conf
├── html
│   └── index.html
├── mime.types
└── nginx.conf
```

The exmaple of nginx.conf.

```
user  nginx;
worker_processes  auto;

pid /var/log/nginx/nginx.pid;

events {
    worker_connections  1024;
}

http {
    lua_package_path  "/usr/local/lib/lua/?.lua;;";
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;

    server {
        listen       80;
        server_name  localhost;

        location / {
            root   html;
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

## Should define parameters:

* `user` : `nginx`
* `pid` : `/var/log/nginx/nginx.pid`
* `lua_package_path` : `/usr/local/lib/lua/?.lua;;`

# How to DryRun and Apply

DryRun

```
ansible-playbook -i inventory --private-key="~/.ssh/your_private_key" -CD webservers.yml --tags nginx
```

Apply

```
ansible-playbook -i inventory --private-key="~/.ssh/your_private_key" -D webservers.yml --tags nginx
```

# Troubesshooting

Nothing in current.

# How to test

Start VM instance kind of Ubuntu 18.04, Ubuntu 20.04, Ubuntu 22.04, CentOS7 and CentOS8 and
run ansible-playbook command such as `ansible-playbook -i inventory -CD playbook.yml --private-key ~/.ssh/your_private_key --tags nginx`.

## Notes:

* Not support CentOS9
* We give up on using `molecule` because sometime molecule are broken and don't work correctly.
