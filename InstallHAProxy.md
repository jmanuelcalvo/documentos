# Instalacion de HAProxy en Red Hat / CentOS

Instalacion del paquete HAProxy
```
[root@jmanuel ~]# yum install haproxy
```

En la configuracion mas basica, para realizar el balanceo del servicio HTTPD el archivo de configuracion puede quedar asi:
```
[root@jmanuel ~]# vim /etc/haproxy/haproxy.cfg
global
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    stats socket /var/lib/haproxy/stats

defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000

# [HTTP Site Configuration]
listen  http_web 172.16.132.1:80
        mode http
        balance roundrobin  # Load Balancing algorithm
        option httpchk
        option forwardfor
        server server1 172.16.132.234:80 weight 1 maxconn 512 check
        server server2 172.16.132.232:80 weight 1 maxconn 512 check

# [HTTPS Site Configuration]
listen  https_web 172.16.132.1:443
        mode tcp
        balance source# Load Balancing algorithm
        reqadd X-Forwarded-Proto: http
        server server1 172.16.132.234:443 weight 1 maxconn 512 check
        server server2 172.16.132.232:443 weight 1 maxconn 512 check
```

De donde la IPs:
```
172.16.132.1    -   Es la IP de la VM del balanceador por donde llegaran las peticiones
172.16.132.234  -   Son los servidores Web a donde llegaran las peticiones
172.16.132.232
```


Iniciar los servicios
```
[root@jmanuel ~]# systemctl start haproxy
```
        
        
        
