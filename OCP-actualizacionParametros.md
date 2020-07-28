### Actualización de parámetros de configuración de los masters

Cuando se requiera actualizar un parámetro de los nodos masters, se debería poder realizar con este procedimiento:

Configurar las variables que se deseen utilizar en el archivo de inventario de ansible:

ejemplo:
```
openshift_master_image_policy_config={"MaxImagesBulkImportedPerRepository": "3", "DisableScheduledImport": "false", "MaxScheduledImageImportsPerMinute": "10", "ScheduledImageImportMinimumIntervalSeconds": "1800"}
``` 

Luego ejecutar el playbook que simplemente actualiza la configuración en los masters:

```
# ansible-playbook -i /etc/ansible/hosts /usr/share/ansible/openshift-ansible/playbooks/byo/openshift-master/config.yml
```

Al finalizar se podría verificar el resultado sobre los nodos masters verificando el archivo /etc/origin/master/master-config.yaml:
```
imagePolicyConfig:
  MaxScheduledImageImportsPerMinute: 10
  ScheduledImageImportMinimumIntervalSeconds: 1800
  disableScheduledImport: false
  maxImagesBulkImportedPerRepository: 3
  
```
Para que los cambios se tomen, se debe reiniciar los servicios
```
# master-restart api
# master-restart controllers
```
