Virtualización *ligera* usando contenedores
===

<!--@
prev: PaaS
next: Gestion_de_configuraciones
-->

<div class="objetivos" markdown="1">

Objetivos
--

### Cubre los siguientes objetivos de la asignatura

* Conocer las diferentes tecnologías y herramientas de virtualización tanto para procesamiento, comunicación y almacenamiento. 
* Instalar, configurar, evaluar y optimizar las prestaciones de un servidor virtual.
* Configurar los diferentes dispositivos físicos para acceso a los
  servidores virtuales: acceso de usuarios, redes de comunicaciones o entrada/salida.
* Diseñar, implementar y construir un centro de procesamiento de datos virtual.
* Documentar y mantener una plataforma virtual.
* Optimizar aplicaciones sobre plataformas virtuales. 
* Conocer diferentes tecnologías relacionadas con la virtualización
  (Computación Nube, Utility Computing, Software as a Service) e
  implementaciones tales como Google AppSpot, OpenShift o Heroku.
* Realizar tareas de administración en infraestructura virtual.

### Objetivos específicos

1. Entender cómo las diferentes tecnologías de virtualización se integran en la creación de contenedores. 

2. Crear infraestructuras virtuales completas.

3. Comprender los pasos necesarios para la configuración automática de las mismas.

</div>

Un  paso más hacia la virtualización completa: *contenedores*
-------

El aislamiento de grupos de procesos formando una *jaula* o
*contenedor* ha sido una característica de ciertos sistemas operativos
de la rama Unix desde los años 80, en forma del programa
[chroot](https://es.wikipedia.org/wiki/Chroot) (creado por Bill Joy, el
que más adelante sería uno de los padres de Java). La restricción de
uso de recursos de las *jaulas `chroot`*, que ya hemos visto, se limitaba a la protección
del acceso a ciertos recursos del sistema de archivos, aunque son
relativamente fáciles de superar; incluso así, fue durante mucho
tiempo la forma principal de configurar servidores de alojamiento
compartidos y sigue siendo una forma simple de crear virtualizaciones *ligeras*. Las
[jaulas BSD](https://en.wikipedia.org/wiki/FreeBSD_jail) constituían un
sistema más avanzado, implementando una
[virtualización a nivel de sistema operativo](https://en.wikipedia.org/wiki/Operating_system-level_virtualization)
que creaba un entorno virtual prácticamente indistinguible de una
máquina real (o máquina virtual real). Estas *jaulas* no sólo impiden
el acceso a ciertas partes del sistema de ficheros, sino que también
restringían lo que los procesos podían hacer en relación con el resto
del sistema. Tiene como limitación, sin embargo, la obligación de
ejecutar la misma versión del núcleo del sistema.

<div class='nota' markdown='1'>

En
[esta presentación](http://www.slideshare.net/dotCloud/scale11x-lxc-talk-16766275)
explica como los espacios de nombres son la clave para la creación de
contenedores y cuáles son sus ventajas frente a otros métodos de
virtualización

</div>


El mundo Linux no tendría capacidades similares hasta bien entrados los años 90, con
[vServers, OpenVZ y finalmente LXC](https://en.wikipedia.org/wiki/Operating_system-level_virtualization#Implementations). Este
último, [LXC](https://linuxcontainers.org/), se basa en el concepto de
[grupos de control o CGROUPS](https://en.wikipedia.org/wiki/Cgroups),
una capacidad del núcleo de Linux desde la versión 2.6.24 que crea
*contenedores* de procesos unificando diferentes capacidades del
sistema operativo que incluyen acceso a recursos, prioridades y
control de los procesos. Los procesos dentro de un contenedor están
*aislados* de forma que sólo pueden *ver* los procesos dentro del
mismo, creando un entorno mucho más seguro que las anteriores
*jaulas*. Estos [CGROUPS han sido ya vistos en otro tema](Intro_concepto_y_soporte_fisico.md). 

Dentro de la familia de sistemas operativos Solaris (cuya última
versión libre se denomina
[illumos](https://en.wikipedia.org/wiki/Illumos), y tiene también otras
versiones como SmartOS) la tecnología
correspondiente se denomina
[zonas](https://en.wikipedia.org/wiki/Solaris_Zones). La principal
diferencia es el bajo *overhead* que le añaden al sistema operativo y
el hecho de que se les puedan asignar recursos específicos; estas
diferencias son muy leves al tratarse simplemente de otra
implementación de virtualización a nivel de sistema operativo.

Un contenedor es, igual que una jaula, una forma *ligera* de virtualización, en el sentido
que no requiere un hipervisor para funcionar ni, en principio, ninguno
de los mecanismos hardware necesarios para llevar a cabo
virtualización. Tiene la limitación de que la *máquina invitada* debe
tener el mismo kernel y misma CPU que la máquina anfitriona, pero si
esto no es un problema, puede resultar una alternativa útil y ligera a
la misma. A diferencia de las jaulas, combina restricciones en el
acceso al sistema de ficheros con otras restricciones aprovechando
espacios de nombres y grupos de control. `lxc` es la solución de
creación de contenedores más fácil de usar hoy en día en Linux.

<div class='ejercicios' markdown="1">

Instala LXC en tu versión de Linux favorita. Normalmente la versión en
desarrollo, disponible tanto en [GitHub](http://github.com/lxc/lxc)
como en el [sitio web](http://linuxcontainers.org) está bastante más
avanzada; para evitar problemas sobre todo con las herramientas que
vamos a ver más adelante, conviene que te instales la última versión y
si es posible una igual o mayor a la 2.0. 

</div>

Esta virtualización *ligera* tiene, entre otras ventajas, una
*huella* escasa: un ordenador normal puede admitir 10 veces más contenedores
(o *tápers*) que máquinas virtuales; su tiempo de arranque es de unos
segundos y, además, tienes mayor control desde fuera (desde el anfitrión) del que se pueda
tener usando máquinas virtuales. 

Usando `lxc`
--

No todos los núcleos del sistema operativo pueden usar este tipo de container; para empezar,
dependerá de cómo esté compilado, pero también del soporte que tenga
el hardware. `lxc-checkconfig` permite comprobar si está preparado
para usar este tipo de tecnología y también si se ha configurado correctamente. Parte de la configuración se
refiere a la instalación de `cgroups`, que hemos visto antes; el resto
a los espacios de nombres y a capacidades *misceláneas* relacionadas
con la red y el sistema de ficheros. 

![Usando lxc-chkconfig](../img/lxcchkconfig.png)

Hay que tener en cuenta que si no aparece alguno de esas capacidades
como activada, LXC no va a funcionar. Pero si no hay ningún problema y
todas están *enabled* se puede
[usar lxc con relativa facilidad](http://www.stgraber.org/2012/05/04/lxc-in-ubuntu-12-04-lts/)
siempre que tengamos una distro como Ubuntu relativamente moderna:

	sudo lxc-create -t ubuntu -n una-caja
	
crea un contenedor denominado `una-caja` e instala Ubuntu en él. La
versión que instala dependerá de la que venga con la instalación de
`lxc`, generalmente una de larga duración y en mi caso la 14.04.4; además,
aparte de la instalación mínima, incluirá en el contenedor una serie
de utilidades como `vim` o `ssh`. Todo esto lo indicará en la consola
según se vaya instalando, lo que tardará un rato.

Alternativamente, se
puede usar una imagen similar a la que se usa en
[EC2 de Amazon](https://aws.amazon.com/es/ec2/), donde se denomina AMI:

	sudo lxc-create -t ubuntu-cloud -n nubecilla

que funciona de forma ligeramente diferente, porque se descarga un
fichero `.tar.gz` usando `wget` (y tarda también un rato). Las
imágenes disponibles las podemos consultar en
[esta web](https://us.images.linuxcontainers.org/), junto con los
nombres que tienen. La opción `-t` es para las plantillas existentes
instaladas en el sistema, que podemos consultar en el directorio
`/usr/share/lxc/templates`. 

>Entre ellas, varias de [Alpine Linux](https://alpinelinux.org/), una
>distribución ligera precisamente dirigida a su uso dentro de
>contenedores. También una llamada `plamo`, escrita en algún tipo de
>letra oriental, posiblemente japonés. Se puede instalar, pero igual
>no se entiende nada. 

Podemos
listar los contenedores que tenemos disponibles con `lxc-ls`, aunque
en este momento cualquier contenedor debería estar en estado
`STOPPED`.

Para arrancar el contenedor y conectarse a él, 

	sudo lxc-start -n nubecilla
	
, donde `-n` es la opción para dar el nombre del contenedor que se va
a iniciar. Dependiendo del contenedor que se arranque, habrá una configuración
inicial; en este caso, se configuran una serie de cosas y
eventualmente sale el login, que será para todas las máquinas creadas
de esta forma `ubuntu` (también clave). Lo que hace esta orden es
automatizar una serie de tareas tales como asignar los CGROUPS, crear
los namespaces que sean necesarios, y crear un puente de red tal como
hemos visto anteriormente. En general, creará un puente llamado
`lxcbr0` y otro con el prefijo `veth`.

Te puedes conectar al contenedor desde otro terminal usando
`lxc-console`

	sudo lxc-console -n nubecilla

La salida te dejará "pegado" al terminal. 

<div class='ejercicios' markdown='1'>

Instalar una distro tal como Alpine y conectarse a ella usando el
nombre de usuario y clave que indicará en su creación

</div>

Una vez arrancados los
contenedores, si se lista desde fuera aparecerá de esta forma:

~~~
$ sudo lxc-ls -f      
NAME       STATE    IPV4        IPV6  AUTOSTART  
-----------------------------------------------
nubecilla  RUNNING  10.0.3.171  -     NO         
una-caja   STOPPED  -           -     NO         
~~~

Y, dentro de la misma, tendremos una máquina virtual con esta
pinta:

![Dentro del contenedor LXC](../img/lxc.png)

Para el usuario del contenedor aparecerá exactamente igual que
cualquier otro ordenador: será una máquina virtual que, salvo error o
brecha de seguridad, no tendrá acceso al anfitrión, que sí podrá tener
acceso a los mismos y pararlos cuando le resulte conveniente. 

	sudo lxc-stop -n nubecilla
	
Las
[órdenes que incluye el paquete](https://help.ubuntu.com/lts/serverguide/lxc.html)
permiten administrar las máquinas virtuales, actualizarlas y explican
cómo usar otras plantillas de las suministradas para crear
contenedores con otro tipo de sistemas, sean o no debianitas. Se
pueden crear sistemas basados en Fedora; también clonar contenedores
existentes para que vaya todo rápidamente. 


>La
>[guía del usuario](https://help.ubuntu.com/lts/serverguide/lxc.html#lxc-startup)
>indica también cómo usarlo como usuario sin privilegios, lo que
>mayormente te ahorra la molestia de introducir sudo y en su caso la
>clave cada vez. Si lo vas a usar con cierta frecuencia, sobre todo en
>desarrollo, puede ser una mejor opción. 

Los contenedores son la implementación de una serie de tecnologías
[que tienen soporte en el sistema operativo: espacios de nombres, CGroups y puentes de red](Tecnicas_de_virtualizacion.md): y como
tales pueden ser configurados para usar sólo una cuota determinada
de recursos, por ejemplo
[la CPU](http://www.slideshare.net/dotCloud/scale11x-lxc-talk-16766275). Para
ello se usan los ficheros de configuración de cada una de las máquinas
virtuales. Sin embargo, tanto para controlar como para visualizar los
tápers (que así vamos a llamar a los contenedores a partir de ahora)
es más fácil usar [lxc-webpanel](http://lxc-webpanel.github.io/), un
centro de control por web que permite iniciar y parar las máquinas
virtuales, aparte de controlar los recursos asignados a cada una de
ellas y visualizarlos; la página principal te da una visión general de los contenedores
instalados y desde ella se pueden arrancar o parar. 

![Página inicial de LXC-Webpanel](../img/Overview-lxc.png)

Cada solución de virtualización tiene sus ventajas e
inconvenientes. La principal ventaja de este tipo de contenedores son el
aislamiento de recursos y la posibilidad de manejarlos, lo que hace
que se use de forma habitual en proveedores de infraestructuras
virtuales. El hecho de que se virtualicen los recursos también implica
que haya una diferencia en las prestaciones, que puede ser apreciable
en ciertas circunstancias.


Configurando las aplicaciones en un táper
----

Una vez creados los tápers, son en casi todos los aspectos como una
instalación normal de un sistema operativo: se puede instalar lo que
uno quiera. Sin embargo, una de las ventajas de la infraestructura
virtual es precisamente la (aparente) configuración del *hardware*
mediante *software*: de la misma forma que se crea, inicia y para
desde el anfitrión una MV, se puede configurar para que ejecute unos
servicios y programas determinados.

A este tipo de aplicaciones y sistemas se les denomina
[SCM por *software configuration management*](http://en.wikipedia.org/wiki/Software_configuration_management);
a pesar de ese nombre, se dedican principalmente a configurar
hardware, no software. Un sistema de este estilo permite, por ejemplo,
crear un táper (o, para el caso, una máquina virtual, o muchas de
ellas) y automáticamente *provisionarla* con el software necesario
para comportarse como un
[PaaS](http://jj.github.io/IV/documentos/temas/Intro_concepto_y_soporte_fisico#usando_un_servicio_paas)
o simplemente como una máquina de servicio al cliente. 

En general, un SCM permite crear métodos para instalar una aplicación
o servicio determinado, expresando sus dependencias, los servicios que
provee y cómo se puede trabajar con ellos. Por ejemplo, una base de
datos ofrece precisamente ese servicio; un sistema de gestión de
contenidos dependerá del lenguaje en el que esté escrito; además, se
pueden establecer *relaciones* entre ellos para que el CMS use la BD
para almacenar sus tablas. 

Hay
[decenas de sistemas CMS](http://en.wikipedia.org/wiki/Comparison_of_open-source_configuration_management_software),
aunque hoy en día los hay que tienen cierta popularidad, como Salt, Rex,
Ansible, Chef, Juju y Puppet. Todos ellos tienen sus ventajas e
inconvenientes, pero para la configuración de tápers se puede usar
directamente [Juju](http://juju.ubuntu.com), creado por Canonical
especialmente para máquinas virtuales de ubuntu que se ejecuten en la
nube de Amazon. En este punto nos interesa también porque se puede
usar directamente con contenedores LXC, mientras que no todos lo
hacen.

En el caso de `lxc`, una forma fácil de gestionar configuraciones es
hacerlo con Vagrant usando
[el driver para lxc](https://github.com/fgrehm/vagrant-lxc). Usando
alguna de las
[*cajas base* de  Atlas](https://github.com/obnoxxx/vagrant-lxc-base-boxes),
pero requiere cierta cantidad de trabajo para construir las plantillas
con Vagrant; eventualmente las cajas se tienen que introducir en el
directorio de plantillas de `lxc` para que se puedan usar, o bien usar
las de Atlas tales como esta

	vagrant init fgrehm/wheezy64-lxc

e iniciarlas con

	sudo vagrant up --provider=lxc

Con esto se puede provisionar o conectarse usando las herramientas habituales,
siempre que la imagen tenga soporte para ello. Por ejemplo, se puede
conectar uno a esta imagen de Debian con `vagrant ssh`  

<div class='ejercicios' markdown="1">

Provisionar un contenedor LXC usando Ansible o alguna otra herramienta
de configuración que ya se haya usado

</div>

Gestión de contenedores con `docker`
---

[Docker](http://docker.com) es una herramienta de gestión de
contenedores que permite no sólo instalarlos, sino trabajar con el
conjunto de ellos instalados (orquestación) y exportarlos de forma que
se puedan usar en diferentes instalaciones. La tecnología de
[Docker](https://en.wikipedia.org/wiki/Docker_%28software%29) es
relativamente reciente, habiendo sido publicado en marzo de 2013;
actualmente está sufriendo una gran expansión, sobre todo por su uso
dentro de [CoreOS](http://coreos.com/), un sistema operativo básico
basado en Linux para despliegue masivo de servidores.

[Instalar `docker` es fácil en las últimas versiones](https://www.docker.com/). Por
ejemplo, para 
[Ubuntu hay que dar de alta una serie de repositorios](https://docs.docker.com/engine/installation/linux/ubuntulinux/)
y no funcionará con versiones más antiguas de la 12.04 (y en este caso
sólo si se instalan kernels posteriores). En las últimas versiones, de
hecho, ya está en los repositorios y para instalarlo no hay más que
hacer

	sudo apt-get install docker.io

aunque la versión en los repositorios puede ser más antigua que la que
se descargue de la web. La instalación coloca también un servicio que
se ejecutará como *daemon* y se arrancará con el inicio del
sistema. La instalación desde docker.com siguiendo las instrucciones
te instalará también una serie de imágenes genéricas con las que se
puede empezar a trabajar de forma más o menos inmediata. Una vez
intalado, el clásico

	sudo docker run hello-world

`docker` permite instalar contenedores y trabajar con
ellos. Normalmente el ciclo de vida de un contenedor pasa por su
creación y, más adelante, ejecución de algún tipo de programa, por
ejemplo de instalación de los servicios que queramos; luego se puede
salvar el estado del táper y clonarlo o realizar cualquier otro tipo
de tareas. 

Así que comencemos desde el principio:
[vamos a ejecutar `docker`y trabajar con el contenedor creado](https://docs.docker.com/engine/installation/linux/ubuntulinux/).

Si no se está ejecutando ya, se puede arrancar como un servicio

	sudo docker -d &
	
La línea de órdenes de docker conectará con este daemon, que mantendrá
el estado de docker y demás. Cada una de las órdenes se ejecutará
también como superusuario, al tener que contactar con este *daemon*
usando un socket protegido.

>Estamos trabajando con docker como superusuario, que es la forma
>adecuada de
>hacerlo. [Puedes seguir estas instrucciones para hacerlo desde un usuario](https://docs.docker.com/engine/installation/linux/ubuntulinux/#manage-docker-as-a-non-root-user) sin privilegios de administración.

A partir de ahí, podemos crear un contenedor

	sudo docker pull alpine
	
Esta orden descarga un contenedor básico de [Alpine Linux](https://alpinelinux.org/) y lo instala. Hay
muchas imágenes creadas y se pueden crear y compartir en el sitio web
de Docker, al estilo de las librerías de Python o los paquetes
Debian. Se pueden
[buscar todas las imágenes de un tipo determinado, como Ubuntu](https://hub.docker.com/search/?isAutomated=0&isOfficial=0&page=1&pullCount=0&q=ubuntu&starCount=0)
o
[buscar las imágenes más populares](https://hub.docker.com/explore/). Estas
imágenes contienen no sólo sistemas operativos *bare bones*, sino
también otros con una funcionalidad determinada. 

<div class='ejercicios' markdown='1'>

Instalar una imagen alternativa de Ubuntu y alguna
adicional, por ejemplo de CentOS.

</div>

El contenedor tarda un poco en instalarse, mientras se baja la
imagen. Una vez bajada, se pueden empezar a ejecutar comandos. Lo
bueno de `docker` es que permite ejecutarlos directamente sin
necesidad de conectarse a la máquina; la gestión de la conexión y
demás lo hace ello, al modo de Vagrant (lo que veremos más adelante).

Podemos ejecutar, por ejemplo, un listado de los directorios

	sudo docker run alpine ls
	
Tras el sudo, hace falta docker; `run` es el comando de docker que
estamos usando, `ubuntu` es el nombre de la máquina, el mismo que le
hayamos dado antes cuando hemos hecho pull y finalmente `ls`, el
comando que estamos ejecutando. Este comando arranca el contenedor, lo
ejecuta y a continuación sale de él. Esta es una de las ventajas de
este tipo de virtualización: es tan rápido arrancar que se puede usar
para un simple comando y dejar de usarse a continuación. 

Esta imagen de Alpine no contiene bash, pero si el shell básico
llamado `ash` y que está instalado en `sh`,
por lo que podremos *meternos* en la misma ejecutando

	sudo docker run -it alpine sh

Dentro de ella podemos trabajar como un consola cualquiera, pero
teniendo acceso sólo a los recursos propios.

### Trabajando con Alpine Linux

Alpine es una instalación peculiar y más bien mínima, pero es muy
interesante para usarla como base para nuestros propios contenedores,
por su minimalismo. Conviene
[consultar el wiki](https://wiki.alpinelinux.org/wiki/Alpine_Linux_package_management)
para ver las tareas que se pueden realizar en ella.

Una de las primeras cosas que habrá que hacer es actualizar la
distribución. Alpine usa `apk` como gestor de paquetes, y la
instalación base no permite hacer gran cosa, así que para empezar
conviene hacer

	apk update
	apk upgrade

Para que actualice la lista de paquetes disponibles. Después, se pueden instalar paquetes, por ejemplo  

	apk add git
	apk add perl

Una vez añadido todo lo que queramos a la imagen, se puede almacenar o
subir al registro. En todo caso, `apk search` te permite buscar los
ficheros y paquetes que necesites para compilar o instalar algo. En
algunos casos puede ser un poco más complicado que para otras distros,
pero merece la pena. 


### Tareas adicionales con contenedores Docker

La máquina instalada la podemos usar con el nombre del SO, pero cada
táper tiene un id único que se puede ver con 

	sudo docker ps -a=false
	
siempre que se esté ejecutando, obteniendo algo así:

	CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
b76f70b6c5ce        ubuntu:12.04        /bin/bash           About an hour ago   Up About an hour                        sharp_brattain     

El primer número es el ID de la máquina que podemos usar también para
referirnos a ella en otros comandos. También se puede usar 
	
	sudo docker images
	
Que devolverá algo así:

	REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu              latest              8dbd9e392a96        9 months ago        128 MB
ubuntu              precise             8dbd9e392a96        9 months ago        128 MB
ubuntu              12.10               b750fe79269d        9 months ago        175.3 MB
ubuntu              quantal             b750fe79269d        9 months ago        175.3 MB

El *IMAGE ID* es el ID interno del contenedor, que se puede usar para
trabajar en una u otra máquina igual que antes hemos usado el nombre
de la imagen:

		sudo docker run b750fe79269d du
		
En vez de ejecutar las cosas una a una podemos directamente [ejecutar
un shell](https://docs.docker.com/engine/getstarted/step_two/):

	sudo docker run -i -t ubuntu /bin/bash

que [indica](https://docs.docker.com/engine/reference/commandline/cli/) que
se está creando un seudo-terminal (`-t`) y se está ejecutando el
comando interactivamente (`-i`). A partir de ahí sale la línea de
órdenes, con privilegios de superusuario, y podemos trabajar con la
máquina e instalar lo que se nos ocurra. Esto, claro está, si tenemos
ese contenedor instalado y ejecutándose. 

Los contenedores se pueden arrancar de forma independiente con `start`

	sudo docker start	ed747e1b64506ac40e585ba9412592b00719778fd1dc55dc9bc388bb22a943a8
	
pero hay que usar el ID largo que se obtiene dando la orden de esta
forma

	sudo docker images -notrunc

Para entrar en ese contenedor tienes que averiguar qué IP está usando
y los usuarios y claves y por supuesto tener ejecutándose un cliente
de `ssh` en la misma. Para averiguar la IP:

	sudo docker inspect	ed747e1b64506ac40e585ba9412592b00719778fd1dc55dc9bc388bb22a943a8
	
te dirá toda la información sobre la misma, incluyendo qué es lo que
está haciendo en un momento determinado. Para finalizar, se puede
parar usando `stop`. 

Hasta ahora el uso de docker [no es muy diferente del contenedor, pero
lo interesante](http://stackoverflow.com/questions/17989306/what-does-docker-add-to-just-plain-lxc) es que se puede guardar el estado de un contenedor tal
como está usando [commit](https://docs.docker.com/engine/reference/commandline/cli/#commit)

	sudo docker commit 8dbd9e392a964056420e5d58ca5cc376ef18e2de93b5cc90e868a1bbc8318c1c nuevo-nombre

que guardará el estado del contenedor tal como está en ese
momento. Este `commit` es equivalente al que se hace en un
repositorio; para enviarlo al repositorio habrá que usar `push` (pero
sólo si uno se ha dado de alta antes).

<div class='ejercicios' markdown='1'>

Crear a partir del contenedor anterior una imagen persistente con
*commit*. 

</div>

Finalmente, `docker` tiene capacidades de provisionamiento similares a
otros [sistemas (tales como Vagrant](Gestion_de_configuraciones.md) usando
[*Dockerfiles*](https://docs.docker.com/engine/reference/builder/). Por
ejemplo, [se
puede crear fácilmente un Dockerfile para instalar node.js con el
módulo express](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/). 


## Usando Dockerfiles

La infraestructura se debe crear usando código, y en Docker pasa
exactamente igual. Tiene un mecanismo llamado Dockerfiles que permite
construir contenedores o tápers de forma que lo que quede en control
de versiones sea el código en sí, no el contenedor, con el
consiguiente ahorro de espacio. La ventaja además es que
en [el Docker hub](http://hub.docker.com) hay multitud de contenedores
ya hechos, que se pueden usar directamente. Veamos un ejemplo, como es
habitual para el bot en Scala que hemos venido usando.

~~~
FROM frolvlad/alpine-scala
MAINTAINER JJ Merelo <jjmerelo@GMail.com>
WORKDIR /root
CMD ["/usr/local/bin/sbt"]

RUN apk update && apk upgrade
RUN apk add git
RUN apk add curl

RUN curl -sL "http://dl.bintray.com/sbt/native-packages/sbt/0.13.13/sbt-0.13.13.tgz" -o /usr/local/sbt.tgz
RUN cd /usr/local && tar xvfz sbt.tgz
RUN mv /usr/local/sbt-launcher-packaging-0.13.13/bin/sbt-launch.jar /usr/local/bin
COPY sbt /usr/local/bin
RUN chmod 0755 /usr/local/bin/sbt
RUN /usr/local/bin/sbt
~~~

En la primera línea se establece cuál es el contenedor de origen que
estamos usando. Siempre es conveniente usar distros ligeras, y en este
caso usamos la ya conocida Alpine, que tiene ya una versión que
incluye Scala. A continuación se pone la dirección del mantenedor,
servidor, y el directorio de trabajo `WORKDIR` en el que se va a
entrar cuando se ejecute algo en el contenedor. El siguiente comando
`CMD` indica qué se va a ejecutar en caso de que se ejecute el
contenedor directamente; se trata de `sbt`, el Scala Build Tool. Como
se ve, la estructura siempre es la misma: órdenes en mayúsculas, al
principio de la
línea. La
[referencia de las mismas se encuentra en la web de Docker](https://docs.docker.com/engine/reference/builder/#copy). 

Las siguientes órdenes son todas `apk`, el gestor de paquetes de
Alpine. No tiene tantos empaquetados como las distros más conocidas,
pero sí los básidos; siempre al principio habrá que actualizar los
repos para que no haya problemas.

El resto son otras órdenes `RUN`, que ejecutan directamente órdenes
dentro del contenedor, y que en este caso descargan un paquete, que es
la forma como se distribuye `sbt`, y lo ponen como ejecutable. Hay que
hacer una cosa adicional: copiar mediante `COPY` un fichero local,
`sbt`, para usarlo como base. 

Para crear una imagen a partir de esto se usa 
	
	sudo docker build -t jjmerelo/bobot .
	
(o el nick que tengas en GitHub). El `-t` es, como es habitual, para
asignar un *tag*, en este caso uno que se puede usar más adelante en
el Docker Hub. Tardará un rato, sobre todo por la descarga de unas
cuantes librerías por parte de sbt, lo que se hace en la última
línea. Una vez hecho esto, si funciona la construcción, se
podrá ejecutar con 

	sudo docker run -it jjmerelo/bot
	
o con

	sudo docker run -it jjmerelo/bot sh
	
para entrar directamente en la línea de órdenes. El repositorio está
en [bobot](https://github.com/JJ/BoBot), como es habitual. 
	
<div class='ejercicios' markdown='1'>

Reproducir los contenedores creados anteriormente usando un Dockerfile.

</div>	

	
A dónde ir desde aquí
-----

Primero, hay que [llevar a cabo el hito del proyecto correspondiente a este tema](../proyecto/4.Docker.md).

Si te interesa, puedes consultar cómo se [virtualiza el almacenamiento](Almacenamiento) que, en general, es independiente de la
generación de una máquina virtual. También puedes ir directamente al
[tema de uso de sistemas](Uso_de_sistemas.md) en el que se trabajará
con sistemas de virtualización completa. 
	
