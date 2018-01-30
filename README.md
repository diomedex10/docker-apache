# Entorno de desarrollo Web con Apache

## Descripción

Este entorno está pensado para poder tener en un docker varios proyectos, funcionando como virtualhosts, y autenticados bajo SSL. Es ideal para entornos de desarrollo o de integración continua, ya que permite, de forma fácil, tener múltiples webs.

El espíritu de este proyecto fue tener un entorno para probar antes del despliegue las builds de los proyectos de AngularJS.

## Instalación
Una vez descargados los archivos del repositorio, la creación de las imágenes e instanciación de los contenedores correspondientes se realizará mediante docker-compose en la carpeta que contiene el fichero docker-compose.yml:

```
docker-compose up -d
```

## Setup

Para configurar el docker, hace falta crear un archivo de entorno .env que incluya las siguientes variables:

* `httpd_htdocs`: Es la ruta al directorio que contendrá nuestros proyectos. Deben de ir situados en subcarpetas (ejemplo: htdocs/myproyect y htdocs/myproyect2).
* `httpd_vhosts`: Es la ruta a un directorio que contendrá los diversos archivos .conf de los distintos virtualhosts que montemos.
* `httpd_conf`: Aquí irá la ruta de el archivo de configuración de apache (**hhtpd.conf**) que usará nuestro container.

Y, para los certificados:

* `httpd_certs`: Directorio local que alberga los certificados. Por defecto es `./httpd/certs`.
* `httpd_cert_name`: Nombre del certificado. Por defecto es `server.crt`.
* `httpd_key_name`: Nombre del archivo con la clave usada para crear el certificado. El valor es `server.key`.
* `httpd_pem_name`: Nombre del archivo de claves Diffie Hellman usado para crear el certificado. El nombre por defecto es `server.pem`.

## Puesta en marcha de un proyecto

1. Copiar las aplicaciones de Angular en `./httpd/htdocs`
2. Copiar los virtualhosts de apache en `./httpd/vhosts_conf`. Recordemos que deben de acabar en *.conf para que apache los trate.
3. Añadir los hosts en nuestro archivo de hosts (ver más adelante para más información de cómo añadir nuevos hosts).

## Casos de uso

### Incluir nuevos virtualhosts en Apache

Para incluir nuevos hosts virtuales en nuestro entorno, debemos seguir los siguientes pasos:

1. Ubicar el proyecto que se pretende crear en `htdocs/*nombre_proyecto*`
2. Crear un archivo de configuración de apache para el nuevo virtualhosts y ubicarlo en `vhosts_conf\*nombre_proyecto*.conf`
3. Añadir el nuevo host en el archivo de hosts de nuestra máquina (/etc/hosts en linux e ios, c:\Windows\System32\Drivers\etc\hosts en windows).

```PROTIP: Se pueden usar wildcards: Ejemplo: *.app.dev, valdría para api.app.dev y para www.app.dev, y cualquiera que se quiera añadir a posteriori con ese dominio```

## Proxy
* *http_proxy*: Indica el path de conexión con el proxy HTTP. Tiene la forma: *http(s)://[<username>:<password>@]<host>[:<port>]*
* *https_proxy*: Contiene el path de conexión con el proxy HTTPS. Usa el mismo formato que el http_proxy.
* *no_proxy*: Contiene los hosts a los que se bypaseará para no conectarse por el proxy.

## Deploy de proyectos de Angular.

Para que angular haga el deploy automático de un proyecto dentro del directorio de apache, debemos cambiar una opción de configuración de `.angular-cli.json`, la cual es `outDir`, en la cual deberíamos poner el PATH (relativo o absoluto) del proyecto dentro de los htdocs.

Por ejemplo, si estuvieramos trabajando en el proyecto `angular-test` dentro de `.src/angular-test`, deberíamos dejar outDir como:

- `"outDir": "../../httpd/htdocs/angular-test",`

No olvidemos que también necesitaría su archivo de virtualHost de apache y dar de alta el host en el archivo de hosts de nuestro equipo.

## Conexión SSL (Apache).

Por defecto, este docker tiene SSL activado, y una clave de ejemplo. Para generar otra, podemos o bien usar una clave reconocida o generar un certificado autofirmado siguiendo la siguiente guía: https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-16-04

Una vez que tengamos los certificados generados, dejamos los archivos en `./httpd/certs` (recordemos que podemos cambiar este path mediante la variable de entorno de *httpd_certs*). Por defecto, los archivos de certificado serán `server.key`, `server.crt` y `server.pem`.

Si queremos desactivar la opción de HTTPS, sólo tendríamos que alterar el archivo `httpd.conf`, comentando la línea `Include conf/extra/httpd-ssl.conf`


## Archivo de virtualhosts

Para ejecutar un proyecto de angular, apache necesita un archivo de configuración que le redirija todo al index.html, dado que es una SPA.

A continuación podemos ver un ejemplo de un archivo:

```xml
<VirtualHost *:80>
    ServerName example.docker-example.dev

    DocumentRoot /usr/local/apache2/htdocs/docker-example

    <Directory /usr/local/apache2/htdocs/docker-example>
        RewriteEngine on

        # Don't rewrite files or directories
        RewriteCond %{REQUEST_FILENAME} -f [OR]
        RewriteCond %{REQUEST_FILENAME} -d
        RewriteRule ^ - [L]

        # Rewrite everything else to index.html
        # to allow html5 state links
        RewriteRule ^ index.html [L]
    </Directory>
</VirtualHost>

<VirtualHost *:443>
    ServerName example.docker-example.dev

    DocumentRoot /usr/local/apache2/htdocs/docker-example

    <Directory /usr/local/apache2/htdocs/docker-example>
        RewriteEngine on

        # Don't rewrite files or directories
        RewriteCond %{REQUEST_FILENAME} -f [OR]
        RewriteCond %{REQUEST_FILENAME} -d
        RewriteRule ^ - [L]

        # Rewrite everything else to index.html
        # to allow html5 state links
        RewriteRule ^ index.html [L]
    </Directory>
</VirtualHost>

```

De este archivo, habría que personalizar en cada proyecto el `ServerName`, el `DocumentRoot` y el `Directory`.
