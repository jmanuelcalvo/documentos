# Ejecución de un playbook en Windows

Tal como en Linux, existen una serie de módulos de ansible para Windows y funcionan de forma similar a lo que quizás ya has aprendido en el pasado

Este es el listado de módulos que se muestra en la documentación

```
[root@rhel7 ansible-win]# ansible-doc  -l | grep win_

win_acl                                              Set file/directory/registry permissions for a system user or group
win_acl_inheritance                                  Change ACL inheritance
win_audit_policy_system                              Used to make changes to the system wide Audit Policy
win_audit_rule                                       Adds an audit rule to files, folders, or registry keys
win_certificate_store                                Manages the certificate store
win_chocolatey                                       Manage packages using chocolatey
win_chocolatey_config                                Manages Chocolatey config settings
win_chocolatey_feature                               Manages Chocolatey features
win_chocolatey_source                                Manages Chocolatey sources
win_command                                          Executes a command on a remote Windows node
win_copy                                             Copies files to remote locations on windows hosts
win_defrag                                           Consolidate fragmented files on local volumes
win_disk_facts                                       Show the attached disks and disk information of the target host
win_disk_image                                       Manage ISO/VHD/VHDX mounts on Windows hosts
win_dns_client                                       Configures DNS lookup on Windows hosts
win_domain                                           Ensures the existence of a Windows domain
win_domain_computer                                  Manage computers in Active Directory
win_domain_controller                                Manage domain controller/member server state for a Windows host
win_domain_group                                     Creates, modifies or removes domain groups
win_domain_membership                                Manage domain/workgroup membership for a Windows host
win_domain_user                                      Manages Windows Active Directory user accounts
win_dotnet_ngen                                      Runs ngen to recompile DLLs after .NET  updates
win_dsc                                              Invokes a PowerShell DSC configuration
win_environment                                      Modify environment variables on windows hosts
win_eventlog                                         Manage Windows event logs
win_eventlog_entry                                   Write entries to Windows event logs
win_feature                                          Installs and uninstalls Windows Features on Windows Server
win_file                                             Creates, touches or removes files or directories
win_file_version                                     Get DLL or EXE file build version
win_find                                             Return a list of files based on specific criteria
win_firewall                                         Enable or disable the Windows Firewall
win_firewall_rule                                    Windows firewall automation
win_get_url                                          Downloads file from HTTP, HTTPS, or FTP to node
win_group                                            Add and remove local groups
win_group_membership                                 Manage Windows local group membership
win_hostname                                         Manages local Windows computer name.
win_hotfix                                           Install and uninstalls Windows hotfixes
win_iis_virtualdirectory                             Configures a virtual directory in IIS
win_iis_webapplication                               Configures IIS web applications
win_iis_webapppool                                   Configure IIS Web Application Pools
win_iis_webbinding                                   Configures a IIS Web site binding
win_iis_website                                      Configures a IIS Web site
win_lineinfile                                       Ensure a particular line is in a file, or replace an existing line using a back-referenced regular expression
win_mapped_drive                                     Map network drives for users
win_msg                                              Sends a message to logged in users on Windows hosts
win_msi                                              Installs and uninstalls Windows MSI files
win_nssm                                             NSSM - the Non-Sucking Service Manager
win_owner                                            Set owner
win_package                                          Installs/uninstalls an installable package
win_pagefile                                         Query or change pagefile configuration
win_path                                             Manage Windows path environment variables
win_pester                                           Run Pester tests on Windows hosts
win_ping                                             A windows version of the classic ping module
win_power_plan                                       Changes the power plan of a Windows system
win_product_facts                                    Provides Windows product information (product id, product key)
win_psexec                                           Runs commands (remotely) as another (privileged) user
win_psmodule                                         Adds or removes a Powershell Module
win_rabbitmq_plugin                                  Manage RabbitMQ plugins
win_reboot                                           Reboot a windows machine
win_reg_stat                                         Get information about Windows registry keys
win_regedit                                          Add, change, or remove registry keys and values
win_region                                           Set the region and format settings
win_regmerge                                         Merges the contents of a registry file into the windows registry
win_robocopy                                         Synchronizes the contents of two directories using Robocopy
win_route                                            Add or remove a static route
win_say                                              Text to speech module for Windows to speak messages and optionally play sounds
win_scheduled_task                                   Manage scheduled tasks
win_scheduled_task_stat                              Get information about Windows Scheduled Tasks
win_security_policy                                  Change local security policy settings
win_service                                          Manage and query Windows services
win_share                                            Manage Windows shares
win_shell                                            Execute shell commands on target hosts
win_shortcut                                         Manage shortcuts on Windows
win_stat                                             Get information about Windows files
win_tempfile                                         Creates temporary files and directories
win_template                                         Templates a file out to a remote server
win_timezone                                         Sets Windows machine timezone
win_toast                                            Sends Toast windows notification to logged in users on Windows 10 or later hosts
win_unzip                                            Unzips compressed files and archives on the Windows node
win_updates                                          Download and install Windows updates
win_uri                                              Interacts with webservices
win_user                                             Manages local Windows user accounts
win_user_right                                       Manage Windows User Rights
win_wait_for                                         Waits for a condition before continuing
win_wait_for_process                                 Waits for a process to exist or not exist before continuing.
win_wakeonlan                                        Send a magic Wake-on-LAN (WoL) broadcast packet
win_webpicmd                                         Installs packages using Web Platform Installer command-line
win_whoami                                           Get information about the current user and process
win_xml                                              Add XML fragment to an XML parent
```


>**Tip**
>
> Recuerda que para usar cualquiera de estos módulos, puedes obtener un ejemplo ejecutando el comando **ansible-doc <modulo>, ejemplo ansible-doc win_copy** y al final de esta documentación se encontrara un ejemplo practico para tu playbook

Ahora para empezar a realizar nuestras pruebas de ejecución de comandos sobre Windows de forma rápida y básica se deben realizar los siguientes pasos:

1. La forma como Ansible ejecuta comandos en Windows es a través de WinRM (Windows Remote Management) en cual ya se encuentra en las ultimas versiones de Windows, en algunos casos unicamente se debe habilitar, para ello se debe ejecutar en el interprete de comandos de windows (CMD) el comando **winrm qc**
Esta tarea se debe realizar en el CMD de Windows como usuario administrador así: Inicio -> buscar -> CMD -> Clic derecho (ejecutar como administrador)

![Ref](images/winrm1.png)

Una vez en el CMD se ejecuta el comando **winrm qc** este comando no solo se encarga de habilitar el winrm, si no que también crea la entrada en el firewall (si la red esta configurada como *Publica* el comando indicara que no puede habilitar el firewall (por que no puede permitir lo que ya esta permitido)

![Ref](images/winrm2.png)


Una vez habilitado podemos comprobar la información así:

```
C:\Windows\system32>winrm enumerate winrm/config/Listener
Listener
    Address = *
    Transport = HTTP
    Port = 5985
    Hostname
    Enabled = true
    URLPrefix = wsman
    CertificateThumbprint
    ListeningOn = 127.0.0.1, 169.254.119.190, 192.168.0.15, ::1, 2800:484:6e80:eb80:50c2:7fc8:4e3e:297e, 2800:484:6e80:eb80:cdd4:fb48:d547:c489, fe80::2080:9244:2762:77be%7, fe80::50c2:7fc8:4e3e:297e%5

```

2. Ya se encuentra habilitado el WinRM, la siguiente es la ejecución de un script que comprueba la configuración actual de WinRM (PS Remoting) y hace los cambios necesarios para permitir que Ansible se conecte, autentique y ejecutar comandos de PowerShell.

> **Advertencia**
>
> En caso de no ejecutar este powershell los playbooks pueden mostrar un mensaje como este:
> 
```
[root@rhel7 ansible-win]# ansible win1 -i hosts -m win_ping
192.168.0.15 | UNREACHABLE! => {
    "changed": false,
    "msg": "ssl: HTTPSConnectionPool(host='192.168.0.15', port=5986): Max retries exceeded with url: /wsman (Caused by ConnectTimeoutError(<urllib3.connection.HTTPSConnection object at 0x7f462a9c82d0>, 'Connection to 192.168.0.15 timed out. (connect timeout=30)'))",
    "unreachable": true
}
```

El script de powershell se puede ubicar en esta ruta:
https://github.com/ansible/ansible/blob/devel/examples/scripts/ConfigureRemotingForAnsible.ps1

o buscando en google **configure remoting for ansible.ps1** 

una vez guardado el archivo, se debe proceder a la ejecución desde PowerShell (**ejecutado como administrador**)


![Ref](images/winrm3.png)


En la siguiente imagen podra observar que al momento de ejecutar el script de PowerShell se mostrara un error con un mensaje **about_Execution_Policies**, para este ejemplo se va a habilitar la ejecución de políticas con el comando:

```
PS C:\Users\jmanuel\Documents> set-executionpolicy unrestricted
```

Finalmente desde la ruta donde se encuentra el Script de PowerShell, ejecutarlo

```
PS C:\Users\jmanuel\Documents> .\ConfigureRemotingForAnsible.ps1
```
![Ref](images/winrm4.png)


3. La maquina con Windows se encuentra lista para la ejecución de comandos desde Linux, ahora desde la maquina con Ansible también se debe realizar un alistamiento, en principio se debe garantizar que la maquina tenga instalado tanto el paquete de Ansible y un paquete llamado python-pip el cual proporcionara el comando **pip** que permita la adición de módulos de python

```
[root@rhel7 ansible-win]# yum install python-pip
Loaded plugins: product-id, search-disabled-repos, subscription-manager
Resolving Dependencies
--> Running transaction check
---> Package python2-pip.noarch 0:8.1.2-12.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================================================================================================================
 Package                                      Arch                                    Version                                       Repository                             Size
================================================================================================================================================================================
Installing:
 python2-pip                                  noarch                                  8.1.2-12.el7                                  epel                                  1.7 M

Transaction Summary
================================================================================================================================================================================
Install  1 Package

Total download size: 1.7 M
Installed size: 7.2 M
Is this ok [y/d/N]: y

```


Una vez con el comando pip, se instala el **pywinrm**

```
[root@rhel7 ansible-win]# pip install pywinrm
Collecting pywinrm
  Downloading https://files.pythonhosted.org/packages/fc/88/be0ea1af44c3bcc54e4c41e4056986743551693c77dfe50b48a3f4ba1bf7/pywinrm-0.4.1.tar.gz
Collecting xmltodict (from pywinrm)
...
...
Downloading https://files.pythonhosted.org/packages/1a/70/1935c770cb3be6e3a8b78ced23d7e0f3b187f5cbfab4749523ed65d7c9b1/requests-2.23.0-py2.py3-none-any.whl (58kB)
    100% |████████████████████████████████| 61kB 598kB/s
      Successfully uninstalled idna-2.4
  Running setup.py install for pywinrm ... done
Successfully installed certifi-2020.4.5.1 chardet-3.0.4 idna-2.9 ntlm-auth-1.4.0 pywinrm-0.4.1 requests-2.23.0 requests-ntlm-1.1.0 urllib3-1.25.9 xmltodict-0.12.0
You are using pip version 8.1.2, however version 20.1 is available.
```

4. Están listos los pre-requisitos para este ejercicio, ahora se debe crear un archivo de inventario con los datos de la maquina/s windows y este debe ser similar a este:
```
[root@rhel7 ~]# mkdir ansible-win/
[root@rhel7 ~]# cd ansible-win/
[root@rhel7 ansible-win]# vim hosts

[win]
192.168.0.15

[win:vars]
ansible_user=jmanuel
ansible_password=XXXXXXX
ansible_connection=winrm
ansible_winrm_server_cert_validation=ignore
```

y tal como se hace con maquinas con sistema operativo Linux, el modulo **win_ping** permite validar la conectividad de Windows así:

```
[root@rhel7 ansible-win]# ansible win -i hosts -m win_ping
192.168.0.15 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

5. Si la salida de ping es correcto, todo esta listo para empezar a ejecutar playbooks, aqui unos ejemplos de tareas de validación de parámetros de las maquinas, (información obtenida de https://geekflare.com/ansible-playbook-windows-example/)

```
[root@rhel7 ansible-win]# cat check.yml
---
- hosts: win
  tasks:
  - name: Get disk facts
    win_disk_facts:

  - name: Output first disk size
    debug:
      var: ansible_facts.disks[0].size

  - name: Convert first system disk into various formats
    debug:
      msg: '{{ disksize_gib }} vs {{ disksize_gib_human }}'
    vars:
      # Get first system disk
      disk: '{{ ansible_facts.disks|selectattr("system_disk")|first }}'

      # Show disk size in Gibibytes
      disksize_gib_human: '{{ disk.size|filesizeformat(true) }}'
      disksize_gib: '{{ (disk.size/1024|pow(3))|round|int }} GiB'


  - name: Get disk facts
    win_command: wmic cpu get caption, deviceid, name, numberofcores, maxclockspeed, status
    register: usage
  - debug: msg={{ usage.stdout }}

  - name: run an executable using win_command
    win_command: whoami.exe

  - name: run a cmd command
    win_command: cmd.exe /c mkdir C:\test
```    

Ejecución 

```
[root@rhel7 ansible-win]# ansible-playbook -i hosts check.yml

PLAY [win] ********************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************
ok: [192.168.0.15]

TASK [Get disk facts] **********************************************************************************************************************************************************
ok: [192.168.0.15]

TASK [Output first disk size] **************************************************************************************************************************************************
ok: [192.168.0.15] => {
    "ansible_facts.disks[0].size": "64424509440"
}

TASK [Convert first system disk into various formats] **************************************************************************************************************************
ok: [192.168.0.15] => {
    "msg": "60 GiB vs 60.0 GiB"
}

TASK [Get disk facts] **********************************************************************************************************************************************************
changed: [192.168.0.15]

TASK [debug] *******************************************************************************************************************************************************************
ok: [192.168.0.15] => {
    "msg": "Caption                               DeviceID  MaxClockSpeed  Name                                       NumberOfCores  Status  \r\r\nIntel64 Family 6 Model 94 Stepping 3  CPU0      2712           Intel(R) Core(TM) i7-6820HQ CPU @ 2.70GHz  2              OK      \r\r\n\r\r\n"
}

TASK [run an executable using win_command] *************************************************************************************************************************************
changed: [192.168.0.15]

TASK [run a cmd command] *******************************************************************************************************************************************************
changed: [192.168.0.15]

PLAY RECAP *********************************************************************************************************************************************************************
192.168.0.15               : ok=8    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

6. Playbooks un poco mas avanzados que permiten la instalación de productos

Como este playbook va a escribir un archivo basado en una plantilla de Jinja, lo primero es crear este archivo:

```
[root@rhel7 ansible-win]# vim file.conf.j2
Y este es un archivo Jinja que soporta variables 
Ambiente {{ ambiente }}
```
Como se puede observar, esta plantilla espera una variable que se llama **{{ ambiente }}** la cual se puede pasar desde el playbook, desde un survey de Tower o desde la linea de comandos con la opción **-e ambiente=**


```
[root@rhel7 ansible-win]# vim install_software.yml
---
- name: Instalación de productos en Windows 10
  hosts: win
  gather_facts: no
  vars:
    iis_test_message: Hola
    ambiente: Desarrollo
  tasks:

  - name: Crear la carpeta de configuracion
    win_file:
      path: C:\Temp\
      state: directory

  - name: Copiar los archivos de configuracion
    win_copy:
      content: "{{ iis_test_message }}"
      dest: C:\Temp\index.html

  - name: Crear un archivo desde plantilla jinja
    win_template:
      src: file.conf.j2
      dest: C:\Temp\file.conf


  - name: Mostrar la direccion del servidor
    debug:
      msg: "http://{{ ansible_host }}"

  - name: Download the Apache installer
    win_get_url:
      url: https://archive.apache.org/dist/httpd/binaries/win32/httpd-2.2.25-win32-x86-no_ssl.msi
      dest: C:\Temp\httpd-2.2.25-win32-x86-no_ssl.msi

  - name: Install MSI
    win_package:
      path: C:\Temp\httpd-2.2.25-win32-x86-no_ssl.msi
      state: present

#  - name: UnInstall MSI
#    win_package:
#      path: C:\ansible_examples\httpd-2.2.25-win32-x86-no_ssl.msi
#      state: absent
``` 
   
Ejecucion 

```
[root@rhel7 ansible-win]# ansible-playbook -i hosts  install_software.yml -e iis_test_message=Casa -e ambiente=produccion

PLAY [Instalacion de productos en Windows 10] **********************************************************************************************************************************

TASK [Crear la carpeta de configuracion] ***************************************************************************************************************************************
ok: [192.168.0.15]

TASK [Copiar los archivos de configuracion] ************************************************************************************************************************************
changed: [192.168.0.15]

TASK [Crear un archivo desde plantilla jinja] **********************************************************************************************************************************
changed: [192.168.0.15]

TASK [Mostrar la direccion del servidor] ***************************************************************************************************************************************
ok: [192.168.0.15] => {
    "msg": "http://192.168.0.15"
}

TASK [Download the Apache installer] *******************************************************************************************************************************************
changed: [192.168.0.15]

TASK [Install MSI] *************************************************************************************************************************************************************
changed: [192.168.0.15]

PLAY RECAP *********************************************************************************************************************************************************************
192.168.0.15               : ok=6    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

Y los resultados sobre la maquina Windows son:

![Ref](images/winrm5.png)



### NOTAS
Un comando ejecutado desde PowerShell de Windows que puede ser útil al momento de configuracion de WinRM es:
```
winrm get winrm/config
```

![Ref](images/winrm6.png)

### NOTAS
En caso que tenga una maquina con Red Hat y no con CentOS, la instalacion del comando pip es diferente:

El comando pip se puede encontrar en cualquiera de los paquetes/repos
https://access.redhat.com/solutions/1519803

Una vez instalado, ejecutar:
```
$ scl enable python27 bash
$ which pip
$ pip -V
```

### FUENTES:
* https://geekflare.com/ansible-playbook-windows-example/

* https://github.com/ansible/workshops/tree/devel/exercises/ansible_windows/5-adv-playbook

* https://www.ansible.com/blog/connecting-to-a-windows-host





============


## Preparacion de Ansible Core e Integracion con los modulos de Windows

[root@container ~]# yum install ansible-core


[root@container ~]# cat win_inventory
[win]
192.168.64.4


[win:vars]
ansible_connection=winrm
ansible_user=ansible
ansible_password=ansible
ansible_port= 5985
ansible_winrm_server_cert_validation=ignore
ansible_winrm_transport=basic
ansible_winrm_scheme=http


[root@container ~]#  ansible win -m win_ping -i /root/win_inventory
192.168.64.4 | FAILED! => {
    "msg": "The module win_ping was redirected to ansible.windows.win_ping, which could not be loaded."
}

[root@container ~]# ansible-galaxy collection install community.windows

[root@container ~]#  ansible win -m win_ping -i /root/win_inventory
192.168.64.4 | FAILED! => {
    "msg": "winrm or requests is not installed: No module named 'winrm'"
}

Importante identificar la version de python que esta ejecutando Ansible

[root@container ~]#  ansible win -m win_ping -i /root/win_inventory  -vv
...
  python version = 3.11.2
...

[root@container ~]# yum install python2-pip
[root@container ~]# yum install python3-pip
[root@container ~]# yum install python3.11-pip
[root@container ~]# python3 -m pip install --upgrade pip



[root@container ~]# python3.11 -m pip list
[root@container ~]# python3.11 -m pip install pywinrm

