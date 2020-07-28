# Instalación de HAProxy en Red Hat / CentOS con Statics

Instalacion del paquete HAProxy
```
[root@jmanuel ~]# yum install haproxy
```

En la configuración mas básica, para realizar el balanceo de un servicio HTTP y HTTPS y habilitar las estadísticas el archivo de configuración puede quedar asi:
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

# [Enable Site Statis]
listen  stats   172.16.132.234:1936
        mode            http
        log             global
        maxconn 10
        stats enable
        stats hide-version
        stats refresh 30s
        stats show-node
        stats auth admin:password
        stats uri  /haproxy?stats
    
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

Algunos de los parámetros mas comunes a tener en cuenta son:
```
nbproc <value> # Número de cores de procesamiento en su maquina.
mode <value> # ‘http’ para sitios http y ‘tcp’ para sitios con https
balance <value> # Tipos de balanceo como ‘source’, ’roundrobin’ etc.
```

Habilitar el puerto 1936 en SELinux para visualizar las estadísticas
```
[root@jmanuel ~]# semanage port -a -t http_port_t -p tcp 1936
```

Iniciar y habilitar los servicios
```
[root@jmanuel ~]# systemctl start haproxy
[root@jmanuel ~]# systemctl enable haproxy
```
        
Realizar las pruebas de balanceo
```
[root@jmanuel ~]# curl  http://172.16.132.234/
La direccion IPv4 del servidor es 172.16.132.234 

[root@jmanuel ~]# curl  http://172.16.132.234/
La direccion IPv4 del servidor es 172.16.132.232
```

Visualizar las estadísticas
http://172.16.132.234:1936/haproxy?stats
usuario: admin
password: password
Seteados en el archivo de configuración principal

![Statics](images/statics.png)


A través de la herramienta ab (apache benckmark) la cual se encuentra en el paquete RPM (httpd-tools) podemos realizar pruebas de conexión al servidor para generar trafico
```
[root@jmanuel ~]# ab -n 100 -c 10 http://172.16.132.234/
```
Supongamos que queremos ver qué tan rápido nuestro sitio puede manejar 100 solicitudes, con un máximo de 10 solicitudes ejecutándose simultáneamente
