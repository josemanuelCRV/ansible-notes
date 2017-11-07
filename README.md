
![Ansible][img-Ansible]
***
# Introducción a ANSIBLE

- *[¿Para qué usamos _ANSIBLE_?][qw1]*
- *[¿Cómo funciona ANSIBLE?][qw1]*
- *[Instalación de ANSIBLE][qw1]*
- *[Archivos de configuración][qw1]*
- *[ANSIBLE - Inventory][qw1]*
- *[Definiendo nuevos archivos de configuración][qw1]*
- *[Comandos básicos ad-hoc][qw1]*
- *[Comandos ad-hoc para controlar módulos][qw1]*
- *[Redactando Playbooks][qw1]*
- *[Conectarse al host remoto con otro usuario][qw1]*
- *[Handlers][qw1]*

[Fuentes][qw1]




**Tutorial:** https://www.youtube.com/watch?v=slNIwBPeQvE&list=PLTd5ehIj0goP2RSCvTiz3-Cko8U6SQV1P
***
---

### _Capítulo 1_.
### ¿Para qué usamos _ANSIBLE_?
---
Permite aprovisionar los servidores con todos los servicios y recursos de software que nos vayan a ser necesarios para que la máquina sea útil.

> Ejemplo: 
- Alquilamos un servidor en DigitalOcean y sólo nos viene con una instalación de Ubuntu 16.04.
- Necesitamos instalar manualmente todo lo que vaya a necesitar (_NGINX, redis, Elasticsearch,_...)

Podríamos hacer este aprovisionamiento a través de shellScript (bin\sh) , escribiendo los comandos en un script y fuese ejecutando comando tras comando. 
```sh
 #!/bin/sh
 apt-get update
 apt-get install -y php5 -fpn
```

#### ¿Cómo funciona _ANSIBLE_?
---
___Es declarativo:___ 
Al ser declarativo, no le vamos a indicar a Ansible los pasos que debe seguir en el aprovisionamiento para preparar el servidor.
En su lugar, le indicamos el estado en el que queremos encontrarnos el servidor cuando termine. 
> Ejemplo:
    - El paquete nginx debe estar instalado en este sistema.
    - El usuario webapplication  debe estar creado.

_ANSIBLE_, en su arranque, automáticamente verificará el estado del servidor y seguidamente correrá una serie de comandos, que él conoce, para realizar las tareas que se le haya solicitado, _(abstrayéndonos de conocer los comandos de cada tarea con precisión)_.

___Gestiona los errores:___
No requiere que tengamos que tratar manualmente los casos en los que puede producirse un error durante la instalación del aprovisionamiento. Por ejemplo:
- Verificar si el servidor ya tiene nginx instalado y no requiere instalar el paquete...
- Ahorra realizar tareas repetitivas, como verificar si ya está instalado un determinado paquete.
```sh
dpkg -s $PKG 2> /dev/null | sudo apt-get install -Y $PKG

pacman -Qi $PKG 2>/dev/null | sudo pacman -S $PKG
if[ -f /etc/debian_version ]; then 
    OS=Debian
    dpkg -s $PKG 2>/dev/null | sudo apt-get install $PKG 
elif [ -f /etc/arch_release ]; then
    OS=Arch
    pacman -Qi $PKG 2>/dev/null | sudo pacman -S $PKG
fi
```
___Usa módulos___
Ansible está organizado en módulos. Cuenta con un gran catálogo de módulos que encapsulan tareas,  permitiéndonos realizar gran variedad de cosas y abstrayendonos de necesitar conocer el cómo se hace y qué comandos requiere cada tarea.
[Índice de categorías de módulos](http://docs.ansible.com/ansible/latest/modules_by_category.html)

___Repositorio Galaxy___
En el caso de no encontrar el módulo que necesitemos en el catálogo, podemos acudir al repositorio [_GALAXY_](https://galaxy.ansible.com/explore#/), donde cualquiera persona puede enviar sus propios proyectos "_playbooks_" para su reutilización.

___Otras consideraciones___
- Software libre (GPL3)
- Solo requiere instalar Python (al ser el leguaje con el que está desarrollado) y SSH (para la comunicación con el servidor)
- Muy portable, funciona en cualquier UNIX. 
(*No permitido para Windows-7,8,Vista.., salvo en Windows-10, siendo compatible gracias al subsistema Linux que trae instalado,"Bash".)
- Muy económico, solo requiere instalar un software en donde se vaya a realizar el deploy, sin necesidad de instalar servidores intermedios, ni servidores de Ansible para el procesado de las tareas, etc.
#
#
---


#### _Capítulo 2_.
### Instalación de ANSIBLE
---
Vamos a utilizar el subsistema de Linux (Bash) distribuido para Windows 10.
> ***¿Cómo configurar las máquinas a las que vamos a conectarnos para aprovisionarlas?***

Podemos utilizar un servidor real o un servidor en una máquina virtual.
Antes de nada, deberíamos configurar la forma de conexión con el servidor con autenticación mediante clave _SSH_. 
La forma tradicional para autenticarse frente a un servidor suele ser con usuario/contraseña, siendo menos eficiente para nuestros propósitos y más inseguro que _SSH_.

Una de las ventajas mediante _SSH_ es conectarnos automáticamente, sin necesidad de introducir la contraseña para conectarnos continuamente. Algo que es muy práctico para Ansible ya que muchos de los comandos que necesitará ejecutar los realizará de forma remota sobre un servidor.

***Comenzando...***
Desde nuestro subsistema en Windows 10, vamos a ***verificar qué distribución tenemos instalada***:
```
josemanuel@Arrakis:~$ cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=16.04
DISTRIB_CODENAME=xenial
DISTRIB_DESCRIPTION="Ubuntu 16.04.2 LTS"
```

> ***Añadimos el repositorio de _ANSIBLE_ a nuestro equipo.***
```
josemanuel@Arrakis:~$ sudo add-apt-repository ppa:ansible/ansible
[sudo] password for josemanuel:
 Ansible is a radically simple IT automation platform that makes your applications and systems easier to deploy. Avoid writing scripts or custom code to deploy and update your applications— automate in a language that approaches plain English, using SSH, with no agents to install on remote systems.
```

> ***Descargamos _ANSIBLE_***
```
josemanuel@Arrakis:~$ sudo apt update
```

> ***Instalamos _ANSIBLE_***
```
josemanuel@Arrakis:~$ sudo apt install ansible
```
Las únicas dependencias que tiene Ansible son python2 y la librería _SSH_.
_En esta versión de Ubuntu viene instalado python3, no obstante descargará python2._

La primera vez, posiblemente comenzará instalando otros paquetes, adicionales y sugeridos, correspondientes a esas dependencias, como python-libdev2, python-crypto,etc..., utilizados para la comunicación _SSH_ principalmente.
```
Se instalarán los siguientes paquetes NUEVOS:
ansible libpython-stdlib libpython2.7-minimal libpython2.7-stdlib python python-cffi-backend python-crypto python-cryptography python-ecdsa python-enum34 python-httplib2 python-idna python-ipaddress python-jinja2 python-markupsafe python-minimal python-paramiko python-pkg-resources python-pyasn1 python-setuptools python-six python-yaml python2.7 python2.7-minimal sshpass
0 actualizados, 25 nuevos se instalarán, 0 para eliminar y 0 no actualizados.
Se necesita descargar 7.927 kB de archivos.
Desempaquetando python-cryptography (1.2.3-1ubuntu0.1) ...
Seleccionando el paquete ansible previamente no seleccionado.
Preparando para desempaquetar .../ansible_2.4.1.0-1ppa~xenial_all.deb ...
```
---

> ***Verificando la instalación***
Antes de continuar, vamos a verificar qué versión de Ansible se descargó y si ya está reconocido por nuestro equipo tras la instalación. 

```
josemanuel@Arrakis:/etc/ansible$ ansible --version
ansible 2.4.1.0
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/josemanuel/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.12 (default, Nov 19 2016, 06:48:10) [GCC 5.4.0 20160609]
josemanuel@Arrakis:/etc/ansible$

```

> ***Algunos parámetros de Ansible:***
```
josemanuel@Arrakis:~$ ansible

  Privilege Escalation Options:
    control how and which user you become as on target hosts

    -s, --sudo          run operations with sudo (nopasswd) (deprecated, use
                        become)
    -U SUDO_USER, --sudo-user=SUDO_USER
                        desired sudo user (default=root) (deprecated, use
                        become)
    -S, --su            run operations with su (deprecated, use become)
    -R SU_USER, --su-user=SU_USER
                        run operations with su as this user (default=None)
                        (deprecated, use become)
    -b, --become        run operations with become (does not imply password
                        prompting)
    --become-method=BECOME_METHOD
                        privilege escalation method to use (default=sudo),
                        valid choices: [ sudo | su | pbrun | pfexec | doas |
                        dzdo | ksu | runas | pmrun ]
    --become-user=BECOME_USER
                        run operations as this user (default=root)
    --ask-sudo-pass     ask for sudo password (deprecated, use become)
    --ask-su-pass       ask for su password (deprecated, use become)
    -K, --ask-become-pass
                        ask for privilege escalation password

Some modules do not make sense in Ad-Hoc (include, meta, etc)
ERROR! Missing target hosts
josemanuel@Arrakis:~$
```


---

#### _Capítulo 3_.
### Archivos de configuración
---

Podemos ver tras la instalación que se han creado 3 nuevos archivos en `/etc/ansible/`:
```
josemanuel@Arrakis:/etc/ansible$ ls
ansible.cfg  hosts  roles
```

> ***ansible.cfg***
Este archivo contiene una plantilla sobre la configuración global de Ansible. 

Cuando ejecutamos Ansible, ya sea para lanzar un playbook o un simple comado por consola, Ansible buscará en el directorio actual los archivos de configuración que necesita para ejecutarse, en caso de no encontrarlos en el path desde donde lanzamos el comando, los buscará en la ruta definida por defecto en este archivo ***ansible.cnfg*** : 
`inventory      = /etc/ansible/hosts`

- Veamos un extracto del archivo de configuración por defecto con los valores que tomará Ansible al ejecutarse, siempre que no se haya indicado que cargue otros archivos de configuración desde otro path,  o bien sobrescribiendo los valores con _flags_ en la línea de comandos o en un ansible-playbook.

```
josemanuel@Arrakis:/etc/ansible$ cat ansible.cfg
# config file for ansible -- https://ansible.com/
# ===============================================

# nearly all parameters can be overridden in ansible-playbook
# or with command line flags. ansible will read ANSIBLE_CONFIG,
# ansible.cfg in the current working directory, .ansible.cfg in
# the home directory or /etc/ansible/ansible.cfg, whichever it
# finds first

[defaults]

# some basic default values...

#inventory      = /etc/ansible/hosts
#library        = /usr/share/my_modules/
#module_utils   = /usr/share/my_module_utils/
#remote_tmp     = ~/.ansible/tmp
#local_tmp      = ~/.ansible/tmp
#forks          = 5
#poll_interval  = 15
#sudo_user      = root
#ask_sudo_pass = True
#ask_pass      = True
#transport      = smart
#remote_port    = 22
#module_lang    = C
#module_set_locale = False
```

---
> ***hosts***: representa al ***Inventory***

```
# This is the default ansible 'hosts' file.
#
# It should live in /etc/ansible/hosts
#
#   - Comments begin with the '#' character
#   - Blank lines are ignored
#   - Groups of hosts are delimited by [header] elements
#   - You can enter hostnames or ip addresses
#   - A hostname/ip can be a member of multiple groups

# Ex 1: Ungrouped hosts, specify before any group headers.

## green.example.com
## blue.example.com
## 192.168.100.1
## 192.168.100.10

# Ex 2: A collection of hosts belonging to the 'webservers' group

## [webservers]
## alpha.example.org
## beta.example.org
## 192.168.1.100
## 192.168.1.110

# If you have multiple hosts following a pattern you can specify
# them like this:

## www[001:006].example.com
```

> ***roles/*** : directorio por defecto para definir los archivos de roles.

Ejemplo de roles definidos en un playblok:
```
- name: Ansible Playbook for configuring brand new Raspberry Pi

  hosts: pis
  roles:
    - pi
  remote_user: pi
  sudo: yes
```
#

---

#### _Capítulo 4_.
### ANSIBLE - Inventory
---
El inventario, como propiamente indica, es el listado de cada uno de los Host, IPs, máquinas que tenemos que aprovisionar.

A través del _Inventory_ le indicaremos a _Ansible_ cuáles son las máquinas en las que tenemos que actuar. Podemos configurarlo de varias formas, como por ejemplo:
Tenemos un archivo en el directorio ***etc/asnible/hosts***  que contiene el inventario global, que muestra un ejemplo "a modo template" donde aparecen las supuestas máquinas que tenemos para administrar.



> ***Extracto de inventario con 4 máquinas:***
```
green.example.com
blue.example.com
192-168.100.1
192-168.100.10
```
> ***Uso de rangos para simplificar los inventarios repetitivos:***
```
## Varios nodos que siguen un patrón
db1.example.com
db2.example.com
db3.example.com

## Usando rangos para simplificar (equivalente a lo anterior)
db[1:3].example.com
```
> ***Agrupando nuestro inventario:***
Si tenemos un inventario complejo, la agrupación es útil para realizar ciertas tareas de forma global sobre todas las máquinas de un determinado grupo (parchear, instalar certificados, instalar nuevos servidores,..)
Para crear la agrupación, escribiremos el nombre del grupo entre corchetes en una línea nueva:
```
[servidoresweb]
green.example.com
blue.example.com
192-168.100.1
192-168.100.10

[servidoresbbdd]
db[1:3].example.com
```
#
---


#### _Capítulo 5_.
### Definiendo nuevos archivos de configuración
---
Para utiliza nuestros propios archivos de configuración debemos indicarle a Ansible la ruta donde se encuentren.


> ***Definiendo nuestro archivo hosts***
Para este ejemplo, crearemos un nuevo directorio donde definiremos nuestros archivos personalizados de configuración. Por ejemplo `/myAnsible`.
```
josemanuel@Arrakis:~/myAnsible$ 
```
Dentro del nuevo directorio crearemos un archivo llamado ***rasphosts.txt***, que representará a nuestro ***_Invetory_*** con el conjunto de máquinas que queremos aprovisionar. 
```
josemanuel@Arrakis:~/myAnsible$ touch rasphosts.txt
josemanuel@Arrakis:~/myAnsible$ ls
rasphosts.txt
```

- Para conectar con servidores, es necesario que tengan soportada la conexión vía _SSH_.
- Para esta ocasión, conectaremos con una Raspberry PI 3, conectada a la red local mediante wifi.

***Para conectar con la Raspberry*** necesitamos indicar a Ansible con qué usuario remoto vamos a conectar, así como sus credenciales, ya que Ansible ejecuta los comandos como `whoami`, es decir, como el usuario que ha ejecutado el comando.

En el siguiente extracto de nuestro archivo `hosts` podemos ver que se ha definido un grupo llamado `[raspis]` que contiene una única máquina. También se ha añadido un grupo de variables `[raspis:vars]` que establecen valores de configuración sobre el grupo creado. En este caso son propiedades para la conexión _SSH_ con la Raspberry.

En nuestro archivo ***_hosts_***:
```
josemanuel@Arrakis:~/myAnsible$ ls
rasphosts.txt

josemanuel@Arrakis:~/myAnsible$ cat rasphosts.txt
[raspis] 
192.168.1.XXX

[raspis:vars]
ansible_ssh_user=pi
ansible_ssh_pass=p@ssw0rd
ansible_ssh_port=22
```

> ***Utilizando el nuevo inventario***
Con el _flag_  ***`-i` seguido del _path_***,   indicamos a Ansible dónde se encuentra el archivo de _hosts_ con las configuraciones que queremos que cargue.
```
 -i rasphosts.txt
```

Si lanzásemos el comando Ansible fuera del directorio donde se encuentra nuestro archivo hosts, debemos indicar la ruta completa:

```
 -i /home/josemanuel/myAnsible/rasphosts.txt
```
#
---
#### _Capítulo 6_.
### Comandos básicos ad-hoc
---
En Ansible se indica mediante la función Task las tareas o acciones que queremos que realice.
Se escriben de una forma declarativa, indicando cómo se supone que debería quedar el sistema una vez termine de aprovisionar las máquinas. Ansible correrá distintas operaciones para que al finalizar coincida a como le indicamos.

Las tareas, normalmente, se escriben en archivos llamados ___Playbooks___.
Para realizar acciones sencillas y directas podemos utilizar ___comandos ad-hoc.___
Se trata de escribir `ansible` seguido de una orden de un solo uso y directa a algún Host dentro del inventario.

> ***Verificando si el Host está levantado***
En este ejemplo, vamos a indicar a Ansible que realice un ping a las máquinas que estén definidas dentro del grupo `raspis`, obteniendo la configuración necesaria de nuestro archivo personalizado de _hosts_, rasphosts.txt.  
```
josemanuel@Arrakis:/$ ansible raspis -m ping -i /home/josemanuel/myAnsible/rasphosts.txt
192.168.1.XXX | SUCCESS => {
    "changed": false,
    "failed": false,
    "ping": "pong"
}
josemanuel@Arrakis:/$
```
#
---

#### _Capítulo 7_.
### Comandos ad-hoc para controlar módulos
---
El _flag_ `-m ping` que acabamos de utilizar en el paso anterior hace referencia al ***módulo Ping***. 
Si no indicamos el nombre del módulo, por defecto usará el [módulo Shell](http://docs.ansible.com/ansible/latest/shell_module.html).

```
## Sin indicar módulo
josemanuel@Arrakis:/$ ansible raspis -a 'echo hola' -i /home/josemanuel/myAnsible/rasphosts.txt
192.168.1.XXX | SUCCESS | rc=0 >>
hola
josemanuel@Arrakis:/$

josemanuel@Arrakis:~$ ansible localhost.com -a 'true'
josemanuel@Arrakis:~$ ansible localhost.com -a 'false'
josemanuel@Arrakis:~$ ansible localhost.com -a 'ls/'
josemanuel@Arrakis:~$ ansible localhost.com -a 'uname'
```

> ***Combinando módulos, parámetros => Instalando Paquetes***  

¿Está vim instalado en mi máquina? 
No..
¿Cómo decir a Ansible  que me instale vim? 
Con el uso del [módulo apt](http://docs.ansible.com/ansible/latest/apt_module.html). 

| parameter | choices | 
| ------ | ------ | 
| name |  |
| state | latest, absent, present, build dep |
| update cache | yes (asegura que está actualizada la última versión de bd de paquetes = apt-get update) |

```
josemanuel@Arrakis:~$ ansible localhost.com -m apt -a 'name=vim state present' -b -K
```
Para instalar paquetes, apt nos solicitará disponer de permisos de super usuario, root.
`-b => becom:` cambia a usuario root antes de ejecutar el comando.
`-K`=> Nos solicitará una contraseña del servidor donde intentamos instalar vim.


>***Verificando la instalación en el sistema remoto.***

***Conectándome a la Raspberry PI por protocolo _SSH_ desde el subsistema de Windowds-10.***
```
josemanuel@Arrakis:/etc/ansible$ ssh pi@192.168.1.XXX
```
Introducimos vim y recibiremos el mensaje
```
josemanuel@localhost:~$ vim
OK!
```
#
---

#### _Capítulo 8_.
### Redactando Playbooks
---
Hemos visto como realizar acciones simples y directas en el servidor mediante comandos ad-hoc. 
Veremos a continuación la forma de aprovisionar las máquinas del _inventory_ mediante ***_Playbooks_***, permitiendonos realizar tareas complejas.

>***Escribiendo Playbooks***
Los playbooks se escriben en lenguaje `.yml`, y su estructura se organiza mediante clave:valor.
```yml
 ## 3 guiones indica archivo .yml
---
- hosts: green.example.com  ## acepta grupos como [servidorersweb] o [all]
  tasks:
   - name: instala vim
     apt: name=vim state=present
     become: true ## lanza la tarea de instalación con permisos de super-usuario
   - name: saluda
     shell: echo hola!
```

>***_Lanzando el playbook_***
```
josemanuel@Arrakis:~$ ansible-playbook tareas.yml -K
```
Lo primero que realiza es recoger "hechos" o `facts` del estado del sistema.
Una vez que termina de ejecutar las tareas nos informará de qué ha cambiado.
```
PLAY [all] ******************************************************************************
TASK [Gathering Facts] ******************************************************************
ok: [localhost.com]

TASK [instala vim] **********************************************************************
changed: [localhost.com]

TASK [saluda] ***************************************************************************
changed: [localhost.com]

PLAY RECAP ******************************************************************************
localhost.com           : ok=3          changed=2       unracheable=0   failed=0
```
Si volvemos a ejecutar el playbook, en lugar de mostrar en el resultado de la tarea de instalación `changed: [localhost.com]` mostrará `ok: [localhost.com]`, dado que vim ya está instalado en esa máquina.

> ***Añadiendo tasks***
Ciertas acciones requieren permisos de super-usuario. Como por ejemplo, lanzar un comando de parada/arranque de un servidor. En el siguiente playbook se asigna permisos root globales, justo antes de definir las tareas, ya que cada una de ellas realiza acciones que lo requieren.
Las tareas de control de servicios (started,stopped,sleep,...) sobre host remotos las facilita el [módulo service](http://docs.ansible.com/ansible/latest/service_module.html).

```yml
---
- hosts: servidorersweb
  become: true
  tasks:
  - name: instala vim
     apt: name=vim state=present

  - name: detiene nginx
     service: name=nginx state=started
```


---
#### _Capítulo 9_.
### Conectarse al host remoto con otro usuario
---
En algunas ocasiones, disponemos de una infraestructura generada por alguna herramienta para la creación y configuración de entornos de desarrollo virtualizados, como puede ser [Vagrant], Docker-AWS,..., donde nos encontramos que el nombre de usuario de dicha máquina viene predefinido por la herramienta, como puede ser el caso de Vagran, donde debemos informar a Ansible que utilice un nombre de usuario específico, ya sea indicándolo a través de una variable de entorno, un _flag_ por línea de comandos, en un archivo de configuración `ansible.cfg`, o bien, directamente dentro de un playbook.


La decisión sobre dónde definir el usuario remoto dependerá sobre dónde queremos que afecte. Sólo en la configuración local, con una variable de entorno (ASIBLE_CONFIG), o bien de forma general a través de un playbook, al compartirlo con otros usuarios.



> ***Indicando otro usuario por línea de comandos, flag -u***
Una de las formas para indicar a Ansible que utilice otro usuario para conectarse al host remoto es pasándole como parámetro del comando el _flag_  `-u` , seguido del nombre de usuario de la máquina remota.
```
josemanuel@Arrakis:/etc/ansible$ ansible 192.168.1.XXX -m ping -u pi
192.168.1.XXX | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: Permission denied (publickey,password).\r\n",
    "unreachable": true
}
```
> ***Indicando otro usuario en el archivo de configuración ansible.cfg***
Otra forma de indicarle a Ansible el usuario remoto con el que debe realizar la conexión es a través del archivo ***ansible.cfg***. Podemos crear y personalizar dicho archivo y ubicarlo al alcance de Ansible.

Ansible seguirá el siguiente orden de búsqueda para procesar el archivo:

* ANSIBLE_CONFIG (una variable de entorno) 
* ansible.cfg (en el directorio actual) 
* .ansible.cfg (en el directorio de inicio)
* /etc/ansible/ansible.cfg (en el directorio global de Ansible)


Vamos a configurar nuestro propio archivo de configuración dentro del directorio utilizado para el ejemplo  `/myAnsible`, desde donde lanzaremos nuestro comando posteriormente.

***Importante***,  no hay que olvidar, al principio del archivo, indicar el grupo ***[defaults]***, dado que con este grupo estaremos indicando a Ansible que debe sobrescribir la configuración global por defecto de Ansible con los nuevos valores definidos.
```
josemanuel@Arrakis:~/myAnsible$ cat ansible.cfg
[defaults]
remote_user = vagrant
```

> ***Indicando otro usuario en un playbook***

Podemos crear el siguiente _playbook.yml_ y ejecutarlo:
```
---
- hosts: all
  remote_user: vagrant
  tasks:
  - name: lanzar un ping
    ping:
```

```
josemanuel@Arrakis:~/myAnsible$ ansible-playbook -i rasphosts.txt playbook.yml
```


> ***Indicando otro usuario en una variable de entorno***

Podemos asignar el valor de nuestro archivo de configuración directamente en la variable de entorno de Ansible:
```
josemanuel@Arrakis:~/myAnsible$ ANSIBLE_CONFIG=ansible.cfg -i rasphosts.txt raspis -m ping
```


---
#### _Capítulo 10_.
### Handlers
---
Son mecanismos que nos permiten pedirle a Ansible que cuando termine de forma satisfactoria una tarea nos notifique para que desencadene otras tareas de forma paralela. 
Los handlers también son tareas, tienen la misma sintaxis,  tienen módulos, parámetros,etc...
La diferencia está en que no se ejecutan de forma secuencial/arriba-abajo cuando les toca, sino que son llamados de forma paralela cuando definimos en una _Task_ una lista de _handlers_ a los que debe notificar cuando finalice satisfactoriamente esa tarea.

> ***Definiendo un Handler***
Vamos a crear un _playbook_ con alguna tarea en la que definiremos un _handler_ asociado:
```
---
- hosts: all
  remote_user: vagrant
  become: true
  task:
  - name: Instala Apache2
    apt: name=Apache2 state=present update_cache=true
    notify:
        - "Reinicia el servidor web"

  - handlers:
      - name: Reinicia el servidor web
      - service: name=Apache2 state=restarted
```
Ejecutando el playbook:
```
josemanuel@Arrakis:~/myAnsible$ ansible-playbook -i rasphosts.txt playbook.yml -K
```

> ***Analizando el playbook...***

Como podemos ver, el playbook se conecta con el usuario remoto predefinido por _vagrant_ a todas las máquinas de nuestro inventario. Seguidamente, invocará la tarea `Instala Apache2`, la cual, al ser un comando `apt`, requiere permisos de super-usuario `become:true` para ser ejecutada. Además, se pide que actualice nuestra cache del repositorio local `update_cache=true`. 

Si la tarea finaliza correctamente, la clave definida como  _`notify:`_ se lanzará e invocará a la tarea  `handlers:` definida como  `name: Reinicia el servidor web`

Si ejecutamos el playbook nuevamente, al no necesitar volver a instalar Apache2, el `Handler` no se va a ejecutar. Ya que sólo se ejecuta en caso que la tarea finalizase satisfactoriamente. Por el contrario, si lo hubiésemos definido como otra tarea, Ansible lo hubiese ejecutado, al no estar vinculada a ninguna condición. 

 ---
 ![](http://blog.itlinux.cl/images/ansible2014_logo_black_tagline.png)


[img-Ansible]: https://upload.wikimedia.org/wikipedia/commons/0/05/Ansible_Logo.png "Icono ANSIBLE"

[Vagrant]: <https://www.vagrantup.com/>

[qw1]: <https://github.com/josemanuelCRV/ansible-notes/blob/master/README.md#¿Para-qué-usamos--_ANSIBLE_?>


