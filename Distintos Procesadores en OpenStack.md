Validar la familia de procesadores:

```
[root@overcloud-computehp-0 config-data]# cat /sys/devices/cpu/caps/pmu_name
haswell

[root@overcloud-compute-0 nova]# cat /sys/devices/cpu/caps/pmu_name
ivybridge
```

Para realizar garantizar que los hypervisores en RHOSP emulen la misma familia de procesadores y se pueda realizar la migracion en vivo (Live Migration), se debe:

1. Validar dentro de los contenedores (docker) el parametro de configutracion actual

```
[root@overcloud-compute-0 ~]# docker exec -it nova_compute  bash
()[nova@overcloud-compute-0 /]$ grep mode /etc/nova/nova.conf |egrep -v '^#'
cpu_mode=host-model
```

Este parametro es equivalente al que se encuentra en la carpeta local del linux: /var/lib/config-data/puppet-generated/nova_libvirt/etc/nova/nova.conf
```
[root@overcloud-compute-0 ~]# grep mode /var/lib/config-data/puppet-generated/nova_libvirt/etc/nova/nova.conf  | egrep -v '^#'
```

2. Reemplazar el parametro en los archivos de configuracion:


NOTA:

Las familias de los procesadores Intel son
```
cpu_models = Conroe,Penryn,Nehalem,Westmere,SandyBridge,IvyBridge,Haswell,Broadwell,Skylake-Client,Skylake-Server
```






### NOTAS DE REFERENCIA
https://access.redhat.com/solutions/3301301
https://access.redhat.com/solutions/4162701
