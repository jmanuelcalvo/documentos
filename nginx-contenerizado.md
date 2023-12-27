# NGINX Contenerizado con Podman

Para este ejemplo, se realizara el despliegue de 2 servidores basicos web, los cualese simularan que se muestra paginas con dominio.com y otro dominio.org, y la idea es que el NGINX pueda realizar el proxy reverso

### Preparacion del sistema operativo
```
[root@rhel9 ~]# subscription-manager  register
Registering to: subscription.rhsm.redhat.com:443/subscription
Username: jcalvo@redhat.com
Password:
The system has been registered with ID: 39256db1-bab6-431c-99bf-8b1880ee0754
The registered system name is: rhel9.jmanuelcalvo.com

[root@rhel9 ~]# subscription-manager  auto-attach
Auto-attach preference: enabled
[root@rhel9 ~]# subscription-manager  attach
Installed Product Current Status:
Product Name: Red Hat Enterprise Linux for ARM 64
Status:       Subscribed

[root@rhel9 ~]# yum install podman
Updating Subscription Management repositories.
Red Hat Enterprise Linux 9 for ARM 64 - BaseOS (RPMs)                                                                                           13 MB/s |  16 MB     00:01
Red Hat Enterprise Linux 9 for ARM 64 - AppStream (RPMs)                                                                                        20 MB/s |  25 MB     00:01
...
...
Installed:
  aardvark-dns-2:1.7.0-1.el9.aarch64           conmon-2:2.1.8-1.el9.aarch64           container-selinux-3:2.221.0-1.el9.noarch       containers-common-2:1-55.el9.aarch64
  criu-3.18-1.el9.aarch64                      criu-libs-3.18-1.el9.aarch64           crun-1.8.7-1.el9.aarch64                       fuse-common-3.10.2-6.el9.aarch64
  fuse-overlayfs-1.12-1.el9.aarch64            fuse3-3.10.2-6.el9.aarch64             fuse3-libs-3.10.2-6.el9.aarch64                libnet-1.2-6.el9.aarch64
  libslirp-4.4.0-7.el9.aarch64                 netavark-2:1.7.0-2.el9_3.aarch64       podman-2:4.6.1-7.el9_3.aarch64                 protobuf-c-1.3.3-13.el9.aarch64
  shadow-utils-subid-2:4.9-8.el9.aarch64       slirp4netns-1.2.1-1.el9.aarch64        yajl-2.1.0-22.el9.aarch64

Complete!
```

### Descargar las imagenes de contenedor de Podman
```
[root@rhel9 ~]# podman pull docker.io/library/httpd
Trying to pull docker.io/library/httpd:latest...
...
...
Writing manifest to image destination
b1866dff9b2714b36aa43e16cc64dbd2e0f3334b2e07884ffbb5eb438f458dac

[root@rhel9 ~]# podman pull docker.io/library/nginx
Trying to pull docker.io/library/nginx:latest...
...
...
Writing manifest to image destination
8aea65d81da202cf886d7766c7f2691bb9e363c6b5d9b1f5d9ddaaa4bc1e90c2

[root@rhel9 ~]# podman images
REPOSITORY               TAG         IMAGE ID      CREATED       SIZE
docker.io/library/nginx  latest      8aea65d81da2  2 months ago  196 MB
docker.io/library/httpd  latest      b1866dff9b27  2 months ago  199 MB

```

### Creacion de las carpetas locales para almacenar las configuraciones especificas de srvweb .org y .com
```
[root@rhel9 ~]# mkdir syscom sysorg
[root@rhel9 ~]# ls
anaconda-ks.cfg  et-hostname  syscom  sysorg
```
- Creacion de los archivos index.html que se mostraran en cada uno de los servidores web
```
[root@rhel9 ~]# cat << EOF > ./syscom/index.html
<html>
  <header>
    <title>SysAdmin.com</title>
  </header>
  <body>
    <p>This is the SysAdmin website hosted on the .com domain</p>
  </body>
</html>
EOF
```

```
[root@rhel9 ~]# cat << EOF > ./sysorg/index.html
<html>
  <header>
    <title>SysAdmin.org</title>
  </header>
  <body>
    <p>This is the SysAdmin website hosted on the .org domain</p>
  </body>
</html>
EOF
[root@rhel9 ~]#
```
### Ejecucion de los contenedores web con sus respectivas paginas web
* Contenedor servidor web que responde al dominio .com 
```
[root@rhel9 ~]# podman run --name=syscom -p 8080:80 -v $HOME/syscom:/usr/local/apache2/htdocs:Z -d docker.io/library/httpd
386178f1104289ace5435b66ac8e244fedfa93b1ad8358f52f673ee3aa3a63cd
```
* Contenedor servidor web que responde al dominio .org
```
[root@rhel9 ~]# podman run --name=sysorg -p 8081:80 -v $HOME/sysorg:/usr/local/apache2/htdocs:Z -d docker.io/library/httpd
c952d8041636aa926223ee372e7a97d327ef2e138dfdede482ddad13946a2e7e

[root@rhel9 ~]# podman ps
CONTAINER ID  IMAGE                           COMMAND           CREATED         STATUS         PORTS                 NAMES
386178f11042  docker.io/library/httpd:latest  httpd-foreground  13 seconds ago  Up 13 seconds  0.0.0.0:8080->80/tcp  syscom
c952d8041636  docker.io/library/httpd:latest  httpd-foreground  5 seconds ago   Up 6 seconds   0.0.0.0:8081->80/tcp  sysorg
```

### Para realizar las pruebas de los servidores web y sus respuestas, se debe incluir en el archivo `/etc/hosts` de la maquina local el siguiente contendico con la IP del la maquina local
```

[root@rhel9 ~]# ip a s enp0s1
2: enp0s1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 0a:30:cd:73:f2:d7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.64.9/24 brd 192.168.64.255 scope global dynamic noprefixroute enp0s1
       valid_lft 85342sec preferred_lft 85342sec

[root@rhel9 ~]# vi /etc/hosts
...
...
192.168.64.9    sysadmin.com
192.168.64.9    sysadmin.org
```
* Pruebas de conexion por el puerto 8080  - dominio.com
```
[root@rhel9 nginx]# curl http://sysadmin.com:8080
<html>
  <header>
    <title>SysAdmin.com</title>
  </header>
  <body>
    <p>This is the SysAdmin website hosted on the .com domain</p>
  </body>
</html>
```

* Pruebas de conexion por el puerto 8081 - dominio.org
```
[root@rhel9 nginx]# curl http://sysadmin.org:8081
<html>
  <header>
    <title>SysAdmin.org</title>
  </header>
  <body>
    <p>This is the SysAdmin website hosted on the .org domain</p>
  </body>
</html>
```


## ¡Ahora comienza la magia del proxy inverso con Nginx!


### Creacion de las carpetas locales para almacenar las configuraciones del nginx
Dentro de este directorio, se crean 3 archivos:
El archivo default.conf, que contiene la configuración predeterminada de Nginx. 
El archivo syscom.conf, que contiene la configuración de la aplicación sysadmin.com 
El archivo sysorg.conf, que contiene la configuración de la aplicación sysadmin.org

Archivo default.conf
```
[root@rhel9 nginx]# cat << EOF > default.conf
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

}
EOF
```
El archivo syscom.conf
NOTA: Tenga en cuenta que la IP debe ser reemplazada por la IP de su maquina local, en mi caso 192.168.64.9
Recuerde que para el dominio.com, se dejo configurado en contenedor con el puerto 8080 respondiendo al dominio sysadmin.com
```
[root@rhel9 nginx]# cat << EOF > syscom.conf
server {
  listen 80;
  server_name sysadmin.com;

  location / {
    proxy_pass http://192.168.64.9:8080;
  }
}
EOF
```

El archivo sysorg.conf
NOTA: Tenga en cuenta que la IP debe ser reemplazada por la IP de su maquina local, en mi caso 192.168.64.9
Recuerde que para el dominio.com, se dejo configurado en contenedor con el puerto 8081 respondiendo al dominio sysadmin.org
```
[root@rhel9 nginx]# cat << EOF > sysorg.conf
server {
  listen 80;
  server_name sysadmin.org;

  location / {
    proxy_pass http://192.168.64.9:8081;
  }
}
EOF
```

Para que esto funcione es necesario aprovechar el parámetro include /etc/nginx/conf.d/*.conf del archivo /etc/nginx/nginx.conf dentro del contenedor Nginx, que permite cargar archivos de configuración modulares dentro desde /etc/ directorio nginx/conf.d/. Cuando se ejecute, estos archivos se montarán en este directorio dentro del contenedor.

### Ejecucion del contenedor de Ngnix por el puerto 80

```
[root@rhel9 nginx]# podman run --name=nginx -p 80:80 -v $HOME/nginx:/etc/nginx/conf.d:Z -d docker.io/library/nginx
d8c24df13fe957375104d055938cc9c8dfce08d2d8cb5eed7e604c7482fa8a9f
```

NOTA: 
En caso que se genere error de puerto al intentar ejecutar el contenedor, se puede ejecutar los siguietnes comandos:
```
podman run --name=nginx -p 80:80 -v $HOME/nginx:/etc/nginx/conf.d:Z -d docker.io/library/nginx
Error: rootlessport cannot expose privileged port 80, you can add 'net.ipv4.ip_unprivileged_port_start=80' to /etc/sysctl.conf (currently 1024), or choose a larger port number (>= 1024): listen tcp 0.0.0.0:80: bind: permission denied

sysctl net.ipv4.ip_unprivileged_port_start=80

```

### Habiliatr el puerto 80 en el firewall de la maquina local
```
[root@rhel9 nginx]# firewall-cmd --add-service=http --permanent
success
[root@rhel9 nginx]# firewall-cmd --add-service=http
success
```

### Realizar las pruebas de conectividad y redireccion
```
[root@rhel9 nginx]# curl http://sysadmin.com:80
<html>
  <header>
    <title>SysAdmin.com</title>
  </header>
  <body>
    <p>This is the SysAdmin website hosted on the .com domain</p>
  </body>
</html>


[root@rhel9 nginx]# curl http://sysadmin.org:80
<html>
  <header>
    <title>SysAdmin.org</title>
  </header>
  <body>
    <p>This is the SysAdmin website hosted on the .org domain</p>
  </body>
</html>
[root@rhel9 nginx]#
```



Source: https://www.redhat.com/sysadmin/podman-nginx-multidomain-applications
