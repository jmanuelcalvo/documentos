# Consultas y creacion de recursos en Vmware desde Mac con govc

`govc` es una CLI de vSphere construida sobre govmomi.

La CLI está diseñada para ser una alternativa amigable a la GUI y muy adecuada para tareas de automatización. También actúa como un sistema de pruebas para las APIs de govmomi y proporciona ejemplos prácticos de cómo utilizar las APIs.

Mas Información
https://github.com/vmware/govmomi/tree/main/govc


1. Instalacion de govc en Mac

```
[jmanuelcalvo@Joses-MacBook ~]$ brew install govc
==> Fetching govc
==> Downloading https://ghcr.io/v2/homebrew/core/govc/manifests/0.30.1
...
```

Exportar las variables de VmWare
```
[jmanuelcalvo@Joses-MacBook ~]$ vi govc.sh
export GOVC_INSECURE=1 # Don't verify SSL certs on vCenter
export GOVC_URL=portal.vc.example.com # vCenter IP/FQDN
export GOVC_USERNAME=sandbox-9l52q@vc.example.com # vCenter username
export GOVC_PASSWORD=P@ssw0rd # vCenter password
export GOVC_DATASTORE=vsanDatastore # Default datastore to deploy to
export GOVC_NETWORK="Cluster01-LAN-1-Routable" # Default network to deploy to
export GOVC_RESOURCE_POOL='cluster01/Resources' # Default resource pool to deploy to
export GOVC_DATACENTER=DC01 # I have multiple DCs in this VC, so i'm specifying the default here
[jmanuelcalvo@Joses-MacBook ~]$ source govc.sh
```


Validar la conectividad
```
[jmanuelcalvo@Joses-MacBook ~]$ govc about
FullName:     VMware vCenter Server 7.0.3 build-19666520
Name:         VMware vCenter Server
Vendor:       VMware, Inc.
Version:      7.0.3
Build:        19666520
OS type:      linux-x64
API type:     VirtualCenter
API version:  7.0.3.2
Product ID:   vpx
UUID:         ee1bef3e-6179-4c1f-9d2a-xxxxxxxxx
```

## Ejemplos de comandos

```
[jmanuelcalvo@Joses-MacBook ~]$ govc datacenter.info
Name:                SDDC-Datacenter
  Path:              /SDDC-Datacenter
  Hosts:             2
  Clusters:          1
  Virtual Machines:  4
  Networks:          1
  Datastores:        1
```

```
[jmanuelcalvo@Joses-MacBook ~]$ govc ls
/SDDC-Datacenter/vm
/SDDC-Datacenter/network
/SDDC-Datacenter/host
/SDDC-Datacenter/datastore
```

```
[jmanuelcalvo@Joses-MacBook ~]$ govc ls /SDDC-Datacenter/network
/SDDC-Datacenter/network/segment-sandbox-9l52q
/SDDC-Datacenter/network/vmc-hostswitch
```

```
[jmanuelcalvo@Joses-MacBook ~]$ govc ls /SDDC-Datacenter/datastore
/SDDC-Datacenter/datastore/WorkloadDatastore
```

```
[jmanuelcalvo@Joses-MacBook ~]$ govc datastore.info /SDDC-Datacenter/datastore/WorkloadDatastore
Name:        WorkloadDatastore
  Path:      /SDDC-Datacenter/datastore/WorkloadDatastore
  Type:      vsan
  URL:       ds:///vmfs/volumes/vsan:3b39c11653124420-83ebad53840b5fa6/
  Capacity:  31851.2 GB
  Free:      24376.1 GB  
```

```
[jmanuelcalvo@Joses-MacBook ~]$ govc ls /SDDC-Datacenter/vm/Workloads/sandbox-9l52q
/SDDC-Datacenter/vm/Workloads/sandbox-9l52q/diego-ansible01
/SDDC-Datacenter/vm/Workloads/sandbox-9l52q/jmanuelcalvo-ansible02
/SDDC-Datacenter/vm/Workloads/sandbox-9l52q/bastion
/SDDC-Datacenter/vm/Workloads/sandbox-9l52q/vmlinux
```

```
[jmanuelcalvo@Joses-MacBook ~]$ govc vm.info /SDDC-Datacenter/vm/Workloads/sandbox-9l52q/jmanuelcalvo-ansible02
```





