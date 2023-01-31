# Configuracion de las Notificaciones en Gmail desde Ansible Automation Platform

En estos caso que requieras que aplicaciones externas en este caso como Ansible puedan realizar el envio de correos, puedes utilizar Google App Passwords para ello puedes realizar los siguientes pasos:

Ingresar a myaccount.google.com -> Click en Security en el panel izquierdo -> Buscar App Passwords.

![AAP](images/AAP-gmail01.png)

Seleccione App Passwords o Contraseñas de aplicaciones y llegará a otra ventana. (Google puede pedirle que introduzca la contraseña para verificar). Cree una nueva contraseña de aplicación seleccionando Otros en el menú desplegable.

![AAP](images/AAP-gmail02.png)

Una vez generada, visualizara una pantalla como la siguiente en donde aparece la contraseña separada por espacios, (haga caso omizso a los espacios, cuando da click en los numeros, estos se copian sin espacios)

![AAP](images/AAP-gmail03.png)


# Configuracion Ansible Automation Platform

Una vez ingrese a la interfase web, vaya a Administration -> Notifications en el panel de la izquierda

![AAP](images/AAP-gmail04.png)

Adicione un nuevo notificador, para este caso, en Type seleccione E-mail

![AAP](images/AAP-gmail05.png)

Tenga en cuanta que en el campo de **Password** debe pegar el password generado en la aplicacion de contraseñas de google (SIN ESPACIOS), tambien es importante que en la parte de **E-mail options** marque la casilla *USE TLS*

![AAP](images/AAP-gmail06.png)

Una vez guarde los cambios puede realizar pruebas de envio de correo para validar si la configuracion es valida

![AAP](images/AAP-gmail07.png)

# Configuracion de notificaciones en un ejecucion de un plabook / template especifico

Dentro de las templates que ya tiene configurados, puede habilitar para que este realice la notificacion asi:

![AAP](images/AAP-gmail08.png)


Dentro del template especifico ingrese a la pestaña llamada **Notificatios** y ahi puede habilitar en que circunstancia quiere que se notifique, cuando la tarea:

* Start / Inicia
* Success / Finaliza satisfactoriamente
* Failure / Falla

