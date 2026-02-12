# Documentación de la práctica - Escenario 1 

### Nombre del alumno: Cristina del Águila Martín

## Entorno de desarrollo y de producción utilizado

### Entorno de desarrollo:
- **Sistema Operativo**: macOS Sonoma 14.2
- **Versión de Docker**: 27.3.1
- **Gestor de contenedores**: Docker

## Descripción de la práctica
### 2.1. Objetivo
Se plantea la implementación de una solución cloud para una pequeña empresa o departamento con un grupo reducido de usuarios. La solución busca centralizar almacenamiento, acceso web, autenticación y gestión de archivos con alta disponibilidad dentro de las limitaciones de infraestructura.  

Para esto vamos a desplegar un entorno basado en ownCloud para almacenamiento y acceso web, gestionado mediante una base de datos MariaDB y optimizado con Redis. La autenticación de usuarios se lleva a cabo a través de un servicio LDAP.  

- **Servidor web OwnCloud**.
- **Servicio de cache Redis**.
- **Autenticación LDAP** 

Siguiendo la siguiente estructura:

![cap24](/img/c24.png)

## Despliegue y gestión de servicios de autenticación de usuarios con LDAP

Creo el archivo `docker-compose.yml`, donde se definen todos los servicios relacionados con OpenLDAP. 

![cap1](/img/c1.png)

Me voy al directorio donde está docker-compose.yml y lo ejecuto

    $ docker-compose up -d

![cap2](/img/c2.png)

Compruebo que está activo

![cap3](/img/c3.png)

A continuación, he copiado los archivos .ldif que contienen las definiciones de usuarios, grupos y unidades organizativas dentro del contenedor LDAP. Estos archivos incluyen las entradas necesarias para configurar usuarios como "cristina", "isabel", "pedro" y "ana", además de grupos como "Editores" y "Lectores". Después de copiarlos, he usado el comando ldapadd desde dentro del contenedor para cargar esta información en el servidor LDAP.

![cap4](/img/c4.png)

Conecto al contenedor usando docker exec

![cap5](/img/c5.png)

Una vez dentro del contenedor, uso el comando ldapsearch para verificar si las entradas ya están en el servidor LDAP. Para esto uso los comandos
 
    $ ldapsearch -x -H ldap://localhost -b "uid=cristina,dc=example,dc=org" -D "cn=admin,dc=example,dc=org" -w admin
    $ ldapsearch -x -H ldap://localhost -b "uid=isabel,dc=example,dc=org" -D "cn=admin,dc=example,dc=org" -w admin 

![cap6](/img/c6.png)

Pruebo a tirar el contenedor y ejecutarlo de nuevo y compruebo que hay persistencia de datos, deteniendo y eliminando el contenedor, para después volver a levantarlo

    $ docker stop dd5565c26869
    $ docker rm dd5565c26869
    $ docker-compose up -d 
    $ docker exec -it openldap-server bash

Como se puede observar en la siguiente imagen, las entradas siguen presentes, lo que confirma que el sistema guarda la información en volúmenes persistentes.

![cap7](/img/c7.png)

### Archivos ldif creados
Los archivos que he creado son los siguientes:

    grupo_editores.ldif
    grupo_lectores.ldif
    pedro.ldif
    ana.ldif
    cristina.ldif
    isabel.ldif
    ou_groups.ldif
    ou_people.ldif

La estructura es 

    dc=example,dc=org
    ├── ou=People
    │   ├── uid=cristina
    │   ├── uid=isabel
    │   ├── uid=pedro
    │   ├── uid=ana
    │
    ├── ou=Groups
        ├── cn=Editores (cristina, isabel)
        ├── cn=Lectores (pedro, ana)

Para copiarlos uso los comandos:

    $ docker cp grupo_editores.ldif openldap-server:/tmp/grupo_editores.ldif
    $ docker cp grupo_lectores.ldif openldap-server:/tmp/grupo_lectores.ldif
    $ docker cp pedro.ldif openldap-server:/tmp/pedro.ldif
    $ docker cp ana.ldif openldap-server:/tmp/ana.ldif
    $ docker cp cristina.ldif openldap-server:/tmp/cristina.ldif
    $ docker cp isabel.ldif openldap-server:/tmp/isabel.ldif
    $ docker cp ou_groups.ldif openldap-server:/tmp/ou_groups.ldif
    $ docker cp ou_people.ldif openldap-server:/tmp/ou_people.ldif

![cap17](/img/c17.png)

Y después los añado:

    $ ldapadd -x -D "cn=admin,dc=example,dc=org" -w admin -H ldap://localhost -f tmp/grupo_editores.ldif
    $ ldapadd -x -D "cn=admin,dc=example,dc=org" -w admin -H ldap://localhost -f tmp/grupo_lectores.ldif
    $ ldapadd -x -D "cn=admin,dc=example,dc=org" -w admin -H ldap://localhost -f tmp/pedro.ldif
    $ ldapadd -x -D "cn=admin,dc=example,dc=org" -w admin -H ldap://localhost -f tmp/ana.ldif
    $ ldapadd -x -D "cn=admin,dc=example,dc=org" -w admin -H ldap://localhost -f tmp/cristina.ldif
    $ ldapadd -x -D "cn=admin,dc=example,dc=org" -w admin -H ldap://localhost -f tmp/isabel.ldif
    $ ldapadd -x -D "cn=admin,dc=example,dc=org" -w admin -H ldap://localhost -f tmp/ou_groups.ldif
    $ ldapadd -x -D "cn=admin,dc=example,dc=org" -w admin -H ldap://localhost -f tmp/ou_people.ldif

Uso el siguiente comando para comprobar que todo esta metido correctamente 

    $ ldapsearch -x -LLL -D "cn=admin,dc=example,dc=org" -W -b "dc=example,dc=org"

### Cambiar el Password de un usuario en LDAP
En mi caso, voy a cambiar la contraseña de todos los usuarios

    $ ldappasswd -s cris -w admin -D "cn=admin,dc=example,dc=org" -x "uid=cristina,dc=example,dc=org"

### Modificar atributos de un usuario
a) Crear un archivo LDIF (cristina_modify.ldif) con lo siguiente:
    
    dn: uid=cristina,dc=example,dc=org
    changetype: modify
    replace: loginShell
    loginShell: /bin/csh

    $ docker cp cristina_modify.ldif openldap-server:/tmp/cristina_modify.ldif
    $ docker exec -it openldap-server bash
    $ ldapmodify -x -D "cn=admin,dc=example,dc=org" -w admin -H ldap://localhost -f tmp/cristina_modify.ldif

Después de ejecutar el comando, verifico que los cambios se hayan aplicado correctamente usando ldapsearch, y efectivamente se ha modificado el loginShell

    $ ldapsearch -x -H ldap://localhost -b "uid=cristina,dc=example,dc=org" -D "cn=admin,dc=example,dc=org" -w admin

![cap8](/img/c8.png)

### Añadir un atributo a un usuario
Voy a añadir una descripcion al usuario cristina

a) Primero creo el archivo cristina_add_descrip.ldif con el siguiente contenido

    dn: uid=cristina,dc=example,dc=org
    changetype: modify
    add: description
    description: Cristina del Aguila

b) Copio el archivo al directorio y lo ejecuto con ldapmodify

    $ docker cp cristina_add_descrip.ldif openldap-server:/tmp/cristina_add_descrip.ldif
    $ docker exec -it openldap-server bash
    $ ldapmodify -x -D "cn=admin,dc=example,dc=org" -w admin -H ldap://localhost -f tmp/cristina_add_descrip.ldif

c) Verifico los cambios con ldapsearch

    $ ldapsearch -x -H ldap://localhost -b "uid=cristina,dc=example,dc=org" -D "cn=admin,dc=example,dc=org" -w admin

![cap9](/img/c9.png)

### Eliminar un atributo de un usuario
En mi caso no quiero eliminar ningún atributo de ningún usuario, pero si lo quisiera hacer el proceso sería similar a los anteriores pero usando en el archivo .ldif 

    delete: <lo que se quiere eliminar>

Por ejemplo, si quisiera eliminar la descripcion que he añadido en el apartado anterior, los pasos serian los siguientes:

1) Crear el archivo cristina_del_descr.ldif con el contenido

    dn: uid=cristina,dc=example,dc=org
    changetype: modify
    delete: description

2) Copiar el archivo al directorio y ejecutar ldapmodify 

    $ docker cp cristina_del_descr.ldif openldap-server:/tmp/cristina_del_descr.ldif
    $ docker exec -it openldap-server bash
    $ ldapmodify -x -D "cn=admin,dc=example,dc=org" -w admin -H ldap://localhost -f tmp/cristina_del_descr.ldif

3) Verificar los cambios con ldapsearch

    $ ldapsearch -x -H ldap://localhost -b "uid=cristina,dc=example,dc=org" -D "cn=admin,dc=example,dc=org" -w admin

### Añadir una OU (Unidad Organizativa)
Voy a añadir la OU People

1) Creo el archivo add_new_ou.ldif con el contenido:

    dn: ou=People,dc=example,dc=org
    ou: People
    objectClass: top
    objectClass: organizationalUnit
    description: Parent object of all PEOPLE accounts

2) Copiar el archivo al directorio y ejecutar ldapadd 

    $ docker cp add_new_ou.ldif openldap-server:/tmp/add_new_ou.ldif
    $ docker exec -it openldap-server bash
    $ ldapadd -x -D "cn=admin,dc=example,dc=org" -w admin -H ldap://localhost -f tmp/add_new_ou.ldif

3) Verificar los cambios con ldapsearch

    $ ldapsearch -x -H ldap://localhost -b "ou=People,dc=example,dc=org" -D "cn=admin,dc=example,dc=org" -w admin
    
![cap10](/img/c10.png)

### Buscar entradas en el DIT
Para buscar entradas en el directorio LDAP, uso el comando ldapsearch. Por ejemplo:

a) Buscar todos los objetos bajo ou=People:

    $ ldapsearch -x -H ldap://localhost -b "ou=People,dc=example,dc=org" -D "cn=admin,dc=example,dc=org" -w admin

b) Buscar un usuario específico:

    $ ldapsearch -x -H ldap://localhost -b "uid=cristina,dc=example,dc=org" -D "cn=admin,dc=example,dc=org" -w admin

## Despliegue de ownCloud

Para el despliegue de OwnCloud, ejecuto el contenedor exponiendo el puerto 20662 para poder acceder vía web. El entorno queda accesible en `http://localhost:20662`, desde donde he hecho las configuraciones iniciales y la integración con LDAP.

Los pasos han sido los siguientes: 

a) Descargar la imagen de ownCloud:

    $ docker pull owncloud

b) Ejecutar el contenedor de ownCloud:

    $ docker run -d -p 20662:80 --name owncloud owncloud:latest

## Despliegue de MariaDB
Para la base de datos, descarguo la imagen de MariaDB y creo un volumen local llamado `MariaDB_data` para asegurar la persistencia de la información. El contenedor lo ejecuto con variables de entorno que establecen el nombre de la base de datos, el usuario, la contraseña, y la contraseña de root. 

Los pasos han sido los siguientes: 

a) Descargar la imagen de MariaDB:

    $ docker pull mariadb

b) Creao un directorio para la persistencia de datos:

    $ mkdir -p MariaDB_data

c) Ejecuto el contenedor de MariaDB:

    $ docker run -d --name mariadb \
    -v MariaDB_data:/var/lib/mysql \
    -e MARIADB_DATABASE=owncloud \
    -e MARIADB_USER=owncloud \
    -e MARIADB_PASSWORD=owncloudpassword \
    -e MARIADB_ROOT_PASSWORD=rootpassword \
    mariadb:latest

## Despliegue de Redis
Redis lo despliego descargando la imagen oficial y ejecutando un contenedor que proporciona servicios de caché. Este componente es fundamental para mejorar el rendimiento del sistema, especialmente en tareas de consulta y respuesta rápida dentro de OwnCloud.

Los pasos han sido los siguientes: 

a) Descargo la imagen de Redis:

    $ docker pull redis

b) Ejecutar el contenedor de Redis:

    $ docker run -d --name redis redis:latest

## Configuración de ownCloud para usar MariaDB y Redis

a) Accedo a la interfaz web de ownCloud:

    http://localhost:20662

![cap11](/img/c11.png)

Sigo el tutorial que se ha proporcionado https://www.youtube.com/watch?v=Jd0JImHj3fk

Me identifico como 

    Nombre de usuario: admin
    Contraseña: admin

Nos vamos a "Market"

![cap12](/img/c12.png)

Busco "LDAP Integration"

![cap13](/img/c13.png)

Lo instalamos

![cap14](/img/c14.png)

Una vez que se ha instalado, nos vamos a la pestaña superior derecha y hacemos click en Perfil -> Ajustes (en mi caso admin -> Ajustes). Podemos observar que hay una nueva entrada llamada "Autentificacion de Usuario"

![cap15](/img/c15.png)

Para rellenar los datos en este apartado, he tenido el problema descrito en problema1.

Una vez resuelto, obtengo esto 
![cap16](/img/c16.png)

Quiero que solamente los usuarios del grupo "Editores" (Es decir, cristina e isabel) tengan acceso a ownCloud, para eso hago lo siguiente 

![cap18](/img/c18.png)

A continuación compruebo que efectivamente solamente esos usuarios tienen acceso.

![cap19](/img/c19.png)
![cap20](/img/c20.png)

Lo compruebo haciendo login desde una ventana de incognito para ver si funciona correctamente. Primero voy a cambiar la contraseña de los usuarios para poder comprobar si puedo iniciar sesion

    $ ldappasswd -s isabelpasswd -w admin -D "cn=admin,dc=example,dc=org" -x "uid=isabel,ou=People,dc=example,dc=org"
    $ ldappasswd -s cristinapasswd -w admin -D "cn=admin,dc=example,dc=org" -x "uid=cristina,ou=People,dc=example,dc=org"
    $ ldappasswd -s pedropasswd -w admin -D "cn=admin,dc=example,dc=org" -x "uid=pedro,ou=People,dc=example,dc=org"
    $ ldappasswd -s anapasswd -w admin -D "cn=admin,dc=example,dc=org" -x "uid=ana,ou=People,dc=example,dc=org"

Como podemos observar a continuación el usuario "isabel" si tiene acceso, pero "pedro" no lo tiene

![cap21](/img/c21.png)
![cap22](/img/c22.png)

Además si desde admin, nos vamos al apartado "Usuarios" (esquina superior derecha), podemos ver los usuarios que tenemos

![cap23](/img/c23.png)

## Problemas encontrados
Al intentar configurar el servidor LDAP en OwnCloud, aparecía el error "Lost connection to LDAP server".
El problema se debía a que los contenedores de OwnCloud y OpenLDAP no estaban en la misma red de Docker, impidiendo la comunicación entre ellos. Para solucionarlo he hecho lo siguiente:

1) Verificar redes disponibles en Docker
He ejecutado el siguiente comando para listar las redes existentes

        $ docker network ls

Esto muestra varias redes, incluyendo p1_default, donde se encontraba OpenLDAP.

2) Verificar a qué red pertenece cada contenedor
Para comprobarlo, he usado:

        $ docker inspect owncloud | grep Network
        $ docker inspect openldap-server | grep Network

El resultado ha indicado que:
- OwnCloud estaba en la red bridge.
- OpenLDAP estaba en la red p1_default.

Lo cual confirma que no estaban en la misma red, lo que impedía la comunicación.

3) Solución

Conectar OwnCloud a la red p1_default ejecutando el comando:

    $ docker network connect p1_default owncloud

Comprobamos que se ha solucionado:

    $ docker inspect owncloud | grep Network
    $ docker inspect openldap-server | grep Network

Reinicio OwnCloud para aplicar cambios

    $ docker restart owncloud

## Referencias
- Documentación oficial de Docker: https://docs.docker.com/
- Documentación oficial de OpenLDAP: https://www.openldap.org/doc/
- Tutorial de configuración LDAP en OwnCloud (YouTube): https://www.youtube.com/watch?v=Jd0JImHj3fk
- Comandos de referencia para LDAP: https://linux.die.net/man/1/ldapsearch