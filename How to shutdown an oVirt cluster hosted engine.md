# Apagar un Cluster RHV hosted engine

1) Detenga todas las máquinas virtuales.

2) Mueva todos los dominios de almacenamiento que no sean hosted_storage a mantenimiento lo que los desmontará de todos los nodos.

3) Pasar el Hosted Engine a mantenimiento global

`hosted-engine --set-maintenance --mode=global`

4) Detener Hosted Engine vm ejecutando el comando

`hosted-engine --vm-shutdown`

5) Confirmar que engine está apagado utilizando el comando

`hosted-engine --vm-status`

6) Detener los servicios del agente ha y broker en todos los nodos ejecutando el
comando:

`systemctl stop ovirt-ha-broker ; systemctl stop ovirt-ha-agent`

7) Desmontar el hosted-engine de todos los hipervisores

`hosted-engine --disconnect-storage`

8) Detener todos los volúmenes.

9) Apagar todos los hipervisores.


# Iniciar un Cluster RHV hosted engine

TPara que vuelva a subir, los siguientes pasos le ayudarán.


1) Encienda todos los hipervisores.

2) Iniciar todos los volúmenes

3) Iniciar los servicios del agente ha y broker en todos los nodos ejecutando el
comando:

`systemctl start ovirt-ha-broker ; systemctl start ovirt-ha-agent`

4) Mueva el hosted-engine de mantenimiento global ejecutando el comando

`hosted-engine --set-maintenance --mode =none`

5) Dele un poco de tiempo para que la vm Hosted Engine suba. compruebe si la vm Hosted Engine vm está arriba.

`hosted-engine --vm-status`


6) Activar todos los dominios de almacenamiento desde la UI.

7) Iniciar todas las máquinas virtuales.


https://lists.ovirt.org/pipermail/users/2017-August/083665.html
