# consul-fabio-integration
consul and fabio make it easy small container orchestration.

## Overview

consul provide service discovery and own cluster networking.
fabio detect application services from consul cluster networking.

consul connect provide service mesh.
but, I guess, This solution is more excelent.

look like netflix eureka and zuul.
therefore, this solution is able to use non java applications and docker container.

fabio installation.
```
# curl -O <URL>
# mv ファイル名 fabio
# chmod 755 fabio
# mv fabio /usr/local/bin
```

fabio execution
```
# ./fabio
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
|         VM1         |
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
|         VM2         |
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
|         VM3         |
=======================
|  proc  :    port    |
=======================
| Consul : 8500, 8600 |
| APP    : 4567       |
=======================
```
