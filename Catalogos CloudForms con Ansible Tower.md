# Integracion de playbooks en Ansible tower con Red Hat CloudFoms

Una vez integrados el proveedor Ansible Tower en CloudForms se pueden crear catalogos (un portal con un market place) que se encarguen de hacer el llamado a playbooks y ademas pasarle parametros desde formularios de Cloudforms.

Para esto, es necesario primero realizar la configuracion del Ansible Tower con por lo menos Un proyecto/playbook/Plantilla

## Pasos de configuracion de Ansible Tower

### 1. Crear proyecto Ansible Tower
Resources -> Projects -> +

![Ref](images/tower7.png)

En caso de no tener repositorio git para el Ansible Tower, crear una carpeta local para almacenar los playbooks
```
[root@tower36 ~]# mkdir /var/lib/awx/projects/cloudforms
[root@tower36 ~]# chown awx:awx /var/lib/awx/projects/cloudforms/
```

Una vez creada la carpeta, ya se debe visualizar a traves de la interfase de tower, ahi se debe seleccionar el SCM TYPE: Manual    La subcarpeta local del hosts de ansible tower y la carpeta creada en el paso anterior
![Ref](images/tower8.png)


### 2. Crear un playbook en Ansible, que soporte la inyeccion de variables
Las variables que se ponen en este playbook basico, posteriormente seran enviadas a traves de un catalogo de CloudForms

```
[root@tower36 ~]# cat <<EOF > /var/lib/awx/projects/cloudforms/archivo.yaml
---
- hosts: all
  become: true
  tasks:
  - name: Crear un archivo de pruebas
    file:
      path: /tmp/jmanuel.conf
      state: touch

  - name: Adicionar linea desde variable
    lineinfile:
      path: /tmp/jmanuel.conf
      line:  "{{ line1 }}"
      create: yes
EOF
```

### 3. Crear un inventario de hosts de pruebas, en el Ansible Tower
Resources -> Inventories -> + -> Inventory

![Ref](images/tower1.png)

Rellenar los datos con la descripcion del Hosts y click en SAVE

![Ref](images/tower2.png)

Ir a la pestaña HOSTS y dar click en + 

![Ref](images/tower3.png)

Adicionar la IP o nombre de la VM

![Ref](images/tower4.png)

### 4. Almacenar las contraseñas de las VMs o crear credenciales

Resources -> Credentials -> + -> Inventory

![Ref](images/tower5.png)

En el tipo de credential, seleccionar Machine cuando se trata de una maquina con Linux, en cuanto al usuario y contraseña, se puede seleccionar conexion directa con el usuario root, o en las mejores practicas un usuario que tenga privilegios de SUDO

![Ref](images/tower6.png)

### 5. Crear una plantilla que se encargara de ejecutar el playbook

Dentro de Resources -> Templates -> + -> Job Template

![Ref](images/tower9.png)

Ahora se deben rellenar los campos realizados en los pasos anteriores, inventarios, proyecto, playbook, credenciales y click en SAVE

![Ref](images/tower10.png)

Como la idea es que el playbook reciba parametros, en este caso se debe adicionar sobre la pestaña SURVEY los siguiente

![Ref](images/tower11.png)

Los valores del playbook y del survey deben coincidir

![Ref](images/tower12.png)


Una vez adicionado el campo, click en Save sobre el SURVEY y click en Save sobre el TEMPLATE

### 6. Validar que la plantilla funcione correctamente

Dentro de la Resources -> Templates -> Playbook con contenido por variable -> click en el cohete

![Ref](images/tower13.png)

Rellenar los campos de los formularios NEXT -> LAUNCH

![Ref](images/tower14.png)

y Finalmente el playbook debe ejecutarse de forma satisfactoria

![Ref](images/tower15.png)


### 7. Validar la creacion del archivo y su contenido
```
[root@tower36 ~]# ssh 192.168.0.14
root@192.168.0.14's password:
cat[root@localhost ~]# cat /tmp/jmanuel.conf
datos personalizados
```




## Pasos de configuracion de CloudForms

Desde la interfase web de administrador de Cloudforms, lo primero que se debe es generar el formulario que recibira el servicion, para ello se debe ir a Automation -> Automate -> Customization

![Ref](images/cf1.png)

En el menu Service Dialogs -> All Dialogs

![Ref](images/cf2.png)

Configuration -> Add a new Dialog

![Ref](images/cf3.png)

Se deben llegar los datos del formulario, tenga en cuenta que en paso 2, a mano derecha esta los posibles campos que se le pueden adicionar al formulario, botones, areas de texto, tags, entre otros, esos iconos se pueden arrastrar para realizar el diseño del formulario, una vez arrastrados, en el punto 3 se puede editar dicha casilla

![Ref](images/cf4.png)

Y es justo aqui donde viene la magia de la integracion, ya que si en el playbook la variable que se le puede insertar se llama line1, entonces el campo del formulario del playbook este se debe llamar param_line1 asi:

Referencia: https://access.redhat.com/solutions/3887651

![Ref](images/cf5.png)

Lo siguiente ahora es crear el catalogo, para ello hay que ir al menu Services -> Catalogs 

![Ref](images/cf6.png)

Ir al sub menu Catalog Items -> All Catalog Items

![Ref](images/cf7.png)

y en Configuration -> Add a New Catalog Item

![Ref](images/cf8.png)

Seleccionar Catalog Item Type -> Ansible Tower

![Ref](images/cf9.png)

y llenar los datos que CF obtiene del proveedor de automatizacion Ansible Tower ( si esta bien configurado el proveedor se pueden visualizar el template previamente creado

![Ref](images/cf10.png)

Es hora de realizar la prueba, en el menu -> Service -> Catalogs -> Service Catalogs -> Crear archivo con parametros -> Order

![Ref](images/cf11.png)

y rellenar los datos para el archivo

![Ref](images/cf12.png)

Para validar que el requerimiento se este ejecutando, se puede ingresar a Services -> Requests  y validar en que estado esta el ultimo requerimiento

![Ref](images/cf13.png)

Si el requerimiento se encuentra en estado finish, los sigueinte es entrar a la maquina fisica a realizar la respectiva validacion

![Ref](images/cf14.png)


```
[root@tower36 ~]# ssh 192.168.0.14
root@192.168.0.14's password:
Last login: Thu Mar 12 20:31:18 2020 from 192.168.0.16
[root@localhost ~]# cat /tmp/jmanuel.conf
datos personalizados
Archivo creado desde Cloudforms
[root@localhost ~]#
```







