Para los que veniamos acostumbrados a utilizar el comando ´screen´ en las diferentes versiones de Red Hat Enterprise Linux y sus primos hermanos, luego de la actualizacion a RHEL 8 nos encontramos con que este lindo comando ya no se encuentra instalado ni tampoco habilitado en los repos, por lo que nos sugieren utilizar el legendario comando `tmux`. sin embargo no se si a ustedes les paso pero yo ya estaba acostumbrado a poder salirme del screen, listar las sessiones, attachar y desatachar una session y algo tambien super importante, poder hacer scroll para ver los errores en una pantalla.

Pues bueno hoy por fin encontre como configurar el scroll y pense en documentarlo para que no se me olvide, y es realmente sencillo

En RHEL8, puede crear un archivo llamado para la configuracion de todas las cuentas `/etc/tmux.conf` o puede crear su propio archivo de configuracion para su perfil local `~/.tmux.conf`

```
[root@rhvh1 ~]# vi /etc/tmux.conf

set -g mouse on
set -g history-limit 50000
```

y de ahora en adelante, ya vas a poder hacer scroll con el mouse y con la segunda linea puede setear el numero de lineas que podra recordar en la sesion de tmux

Otros comandos/convinaciones de letras utiles

```
ctrl b + d = Esta convinacion permite salir de la session
```

Para listar las sessiones activas

```
[root@rhvh1 ~]# tmux ls
0: 1 windows (created Tue Nov 16 09:18:02 2021) [140x37]
1: 1 windows (created Tue Nov 16 09:23:26 2021) [140x37]
```

Para conectarse a una session especifica

```
[root@rhvh1 ~]# tmux a -t 1
```

Es posible crear una sesion con un nombre especifico

```
[root@rhvh1 ~]# tmux new -s instalacion

[root@rhvh1 ~]# tmux ls
0: 1 windows (created Tue Nov 16 09:18:02 2021) [140x37]
1: 1 windows (created Tue Nov 16 09:23:26 2021) [140x37]
instalacion: 1 windows (created Tue Nov 16 09:27:03 2021) [140x37]

[root@rhvh1 ~]# tmux a -t instalacion
```

Una opcion interesante, es ingresando dentro de un Tmux es posible cambiarse de forma mas "grafica" entre sessiones con la convinacion

```
ctrl b + s
```

Para matar o eliminar uan session puedo ingresar a la session y darle el comando exit o tambien con la convinacion

```
[root@rhvh1 ~]# tmux kill-session -a -t 0
```


