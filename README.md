# consul-fabio-integration
consul and fabio make it easy small container orchestration.

## Overview

consul provide service discovery and own cluster networking.
fabio detect application services from consul cluster networking.

consul connect provide service mesh.
but, I guess, This solution is more excelent.

look like netflix eureka and zuul.
therefore, this solution is able to use non java applications and docker container.

## Features

consul and fabio provide following features.

* Service Discovery
* Services Self Registration to fabio 
* API Gateway

## Installation

### Consul

[Install Consul](https://www.consul.io/docs/install/index.html)

### Fabio

fabio install
```
# curl -O https://github.com/fabiolb/fabio/releases/download/v1.5.10/fabio-1.5.10-go1.11.1-linux_amd64
# mv fabio-1.5.10-go1.11.1-linux_amd64 fabio
# chmod 755 fabio
# mv fabio /usr/lcocal/bin
```

To start fabio run
```
# fabio
```

or use custom configuration file.
```
# fabio -cfg fabio.properties
```

fabio default http listner port is '0.0.0.0:9990' and web ui listener port '0.0.0.0:9998'

## Servers

Runnging Processes and Lister Port into VM01.

```
=======================
|        VM01         |
=======================
|  proc  :    port    |
=======================
| Consul : 8500, 8600 |
| fabio  : 9999, 9998 |
| Nginx  : 80         |
=======================
```

Runnging Processes and Lister Port into VM02.
```
=======================
|        VM02         |
=======================
|  proc  :    port    |
=======================
| Consul : 8500, 8600 |
| APP    : 5000       |
=======================
```

Runnging Processes and Lister Port into VM03.
```
=======================
|        VM03         |
=======================
|  proc  :    port    |
=======================
| Consul : 8500, 8600 |
| APP    : 4567       |
=======================
```

## Consul Server configuration

following step common procedure to all vm[host].

create consul configuration file.
```
# mkdir -p /etc/consul.d
# touch /etc/consul.d/config.json
# vi /etc/consul.d/config.json
```

/etc/consul.d/config.json
```
{
    "bootstrap_expect": 3,
    "client_addr": "0.0.0.0",
    "datacenter": "hoge-datacenter",
    "data_dir": "/var/consul",
    "domain": "consul",
    "enable_script_checks": true,
    "dns_config": {
        "enable_truncate": true,
        "only_passing": true
    },
    "enable_syslog": false,
    "encrypt": "XXXXXXXXXXXXXXXXXXXXXXXXXXXX",
    "leave_on_terminate": true,
    "log_level": "INFO",
    "rejoin_after_leave": true,
    "server": true,
    "start_join": [
        "<YOUR VM01[HOST01] IPADDERSS>",
        "<YOUR VM02[HOST02] IPADDERSS>",
        "<YOUR VM03[HOST03] IPADDERSS>"
    ],
    "ui": true,
    "connect": {
        "enabled": true
    }
}
```

create unit file to consul. 
```
# touch /etc/systemd/system/consul.service
# vi /etc/systemd/system/consul.service
```

/etc/systemd/system/consul.service
```
[Unit]
Description="HashiCorp Consul - A service mesh solution"
Documentation=https://www.consul.io/
Requires=network-online.target
After=network-online.target

[Service]
User=root
Group=root
ExecStart=/usr/local/bin/consul agent -config-dir=/etc/consul.d/ -bind=<YOUR VM[HOST] IPADDERSS>
ExecReload=/usr/local/bin/consul reload
KillMode=process
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

consul start on boot and starting now.
```
# systemctl enable consul.service
# systemctl start consul.service
```

## Register Consul Service Configuration

this configuratoin file create per service.
it is neccesary to register middleware or own my created service[application] into consul cluster and networking.

### VM01

nginx service register consul cluster.

create service definition file.
```
# touch /etc/consul.d/nginx-service.json
# vi /etc/consul.d/nginx-service.json
```

```
{
    "service": {
      "id": "nginx",
      "name": "nginx",
      "check": {
        "id": "api",
        "name": "HTTP API on port 80",
        "http": "http://localhost:80/",
        "tls_skip_verify": false,
        "method": "GET",
        "interval": "30s",
        "timeout": "5s"
      },
      "port": 80,
      "tags": ["urlprefix-/"]
    }
  }
```

It is nessesary to reload add configuration files.
```
# consul reload
```

### VM02

my python flask rest api application register consul cluster.

create service definition file.
```
# touch /etc/consul.d/page-service.json
# vi /etc/consul.d/page-service.json
```

```
{
    "service": {
      "id": "page",
      "name": "page",
      "check": {
        "id": "api",
        "name": "HTTP API on port 5000",
        "http": "http://localhost:5000/ping",
        "tls_skip_verify": false,
        "method": "GET",
        "interval": "30s",
        "timeout": "5s"
      },
      "port": 5000,
      "tags": ["urlprefix-/page"]
    }
  }
```

execute reload command.
```
# consul reload
```

### VM03

my python flask rest api application register consul cluster.

create service definition file.
```
# touch /etc/consul.d/page-service.json
# vi /etc/consul.d/page-service.json
```

```
{
    "service": {
      "id": "page",
      "name": "page",
      "check": {
        "id": "api",
        "name": "HTTP API on port 4567",
        "http": "http://localhost:4567/hoge",
        "tls_skip_verify": false,
        "method": "GET",
        "interval": "30s",
        "timeout": "5s"
      },
      "port": 4567,
      "tags": ["urlprefix-/hoge"]
    }
  }
```

execute reload command.
```
# consul reload
```

## Start Services[Application]

VM01
```
# docker run nginx -d
```

VM02
```
# python /opt/service/page.py
```

VM03
```
# python /opt/service/hoge.py
```

## Magic

Fabio read "port" and "tags" values into consul cluster.
therefore, it is enable to redirect backend services.
fabio listen http listener port 9999.

/etc/consul.d/page-service.json

``
"port": 5000,
"tags": ["urlprefix-/page"]
``

/etc/consul.d/hoge-service.json

``
"port": 4567,
"tags": ["urlprefix-/hoge"]
``

redirect vm01 to vm02
```
VM01 : http://<VM01 IPADDRESS>:9999/page
⇒　redirect
VM02 : http://<VM02 IPADDRESS>:5000/page
```

redirect vm01 to vm03
```
VM01 : http://<VM01 IPADDRESS>:9999/hoge
⇒　redirect
VM03 : http://<VM03 IPADDRESS>:4567/hoge
```

## WEB UI

Consul and Fabio provide WEB UI Console.
It is enable to confirm registerd services and added routing.

### Consul

http://[VM01 IPAddress]:8500

![consul](./image/consul.png)

### Fabio

http://[VM01 IPAddress]:9998

![fabio](./image/fabio.png)

## REST API

Confirm registerd servce using REST API on VM01
```
# curl -s http://localhost:9998/api/routes
[
    {
        "service":"page",
        "host":"",
        "path":"/page",
        "src":"/page",
        "dst":"http://172.20.92.12:5000/",
        "opts":"",
        "weight":1,
        "cmd":"route add",
        "rate1":0,
        "pct99":0
    },
    {
        "service":"hoge",
        "host":"",
        "path":"/hoge",
        "src":"/hoge",
        "dst":"http://172.20.92.13:4567/",
        "opts":"",
        "weight":1,
        "cmd":"route add",
        "rate1":0,
        "pct99":0
    },
    {
        "service":"nginx",
        "host":"",
        "path":"/",
        "src":"/",
        "dst":"http://172.20.92.11:80/",
        "opts":"",
        "weight":1,
        "cmd":"route add",
        "rate1":0,
        "pct99":0
    }
]
```

## Confirm

Route Nginx
```
# curl http://localhost:9999
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Route page service
```
# curl http://localhost:9999/page
{ message: "page" }
```

Route hoge service
```
# curl http://localhost:9999/hoge
{ message : "hoge"}
```

## House Keeping

Consul Cluster persist state data into /var/consul.
So, It is careful to be not disk full situation.

```
# systemctl stop cousul
# cd /var/consul/
# rm -rf ./*
# systemctl start consul
```

## Refer

[Consul WEB Site](https://www.consul.io/)

[Fabio WEB Site](https://fabiolb.net/)

[Fabio Github Site](https://github.com/fabiolb/fabio)


