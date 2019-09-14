### Actualizacion de parametros de configuracion de los masters

Cuando se requiera actualizar un parametro de los nodos masters, se deberia poder realizar con este procediminto:

Configurar las variables que se deseen utilizar en el archivo de inventario de ansible:

ejemplo:
```
openshift_master_image_policy_config={"MaxImagesBulkImportedPerRepository": "3", "DisableScheduledImport": "false", "MaxScheduledImageImportsPerMinute": "10", "ScheduledImageImportMinimumIntervalSeconds": "1800"}
``` 

Luego ejecutar el playbook que simplemente actualiza la configuracion en los masters:

```
# ansible-playbook -i /etc/ansible/hosts /usr/share/ansible/openshift-ansible/playbooks/byo/openshift-master/config.yml
```

Al finalizar se podria verificar el resultado sobre los nodos masters verificando el archivo /etc/origin/master/master-config.yaml:
```
imagePolicyConfig:
  MaxScheduledImageImportsPerMinute: 10
  ScheduledImageImportMinimumIntervalSeconds: 1800
  disableScheduledImport: false
  maxImagesBulkImportedPerRepository: 3
  
```
Para que los cambios se tomen, se debe reinicar los servicios
```
# master-restart api
# master-restart controllers
```
