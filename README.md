# consul-fabio-integration
consul and fabio make it easy small container orchestration.

## Overview

consul provide service discovery and own cluster networking.
fabio detect application services from consul cluster networking.

consul connect provide service mesh.
but, I guess, This solution is more excelent.

look like netflix eureka and zuul.
therefore, this solution is able to use non java applications and docker container.

## Perpose

consul and fabio provide following features.

* Service Discovery
* Services Self Registration to fabio 
* API Gateway

## Installation

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

started fabio process, after connect consul cluster.
and discovery consul services.

We will beadding tiny logic into consul configuration files. this is magic.
```

```

Important, adding tag property urlprefix.

Consul Services into Other VM, 


Runnging Processes and Lister Port into VM1.

```
=======================
|        VM01         |
=======================
|  proc  :    port    |
=======================
| Consul : 8500, 8600 |
| fabio  : 9990, 9998 |
| Nginx  : 80         |
=======================
```

Runnging Processes and Lister Port into VM2.
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

Runnging Processes and Lister Port into VM3.
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

## WEB UI


### Consul

![consul](./image/consul.png)

### Fabio

![fabio](./image/fabio.png)

