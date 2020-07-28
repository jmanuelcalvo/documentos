# Crear un DNS que soporte actualizaciones

Para un proyecto en el que me encontraba trabajando, requería que un playbook dentro de sus múltiples tareas, también pudiese lanzar la creación de un registro en el DNS.

El primer reto para esto fue montar un ambiente de pruebas que me permitiera primero realizar la tarea manual, es por esto voy a compartir el procedimiento de montar un servidor bind que permite actualizaciones


Este laboratorio lo realice sobre una maquina con RHEL 7.6

1. Instalar el paquete de bind y bind-utils
```
[root@server02 ~]# yum -y install bind bind-utils
```

2. Crear una llave de DNS con la que posteriormente se realizara la actualización
```
[root@server02 ~]# cd /tmp/
[root@server02 tmp]# dnssec-keygen -a HMAC-SHA512 -b 512 -n USER mrslave.jmanuelcalvo.com.
Kmrslave.jmanuelcalvo.com.+165+26666
```
Los contenidos deben ser similares a estos:
```
[root@server02 tmp]# more Kmrslave.jmanuelcalvo.com.+165+26666.key
mrslave.domain.tld. IN KEY 0 3 165 wrm1CZ36Us0bNHn9c4K9tfS2p3bl4kBnvB+wcUynrzxa0vmjopqBzJ0p smOaA3owSLRtzfwNZ6FsDAJNjvIWhw==
```

```
[root@server02 tmp]# more Kmrslave.jmanuelcalvo.com.+165+26666.private
Private-key-format: v1.3
Algorithm: 165 (HMAC_SHA512)
Key: wrm1CZ36Us0bNHn9c4K9tfS2p3bl4kBnvB+wcUynrzxa0vmjopqBzJ0psmOaA3owSLRtzfwNZ6FsDAJNjvIWhw==
Bits: AAA=
Created: 20200320203004
Publish: 20200320203004
Activate: 20200320203004
```

3. Crear un archivo que contenga la clave publica (wrm1C.....)
```
[root@server02 tmp]# vi /etc/bind-keys.conf
key mrslave.jmanuelcalvo.com. {
    algorithm HMAC-SHA512;
    secret "wrm1CZ36Us0bNHn9c4K9tfS2p3bl4kBnvB+wcUynrzxa0vmjopqBzJ0p smOaA3owSLRtzfwNZ6FsDAJNjvIWhw=="
};
```

4. Garantizar los permisos del archivo para que bind lo pueda visualizar
```
[root@server02 tmp]# chown  named /etc/bind-keys.conf
```

5. Realizar la configuración del bind, para ello lo que generalmente hago es que saco una copia del archivo original y copio desde 0 un archivo con el siguiente contenido.
```
[root@server02 tmp]# cd /etc
[root@server02 etc]# mv  named.conf named.conf.default
[root@server02 etc]# vim named.conf

options {
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        allow-query     { any; };
        recursion no;
        allow-transfer { none; };

};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "jmanuelcalvo.com" IN {
        type master;
        file "db.jmanuelcalvo.com.zone";
        allow-update { any; };
};

zone "132.16.172.in-addr.arpa" IN {
        type master;
        file "db.172.16.132.zone";
        allow-update { any; };
};

include "/etc/bind-keys.conf";


[root@server02 named]# chown  :named named.conf
```


6. Luego se deben crear las zonas, para lo que también copio el archivo desde 0
```
[root@server02 etc]# vi /var/named/db.jmanuelcalvo.com.zone
$TTL 86400      ; 1 day
jmanuelcalvo.com        IN SOA  ns1.jmanuelcalvo.com. root.jmanuelcalvo.com. (
                                2001102530 ; serial
                                10800      ; refresh (3 hours)
                                3600       ; retry (1 hour)
                                604800     ; expire (1 week)
                                86400      ; minimum (1 day)
                                )
                        NS      ns1.jmanuelcalvo.com.
                        A       172.16.132.245
                        MX      20 mail.jmanuelcalvo.com.
$ORIGIN jmanuelcalvo.com.
$TTL 600        ; 10 minutes
ext                     A       192.168.1.2
$TTL 86400      ; 1 day
ns1                     A       172.16.132.245
server02                A       172.16.132.245


[root@server02 ~]# vi /var/named/db.172.16.132.zone
$TTL 86400      ; 1 day
@               IN SOA  ns1.jmanuelcalvo.com. root.mail.jmanuelcalvo.com. (
                                2001102529 ; serial
                                10800      ; refresh (3 hours)
                                3600       ; retry (1 hour)
                                604800     ; expire (1 week)
                                86400      ; minimum (1 day)
                                )

                IN      NS      jmanuelcalvo.com.


245             IN      PTR     server02.jmanuelcalvo.com.
```

7. Garantizar los permisos de los archivos creados
```
[root@server02 etc]# cd /var/named
[root@server02 named]# chown  named:named db.*
```

8. Por ultimo reiniciar el servicio de Bind
```
[root@server02 named]# systemctl restart named
```



## Pruebas de funcionamiento


Si las pruebas se van a realizar desde la misma maquinas:

1. Garantizar que la maquina resuelva por la IP del DNS
```
[root@server02 ~]# vim /etc/resolv.conf
search jmanuelcalvo.com
nameserver 127.0.0.1
```

2. Garantizar que se resuelva correctamente de forma local
```
[root@server02 ~]# nslookup  server02.jmanuelcalvo.com
Server:		127.0.0.1
Address:	127.0.0.1#53

Name:	server02.jmanuelcalvo.com
Address: 172.16.132.245

[root@server02 ~]# nslookup 172.16.132.245
245.132.16.172.in-addr.arpa	name = server02.jmanuelcalvo.com.
```

3. Crear un archivo con la sintaxis de actualización de DNS con un contenido similar al siguiente
```
[root@server02 ~]# more nsupdate.txt
server server02.jmanuelcalvo.com
zone jmanuelcalvo.com
update delete ext.jmanuelcalvo.com A
update add ext.jmanuelcalvo.com 600 A 192.168.1.2
show
send
```

4. Realizar la prueba de creación a través del comando nsupdate
```
[root@server02 ~]# nsupdate -k /etc/bind-keys.conf -v nsupdate.txt
Outgoing update query:
;; ->>HEADER<<- opcode: UPDATE, status: NOERROR, id:      0
;; flags:; ZONE: 0, PREREQ: 0, UPDATE: 0, ADDITIONAL: 0
;; ZONE SECTION:
;jmanuelcalvo.com.		IN	SOA

;; UPDATE SECTION:
ext.jmanuelcalvo.com.	0	ANY	A
ext.jmanuelcalvo.com.	600	IN	A	192.168.1.2

[root@server02 ~]# nslookup ext.jmanuelcalvo.com.
Server:		127.0.0.1
Address:	127.0.0.1#53

Name:	ext.jmanuelcalvo.com
Address: 192.168.1.2
```

5. En caso de funcionar de forma correcta el comando, el resultado debería ser el nuevo registro en el DNS
```
[root@server02 ~]# nslookup ext.jmanuelcalvo.com.
Server:		127.0.0.1
Address:	127.0.0.1#53

Name:	ext.jmanuelcalvo.com
Address: 192.168.1.2
```


## NOTAS: 
En caso de querer ser un poco mas restrictivo con la creación de registros, en el archivo named.conf en la configuración de la zona se pueden realizar configuraciones de este tipo:



```
zone "jmanuelcalvo.com" {
    type master;
    file "db.jmanuelcalvo.com.zone";
    allow-update {
        key mrslave.jmanuelcalvo.com.;
    };
};
```

o si solo desea permitir que la clave mrslave.jmanuelcalvo.com actualice el registro A de jmanuelcalvo.com, el archivo se vería así:

```
zone "jmanuelcalvo.com" {
    type master;
    file "db.jmanuelcalvo.com.zon";
    update-policy {
        grant mrslave.jmanuelcalvo.com. name jmanuelcalvo.com. A;
    };
};
```

Material de ayuda:
https://dnns.no/dynamic-dns-with-bind-and-nsupdate.html

## Bonus 

Este seria el playook que actualiza el DNS, es necesario tener instalado el paquete python-dns:
```
[root@server02 ~]# yum install python-dns
```

Esto es gracias a un mensaje de error como este:
```
fatal: [172.16.132.245]: FAILED! => {"changed": false, "msg": "python library dnspython required: pip install dnspython"}
```

Una vez instalado el paquete, crear un playbook con el siguiente contenido
```
[root@server02 ~]# vim modify.yml
---
- name: Actualizar registro en el DNS
  hosts: webservers
  gather_facts: no
  tasks:
  - name: Add or modify ext3.jmanuelcalvo A to 192.168.1.3"
    nsupdate:
      key_algorithm: hmac-sha512
      key_name: "mrslave.jmanuelcalvo.com"
      key_secret: "wrm1CZ36Us0bNHn9c4K9tfS2p3bl4kBnvB+wcUynrzxa0vmjopqBzJ0p smOaA3owSLRtzfwNZ6FsDAJNjvIWhw=="
      server: "server02.jmanuelcalvo.com"
      zone: "jmanuelcalvo.com"
      record: "ext3"
      value: "192.168.1.3"
```

y probar que funcione correctamente
```
[root@server02 ~]# ansible-playbook modify.yml

PLAY [Actualizar registro en el DNS] *********************************************************************************************************

TASK [Add or modify ansible.example.org A to 192.168.1.1"] ***********************************************************************************
changed: [172.16.132.245]

PLAY RECAP ***********************************************************************************************************************************
172.16.132.245             : ok=1    changed=1    unreachable=0    failed=0
```


El resultado deberia ser algo asi:
```
[root@server02 ~]# nslookup ext3.jmanuelcalvo.com
Server:		127.0.0.1
Address:	127.0.0.1#53

Name:	ext3.jmanuelcalvo.com
Address: 192.168.1.3
```
