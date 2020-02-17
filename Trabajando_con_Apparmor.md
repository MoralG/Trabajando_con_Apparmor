# Trabajando con Apparmor

AppArmor es similar a SELinux. Aunque funcionan de forma diferente, tanto AppArmor como SELinux proporcionan seguridad de control de acceso obligatorio (MAC). En efecto, AppArmor permite a los administradores restringir las acciones que pueden realizar los procesos.

AppArmor aplica un conjunto de reglas, guardados en un perfil. El perfil aplicado por el núcleo depende de la ruta de instalación del programa a ejecutar. Al contrario que SELinux, los perfiles que se aplican no dependen del usuario, sino que se aplican a todos los usuarios que ejecuten dicho programa.

Los perfiles de Apparmor se guardan en `/etc/apparmor.d` y estos se pueden cargar dos modos:

* **Modo estricto** (enforcing): Aplica las reglas y registra las violaciones, es decir, reconoce la violación de la regla, las registras y aplica la restrinciones.
* **Modo relajado** (complaining): Solo registra las violaciones pero no aplica la restrinción. 

Los sistemas Debian y Ubuntu tienen Apparmor por defecto instalado, pero si no lo tenemos instalado o lo tenemos desactivado, tenemos que:

**Instalamos Apparmor**
~~~
sudo apt install apparmor
~~~

**Establecemos los parametros en la línea de órdenes del núcleo**
~~~
sudo perl -pi -e 's,GRUB_CMDLINE_LINUX="(.*)"$,GRUB_CMDLINE_LINUX="$1 apparmor=1 security=apparmor",' /etc/default/grub
~~~

**Actualizamos el grup**
~~~
sudo update-grub
~~~

**Reiniciamos Apparmor**
~~~
sudo systemctl restart apparmor
~~~

Por defecto tenemos perfiles activos. Podemos comprobarlo viendo el estado de Apparmor.
~~~
sudo apparmor_status
    apparmor module is loaded.
    15 profiles are loaded.
    15 profiles are in enforce mode.
       /sbin/dhclient
       /usr/bin/lxc-start
       /usr/bin/man
       /usr/lib/NetworkManager/nm-dhcp-client.action
       /usr/lib/NetworkManager/nm-dhcp-helper
       /usr/lib/connman/scripts/dhclient-script
       /usr/lib/snapd/snap-confine
       /usr/lib/snapd/snap-confine//mount-namespace-capture-helper
       /usr/sbin/tcpdump
       lxc-container-default
       lxc-container-default-cgns
       lxc-container-default-with-mounting
       lxc-container-default-with-nesting
       man_filter
       man_groff
    0 profiles are in complain mode.
    0 processes have profiles defined.
    0 processes are in enforce mode.
    0 processes are in complain mode.
    0 processes are unconfined but have a profile defined.
~~~

Como podemos ver, en Tortilla tenemos 15 perfiles cargados.

> **NOTA**: Si queremos perfiles desarrollados por la comunidad o perfiles adicionales desarrolados por Ubuntu y Debian tenemos que descargar los paquetes `apparmor-profiles` y `apparmor-profiles-extra`.

Para saber los procesos que deberiamos de asignarle un perfil, porque no tienen uno por defecto. Podemos ejecutar un comando para saber que procesos no estan confinados.

Tendremos que instalar el paquete `apparmor-utils`.
~~~
sudo apt install apparmor-utils
~~~

Los programas que deben ser confinados son los expuesto a la red, puesto que seŕan los que más problemas tendrán a ataques remotos. Apparmor tiene el comando `aa-unconfined` que lista los programas que tienen al menos un zocalo de red y no tienen ningún perfil asociado.
~~~
sudo aa-unconfined
    916 /usr/sbin/sshd not confined
    7746 /usr/sbin/bacula-fd not confined
    22481 /lib/systemd/systemd-networkd not confined
    22512 /lib/systemd/systemd-resolved not confined
    25261 /usr/sbin/mysqld not confined
    26963 /usr/sbin/netdata not confined
~~~

Como podemos ver, tenemos **sshd**, **bacula-fd**, **mysqld** y **netdata** sin ningún perfil asignado y estan conectados en remoto a otro programa o viceversa. Para sorventar esto vamos a crearles 


creamos un perfil en blanco
~~~
sudo aa-autodep nginx
    Writing updated profile for /usr/sbin/nginx.
~~~

le indicamos un el modo complain
~~~
sudo aa-complain nginx
    Setting /usr/sbin/nginx to complain mode.
~~~