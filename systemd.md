# systemd
Capas:
- BIOS / EFI
- Gestor de arranque: Boot loader
- Kernel (temp. initramfs)
- Proceso init: una vez todas las capas anteriores arrancadas comienza lo que se conoce como init system, que realiza todo el proceso de arranque de la máquina. Esto había dos ramas fundamentales en Unix, dos formas de hacerlo, uno de ellos era System V init. 

## Orígenes de systemd
Se plantea como sustituto de System V init (PID=1). 

System V era secuencial, por lo que se estaba quedando obsoleto porque era muy lento. Los procesos entre sí tiene relaciones (si es secuencial, no puedes levantar apache sin haber levantado la red primero, por ejemplo), pero hay otros procesos que no tienen relación y sí puedden levantarse simultáneamente.

Para obtener un arranque rápido en paralelo de los servicios en el espacio de usuario. Comienza su desarrollo en 2010.

Compite inicialmente con upstart que comenzó a desarrollarse en 2006. Por Ubuntu.

Englobado en el proyecto freedesktop.org.

La motivación de cambiar esto para un administrador de sistemas clásicos es irrelevante, porque el arranque del sistema de un servidor no se realiza ccontinuamente.

Pero finalmente se cambió. **upstart** fue inicialmente adoptado por muchos distros, pero fue cuando Debian tomó la decisión cuando se consiguió llegar a una determinación. Y se decidió que en Debian Jessie se use **systemd**. RedHat también optó por systemd y finalmente Ubuntu también cambió a systemd, dejando su proyecto, por no quedarse solo.

Adopcion de systemd:
- Fedora: Mayo 2011
- Arch Linux: Octubre 2012
- RHEL y CentOS: Junio/Julio 2014
- Ubuntu: Abril 2015
- Debian: Abril 2015

En paralelo, systemd sigue evolucionando. Y a su vez, en Debian se sigue tratando el tema el Debian, a través de votaciones.

## Funcionalidades iniciales
- Sustituye **/sbin/init** 
- Sustituye los niveles de ejecución (run-levels). Son los pasos, el orden, a la hora de ejecutar los demonios.
- Controla los demonios (olvídate de **/etc/init.d/**...o service)
- Desaparece **/etc/inittab**
- Sustituye **/etc/fstab**
- Se encarga de borrar los directorios temporales. Existen dos directorios temporales (/tmp y /var/tmp, este segundo no se borra al reiniciar). El directorio **/tmp** no se borra al apagar la máquina sino al encenderla.
- Gestiona los servicios en paralelo.
- Gestiona los procesos con **cgroups**. Los cgroups  limita los recursos a un conjunto de procesos que estén agrupados. De esta forma se controla el uso de los recursos. Esto está implementado en el kernel pero anteriormente se utilizaba a través de comandos sin System V en medio.
- Soporta instant ́aneas y restauración.

Otras funcionalidades:
- Las sesiones de usuario y las TTYs (systemd-logind)
- Syslog (systemd-journald)
- Dispositivos (systemd-udevd)
- Variables del kérnel (systemd-sysctl)
- Sesiones de usuario (systemd-user-sessions)
- Puntos de montaje (*.mount)
- Control de eventos como alternativa a cron (*.timer)
- Hora del sistema (systemd-timedated)
- Contenedores (systemd-nspawn)
- Interfaces de red (systemd-networkd)
- Chequeo del sistema de ficheros (systemd-fsck)
- Resolución de nombres (systemd-resolved)
- Volcados de memoria (coredumps)
- Cuotas (systemd-quotacheck)

> Críticas
###### No solo sustituye init, sino que se ha convertido en un supersistema interno que está reemplazando muchos componentes esenciales.

###### Rompe con las filosofía UNIX de pequeñas aplicaciones que realizan muy bien una tarea interactuando entre sí.

###### Incluye cada vez más dependencias que hacen difícil tener un sistema sin systemd.

###### Es exclusivo de Linux, no es portable, y no sirve en otros Unices.

## Características principales
- Activación de servicios basados en sockets
- Activación de servicios basados en D-Bus
- Activación de servicios basados en dispositivos
- Activación de servicios basados en ficheros
- Montaje y automontaje
- Compatible con SysV init

## Conceptos de systemd. Unidades
#### Unidades
Los que existe son unidades y la principal instrucción es **systemctl**.
Unidad (unit) Elemento básico de systemd asociado a un fichero de configuración.

- **systemctl status** info.
- **systemctl --failed [--all]** unidades que han fallado.
- **systemctl list-units** listar de unidades en memoria.
- **systemctl list-unit-files** listar unidades instaladas.

Las unidades se ubican en diferentes directorios:
- **/lib/systemd/system/** Al instalar los difernetes paquetes, estén o no habilitadas.
- **/run/systemd/** Creadas durante la ejecución del sistema. Tiene prioridad sobre el anterior. Para compatibilidad de system V es muy útil. 
- **/etc/systemd/system/** Unidades habilitadas.

Las unidades encargadas de la gestión de servicios tienen la extensión .service y se gestionan con systemctl (pero no todas las unidades son servicios):

Es lo mismo foo que foo.service
~~~
systemctl start foo | foo.service
systemctl stop foo | foo.service
systemctl restart foo | foo.service
systemctl status foo | foo.service
systemctl enable foo | foo.service
systemctl disable foo | foo.service
# forma de asegurarse que el servicio no se levanta aunque se invoque:
systemctl mask foo | foo.service
systemctl unmaskfoo | foo.service
# para script se utilizan los comandos posteriores, que son como preguntas que devuelve 0 si no está levantado y 1 si lo está en el caso de active, pero se puede hacer con enabled y failed:
systemctl is-enable foo | foo.service
systemctl is-active foo | foo.service
systemctl is-failed foo | foo.service
~~~

#### Target
Conjunto de unidades. Los niveles de ejecución se sustituyen en systemd por **targets**, pero se usan con más fines. Cada distro puede definir sus propios targets. 
- **basic.target**: unidades para el inicio del sistema.
- **multi-user.target**: agrupa la mayoría de los servicios no esenciales. Equivale a los niveles de ejecución 2-4. 
- **graphical.target**: agrupa los servicios del entorno gráfico. Equivalente al nivel de ejecución 5.
~~~
systemctl  get -default
systemctl  list -units  --type  target [--all]
systemctl  set -default  nombre.target
systemctl  isolate  nombre.target
systemctl [--no-wall] rescue
systemctl [--no-wall] emergency
~~~

#### Pociones
Utilizado para gestionar grupos de procesos creando nodos en el árbol de Linux cgroups, por lo que sepueden aplicar límites a los recursos utilizados.
- **system.slice**: Utilizado por defecto por servicios
- **user.slice**: Jerarquía de porciones de recursos deusuarios (los hijos se representan con‘‘-’’)
- **user-[].slice**: Sesiones de usuarios gestionadaspor systemd-logind
~~~
systemctl  list -units  --type  slice [--all
~~~

#### Puntos de montaje (mount)
Define unidades para los puntos demontaje de los sistemas de ficheros en unidades conextensión .mount. Sustituye así las entradas de **/etc/fstab**, pero por compatibilidad permite definir allí las entradas que se traducen en tiempo de ejecución a unidades en **/run/systemd** 

**automount**: Para montaje automático (requiere una unidad .mount y otra .automount)
**Intercambio (swap)**: Define unidades para los dispositivos o ficheros de swap. Al igual que en el caso anterior, pueden seguir definiéndose en **/etc/fstaby** se crean las correspondientes unidades durante la ejecución delsistema.

#### Temporizadores (timer)
Como alternativas a la definici ́on de procesosdecron. Hay de dos tipos:
- Definidos conforme a fechar fijas, como los cronjobs.
- Definido por un intervalo de tiempo tras un evento.
~~~
systemctl  list -timers
NEXT      LEFT            LAST      PASSED         UNIT                  ACTIVATES
Sun  ...   3h 35min  left  Sun  ...   10h ago       apt -daily.timer     apt -daily.service
Mon  ...   18h left        Sun  ...   4h 47min  ago apt -daily -upgra ... apt -daily -upgrade.service
Mon  ...   22h left        Sun  ...   1h 29min  ago  systemd -tmpfile ...  systemd -tmpfiles -clean.service
~~~

#### Apagar / suspender / hibernar / reiniciar
systemd también controla las funciones de apagado y reinicio, juntocon la suspensión (guarda estado en memoria) o la hibernación (guarda el estado en disco) del sistema:
~~~
systemctl [--no-wall] [-f] halt
systemctl [--no-wall] [-f] poweroff
systemctl [--no-wall] [-f] reboot
systemctl  suspend
systemctl  hibernate
systemctl  hybrid -sleep
~~~

#### timedatectl
Instrucción utilizada para consultar y fijar la hora del sistema.
~~~
timedatectl
Local  time: dom  2017 -12 -10  12:14:57  CET
Universal  time: dom  2017 -12 -10  11:14:57  UTC
RTC  time: dom  2017 -12 -10  11:14:57
Time  zone: Europe/Madrid (CET , +0100)
Network  time on: yes
NTP  synchronized: yes
RTC in  local  TZ: noIncluye ntp integrado a trav ́es desystemd-timesyncd.service
~~~

Incluye ntp integrado a través de systemd-timesynd.service.


#### loginctl
Puesto (seat): Hardware asociado a un puesto de trabajo.

Sesión: Definida por el tiempo entre el ingreso y la salida del usuario.

systemd-logind permite definir sistemas multisesión y/o multipuesto.

loginctl es la instrucción para controlar las sesiones, puestos y demás aspectos relacionados:
~~~
loginctl  list -users
loginctl  list -seats
loginctl  list -sessions
loginctl  user -status [USER]...
~~~

# Registros de log
Entrada - stdin
Salida - stdout u stderr. Y son salidas diferentes. En programación cuando se capturan errores o respuestas los mensajes hay que mandarlos a una salida u otra según el tratamiento que va a tener. 

#### Formato del fichero /etc/syslog.conf
Cada línea de este fichero tiene dos campos: selector y action.Escala de prioridad:
- debug
- info
- notice
- warning (warn)
- error (err)
- crit
- alert
- panic (emerg)

El formato por defecto es syslog, pero hay aplicaciones que utilizan, por sus características, otros formatos. 

#### Journalctl
> Ejemplos
Muestra los mensajes de prioridad error:
~~~
journalctl -p err
~~~

Muestra los mensajes de una unidad concreta:
~~~
journalctl -u ssh
~~~


## Practiquita: configurar la hora del sistema con systemctl
