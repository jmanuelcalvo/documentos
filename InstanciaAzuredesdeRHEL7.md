# Pasos para la creacion de una instancia en Azure desde una maquina RHEL7 / CentOS

## Habilitar los repositorios necesarios para instalar los modulos de PIP 
```
[root@localhost ~]# subscription-manager repos --enable rhel-server-rhscl-7-rpms
``` 

## Instalar python pip
```
[root@localhost ~]# yum install python27-python-pip
[root@localhost ~]# scl enable python27 bash
```
## Instalar los modulos de Azure
```
[root@localhost ~]# pip install ansible[azure]
Collecting ansible[azure]
...
...
```

Descargar modulos adicionales para Azure
```
[root@localhost ~]# vim requirements-azure.txt
packaging
requests[security]
azure-cli-core==2.0.35
azure-cli-nspkg==3.0.2
azure-common==1.1.11
azure-mgmt-authorization==0.51.1
azure-mgmt-batch==5.0.1
azure-mgmt-cdn==3.0.0
azure-mgmt-compute==4.4.0
azure-mgmt-containerinstance==1.4.0
azure-mgmt-containerregistry==2.0.0
azure-mgmt-containerservice==4.4.0
azure-mgmt-dns==2.1.0
azure-mgmt-keyvault==1.1.0
azure-mgmt-marketplaceordering==0.1.0
azure-mgmt-monitor==0.5.2
azure-mgmt-network==2.3.0
azure-mgmt-nspkg==2.0.0
azure-mgmt-redis==5.0.0
azure-mgmt-resource==2.1.0
azure-mgmt-rdbms==1.4.1
azure-mgmt-servicebus==0.5.3
azure-mgmt-sql==0.10.0
azure-mgmt-storage==3.1.0
azure-mgmt-trafficmanager==0.50.0
azure-mgmt-web==0.41.0
azure-nspkg==2.0.0
azure-storage==0.35.1
msrest==0.6.1
msrestazure==0.5.0
azure-keyvault==1.0.0a1
azure-graphrbac==0.40.0
azure-mgmt-cosmosdb==0.5.2
azure-mgmt-hdinsight==0.1.0
azure-mgmt-devtestlabs==3.0.0
azure-mgmt-loganalytics==0.2.0
azure-mgmt-iothub==0.7.0
```
requirements-azure.txt


## Loguearse en Azure se tienen 3 Opciones:

### 1. Setear las sigiuentes variables de ambiente:
```
AZURE_CLIENT_ID=<service_principal_client_id>
AZURE_SECRET=<service_principal_password>
AZURE_SUBSCRIPTION_ID=<azure_subscription_id>
AZURE_TENANT=<azure_tenant_id>
```

### 2. Adicionar el siguiente contenidos en el archivo $HOME/.azure/credentials:
```
[default]
subscription_id=xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
client_id=xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
secret=xxxxxxxxxxxxxxxxx
tenant=xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```


### 3. ejecutar el comando az login y este re-dirigira a la un navegador web para realizar el logueo
```
az login
```
## Para instalar el comando cliente Azure Client az se pueden ejecutar estos comandos:
```
[root@localhost ~]# sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
[root@localhost ~]# sudo sh -c 'echo -e "[azure-cli]\nname=Azure CLI\nbaseurl=https://packages.microsoft.com/yumrepos/azure-cli\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/azure-cli.repo'
[root@localhost ~]# sudo yum install azure-cli
```
https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-yum?view=azure-cli-latest

## Crear la llave para inyectar a la instancia

```
[root@localhost ~]# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:rQbr0Hzp16Gh1TduxUv44TWfv838e9rF6i6FthQb8Sk root@localhost.localdomain
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|            .    |
|             o . |
|         .  E o  |
|      . S .. *.. |
|     o o oo B.+=+|
|    . + =o * *+oX|
|     o +. o + oX=|
|      . ..   ==+&|
+----[SHA256]-----+ 
```

## Ejecutar una instancia en Azure
```
[root@localhost ~]# vim instancia.yml
- name: Create Azure VM
  hosts: localhost
  connection: local
  tasks:
  - name: Create resource group
    azure_rm_resourcegroup:
      name: myResourceGroup
      location: eastus

  - name: Create virtual network
    azure_rm_virtualnetwork:
      resource_group: myResourceGroup
      name: myVnet
      address_prefixes: "10.0.0.0/16"

  - name: Add subnet
    azure_rm_subnet:
      resource_group: myResourceGroup
      name: mySubnet
      address_prefix: "10.0.1.0/24"
      virtual_network: myVnet

  - name: Create public IP address
    azure_rm_publicipaddress:
      resource_group: myResourceGroup
      allocation_method: Static
      name: myPublicIP
    register: output_ip_address

  - name: Dump public IP for VM which will be created
    debug:
      msg: "The public IP is {{ output_ip_address.state.ip_address }}."

  - name: Create Network Security Group that allows SSH
    azure_rm_securitygroup:
      resource_group: myResourceGroup
      name: myNetworkSecurityGroup
      rules:
        - name: SSH
          protocol: Tcp
          destination_port_range: 22
          access: Allow
          priority: 1001
          direction: Inbound

  - name: Create virtual network interface card
    azure_rm_networkinterface:
      resource_group: myResourceGroup
      name: myNIC
      virtual_network: myVnet
      subnet: mySubnet
      public_ip_name: myPublicIP
      security_group: myNetworkSecurityGroup

  - name: Create VM
    azure_rm_virtualmachine:
      resource_group: myResourceGroup
      name: myVM
      vm_size: Standard_DS1_v2
      admin_username: azureuser
      ssh_password_enabled: false
      #ssh_public_keys:
      #  - path: /home/azureuser/.ssh/authorized_keys
      #    key_data: /root/.ssh/id_rsa.pub
      network_interfaces: myNIC
      image:
        offer: CentOS
        publisher: OpenLogic
        sku: '7.5'
        version: latest

[root@localhost ansible-playbooks]# ansible-playbook instancia.yml
```
