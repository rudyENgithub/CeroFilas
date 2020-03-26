# [Cero Filas](https://cerofilas.veracruzmunicipio.gob.mx)

Proyecto de generación, confirmación y evaluación de citas programadas y turnos para la atención de trámites realizados por el Gobierno de la Ciudad de Veracruz.

  - [Acerca de Cero Filas](#acerca-de-cero-filas)
  - [Base de datos](#base-de-datos)
    - [Tabla de citas](#tabla-de-citas)
	- [Tabla de turnos](#tabla-de-turnos)
	- [Tabla de usuarios](#tabla-de-usuarios)
	- [Tabla de trámites](#tabla-de-tramites)
  - [Archivos necesarios para el correcto funcionamiento del sitio](#archivos-necesarios-para-el-correcto-funcionamiento-del-sitio)
    - [Proyecto Laravel](#proyecto-laravel)
    - [Archivos publicos](#archivos-publicos)
  - [Instrucciones para desarrolladores](#instrucciones-para-desarrolladores)
    - [Adaptación estetica](#adaptaci%C3%B3n-estetica)
  - [Levantar el proyecto en un ambiente productivo](#levantar-el-proyecto-en-un-ambiente-productivo)
    - [Requerimientos](#requerimientos)
    - [Configuración](#configuraci%C3%B3n)
    - [Instalación por primera vez](#instalaci%C3%B3n-por-primera-vez)
    - [Actualizaciones](#actualizaciones)

## Acerca de Cero Filas 

Cero Filas nace por iniciativa del H. Ayuntamiento de Veracruz en la administración 2018 - 2021 encabezada por el Maestro Fernando Yunes Márquez, Presidente Municipal Constitucional, el objetivo de dicha iniciativa es que a través de herramientas digitales, se le dé la oportunidad al ciudadano de escoger fecha y hora para asistir a la realización de un trámite y con ello:

* Evitar las largas filas para la atención a la realización de trámites
* Disminuir el tiempo de espera del ciudadano para ser atendido
* Eficientar el proceso del tramitador para atender a un ciudadano

La solicitud de una cita se lleva acabo a través de la plataforma Cero Filas, ingresando a [cerofilas.veracruzmunicipio.gob.mx](https://cerofilas.veracruzmunicipio.gob.mx)

El tramitador tendrá acceso a la plataforma ingresando a [cerofilas.veracruzmunicipio.gob.mx/sistema](https://cerofilas.veracruzmunicipio.gob.mx/sistema), es aquí donde podrá visualizar las citas y turnos pasados, citas y turnos actuales y los futuros. Dentro de esta plataforma, el tramitador podrá crear un historial del ciudadano o acceder a él a través de la CURP.

Para mejorar la atención del tramitador al ciudadano, se le da al tramitante la herramienta de la evaluación, esta se realizará una vez que haya concluido la atención, y llegará mediante correo electrónico, aquí podrá calificar la atención y dejar comentarios acerca de la atención recibida y su experiencia al realizar el trámite.

De la misma manera, se cuenta con un administrador de oficina (ingresando al mismo sistema) que podrá gestionar los catálogos de trámites y los tramitadores que desempeñarán ese trámite, así como sus horarios de atención de cada uno de ellos. Podrán gestionar las ausencias de los tramitadores para planificar de mejor manera la atención al ciudadano.

Se trata de una aplicación backend y frontend, construido con el framework Laravel 5.5 (PHP 7.1) usando como gestor de base de datos MySQL (5.0+). 
Tanto vistas, modelo y controlador se usaron a plenitud con el framework Laravel. 
Contiene formularios convencionales e invocaciones por ajax a backend con información que tiene acceso a base de datos.

## Base de datos

Se hace uso de una base de datos para almacenar la información de las citas y los turnos en las entidades: turnos, citas. Para llegar a estas entidades, se necesitan tener catálogos de: usuarios, trámites, oficinas, dependencias. Y algunas entidades de relación adicionales: configuraciones, holdingcitas, tramitesxoficinas, tramitesxusers, valoraciones, ausencias.

A continuación se explica cada una de estas entidades.

### Tabla de citas

Tabla llamada `citas`. Esta entidad almacena las citas programadas por los visitantes del portal. Hace uso de las entidades catálogo como [tramites](#tabla-de-tramites), [oficinas](#tabla-de-oficinas).

| Nombre de la columna          | Columna obligatoria / opcional | Tipo de dato      | Detalle                                              |
| ----------------------------- |------------------------------- | ----------------- | ---------------------------------------------------- |
| `id_cita`                     | Obligatoria                    | Numerico¹         | Identificador autonumerico de la cita                |
| `tramite_id`                  | Obligatoria                    | Numerico¹         | Idenfificador de trámite de la tabla "tramites"      |
| `oficina_id`                  | Obligatoria                    | Numerico¹         | Idenfificador de oficina de la tabla "oficinas"      |
| `fechahora`                   | Obligatoria                    | Fechahora         | Fecha y hora de la cita                              |
| `nombre_ciudadano`            | Obligatoria                    | Texto             | Nombre(s) del ciudadadno que registra la cita        |
| `appaterno_ciudadano`         | Obligatoria                    | Texto             | Apellido paterno del ciudadano                       |
| `apmaterno_ciudadano`         | Obligatoria                    | Texto             | Apellido materno del ciudadano                       |
| `email`                       | Opcional                       | Texto             | Email del ciudadano                                  |
| `curp`                        | Obligatoria                    | Texto             | CURP del ciudadano                                   |
| `telefono`                    | Opcional³                      | Texto             | Teléfono del ciudadano                               |
| `folio`                       | Opcional                       | Texto             | Folio único de la cita de 8 caracteres               |
| `ip`                          | Opcional                       | Texto             | IP de la computadora que registra la cita            |
| `statuscita`                  | Opcional                       | Texto²            | Estatus de la cita (null or cancelada)               |
| `created_by`                  | Opcional                       | Numerico¹         | Identificador de usuario creador del registro        |
| `created_at`                  | Opcional                       | Marca de tiempo   | Marca de tiempo de creación del registro             |
| `updated_by`                  | Opcional                       | Numerico¹         | Identificador de usuario modificador del registro    |
| `updated_at`                  | Opcional                       | Marca de tiempo   | Marca de tiempo de modificación del registro         |

¹ Tener en cuenta que es importante formatear los numeros de forma tal que el separador decimal sea `.` y no deben 
contar con separadores de miles, comillas, caracter de moneda, ni otros caracteres especiales. 

² Los estatus de la cita son: `cancelada`, null. Cuando una cita es creada, no tiene un estatus, solo es modificada al ser cancelada.

### Tabla de turnos

Tabla llamada `turnos`. Esta entidad almacena los turnos (check-in de ciudadanos), es cuando un ciudadano con cita o sin cita se presenta en las oficinas y marca su presencia, si trae cita, entonces trae consigo un folio de cita o QR. Hace uso de las entidades catálogo como [tramites](#tabla-de-tramites), [oficinas](#tabla-de-oficinas), [citas](#tabla-de-citas), [users](#tabla-de-usuarios).

| Nombre de la columna          | Columna obligatoria / opcional | Tipo de dato      | Detalle                                              |
| ----------------------------- |------------------------------- | ----------------- | ---------------------------------------------------- |
| `id_turno`                    | Obligatoria                    | Numerico¹         | Identificador autonumerico del turno                 |
| `cita_id`                     | Opcional                       | Numerico¹         | Idenfificador de cita de la tabla "citas"            |
| `oficina_id`                  | Obligatoria                    | Numerico¹         | Idenfificador de oficina de la tabla "oficinas"      |
| `user_id`                     | Opcional                       | Numerico¹         | Idenfificador de tramitador de la tabla "users"      |
| `tramite_id`                  | Obligatoria                    | Numerico¹         | Idenfificador de trámite de la tabla "tramites"      |
| `fechahora_inicio`            | Opcional                       | Fechahora         | Fecha y hora de inicio de atención del turno         |
| `fechahora_fin`               | Opcional                       | Fechahora         | Fecha y hora de fin de atención del turno            |
| `observaciones`               | Opcional                       | TextoLargo        | Observaciones registradas por el tramitador sobre la atención del turno |
| `nombre_ciudadano`            | Opcional                       | Texto             | Nombre(s) del ciudadadno que registra el turno       |
| `curp`                        | Obligatoria                    | Texto             | CURP del ciudadano                                   |
| `email`                       | Opcional                       | Texto             | Email del ciudadano                                  |
| `estatus`                     | Obligatoria                    | Enum²             | Estatus del turno                                    |
| `folio`                       | Obligatoria                    | Texto             | Folio único del turno de 8 caracteres                |
| `tiempoaproxmin`              | Obligatoria                    | Numerico¹         | Tiempo aproximado en minutos en que será atendido el turno |
| `created_by`                  | Opcional                       | Numerico¹         | Identificador de usuario creador del registro        |
| `created_at`                  | Opcional                       | Marca de tiempo   | Marca de tiempo de creación del registro             |
| `updated_by`                  | Opcional                       | Numerico¹         | Identificador de usuario modificador del registro    |
| `updated_at`                  | Opcional                       | Marca de tiempo   | Marca de tiempo de modificación del registro         |


¹ Tener en cuenta que es importante formatear los numeros de forma tal que el separador decimal sea `.` y no deben 
contar con separadores de miles, comillas, caracter de moneda, ni otros caracteres especiales. 

² Los estatus del turno son: `creado`, `enproceso`, `finalizado`, `cancelado`. Estos estatus van cambiando conforme acciones del sistema.

### Tabla de usuarios

Tabla llamada `users`. Esta entidad almacena los usuarios del sistema (tramitadores, administrador de oficina, kiosko, turnera y super administrador). Cada tipo de usuario cuenta con diferentes acciones dentro del sistema. Hace uso de las entidades catálogo como [oficinas](#tabla-de-oficinas).

| Nombre de la columna          | Columna obligatoria / opcional | Tipo de dato      | Detalle                                              |
| ----------------------------- |------------------------------- | ----------------- | ---------------------------------------------------- |
| `id_user`                     | Obligatoria                    | Numerico¹         | Identificador autonumerico del usuarios              |
| `tipo_user`                   | Obligatoria                    | Enum²             | Tipo de usuario del sistema                          |
| `estatus`                     | Obligatoria                    | Enum³             | Estatus del usuario                                  |
| `email`                       | Opcional                       | Texto             | Email del usuario con el que hara login en el sistema |
| `password`                    | Opcional                       | Texto             | Password del usuario con el que hara login en el sistema |
| `nombre`                      | Obligatoria                    | Texto             | Nombre(s) del usuario                                |
| `oficina_id`                  | Obligatoria                    | Numerico¹         | Idenfificador de oficina de la tabla "oficinas"      |
| `disponible_turno`            | Opcional                       | Enum⁴             | Indica si el usuario esta disponible para atender turnos |
| `ventanilla`                  | Opcional                       | Texto             | Clave alfanumerica para indicar la ventanilla donde atiende el usuario los turnos |
| `REMEMBER_TOKEN`              | Opcional                       | Texto¹            | Token de sesión del usuario                          |
| `created_by`                  | Opcional                       | Numerico¹         | Identificador de usuario creador del registro        |
| `created_at`                  | Opcional                       | Marca de tiempo   | Marca de tiempo de creación del registro             |
| `updated_by`                  | Opcional                       | Numerico¹         | Identificador de usuario modificador del registro    |
| `updated_at`                  | Opcional                       | Marca de tiempo   | Marca de tiempo de modificación del registro         |


¹ Tener en cuenta que es importante formatear los numeros de forma tal que el separador decimal sea `.` y no deben 
contar con separadores de miles, comillas, caracter de moneda, ni otros caracteres especiales. 

² Los tipos de usuario del sistema son: `superadmin`, `admin_oficina`, `kiosko`, `tramitador`, `turnera`. El `superadmin` permite administrar todos los usuarios del sistema, el `adminoficina` administra los usuarios y tramites de su jurisdiccion,  `kiosko` permite recibir a los ciudadanos por turno o cita, `turnera` nos permite ver en pantalla los turnos y su asignación a ventanilla de atención.

³ Los usuarios pueden estar activos o inactivos, un usuario inactivo no puede iniciar sesión en el sistema, ni es tomado en cuenta para los calculos de los horarios disponibles de una oficina.

⁴ Un usuario puede (`si`) o no (`no`) estar disponible para atender un turno. 

## Archivos necesarios para el correcto funcionamiento del sitio

### Proyecto Laravel

Dentro de este folder de git ...

### Archivos publicos

Dentro de este folder de git ...

## Instrucciones para desarrolladores

Para realizar modificaciones al sitio actual, realizar los siguientes pasos

* Clonar el proyecto usando git
* Instalar NodeJS. Recomendamos usar [nvm](https://github.com/creationix/nvm). Una vez instalado `nvm` ejecutar 
`nvm use` en la carpeta raiz del proyecto para asegurarse de usar la misma version de node.
* Instalar las dependencias de npm via `npm install` en la carpeta raiz del proyecto 
* Instalar las dependencias de bower via `bower install` en la carpeta raiz del proyecto
* Archivo de configuración: En /app duplicar el archivo config.js.example con el nombre config.js y configurar 
acorde a la [seccion correspondiente al archivo de configuración](#archivo-de-configuraci%C3%B3n)
* En caso de que el archivo de datos del sitio sea parte del sitio, ubicarlo dentro de la carpeta `app`
* Levantar el servidor de desarrollo via `grunt serve` en la carpeta raiz del proyecto
* Hacer los cambios en /app y con live reloading se actualizará en http://localhost:10000
* Una vez que se cuenta con los cambios deseados, compilar assets via `grunt build` en la carpeta raiz del proyecto

### Adaptación estetica

En caso de necesitar agregar cambios al css del sitio, ubicarlos en el archivo `app/styles/custom.css`. Este archivo
tiene precedencia por sobre los estilos base del sitio. 

Una vez modificado este archivo, ejecutar el comando `grunt build` para empaquetar los cambios y disponibilizarlos
en la carpeta `dist` con el resto de los assets productivos.


## Levantar el proyecto en un ambiente productivo

### Requerimientos

Sólo requiere un servidor web que pueda servir los contenidos. 
La aplicación contiene su código y las librerías que necesita, ya compiladas en la carpeta `/dist`.

* Apache
* No se requiere hacer build de ningún tipo.
* No se requiere salida a internet desde el servidor.
* No se requieren instalación de dependencias.
* Todo lo necesario está compilado y minificado en estáticos (html, css, js e imágenes), dentro de la carpeta `/dist`.

### Configuración

* La única configuración requerida es la creación del ARCHIVO de configuración y al archivo `csv` de datos 
(en caso de no cargarlo via un servicio externo)
* No requiere parámetros de ambiente del servidor, ni de proceso. Es todo web y js client – side.

### Instalación por primera vez

1. Crear un subdominio o definir la url donde vivirá la aplicación, podría ser: 
`https://obras-site.buenosaires.gob.ar/`.
2. Definir un servidor con nginx o apache y clonar el proyecto usando 
`git clone https://github.com/datosgcba/ba_obras.git`.
3. Crear el [archivo de configuración](#archivo-de-configuraci%C3%B3n)
4. En caso de que el archivo de datos del sitio sea parte del sitio, ubicarlo dentro de la carpeta `dist`
5. Apuntar las configuraciones del web server y subdominio a la carpeta `dist`, donde se encuentran los archivos 
finales y compilados.

### Actualizaciones

En caso de querer actualizar a un tag especifico, ubicarse en la carpeta raíz del proyecto, dentro del servidor y 
ejecutar el pull al tag de la versión indicada via `git fetch --tags` y luego `git checkout TAG-DE-VERSION`.

En caso de querer actualizar a la ultima version disponible, ubicarse en la carpeta raiz del proyecto y ejecutar
`git pull`.

En ambos casos, la carpeta `dist` quedara con sus archivos actualizados. Si se han realizados cambios al proyecto
utilizando versionado via git, mergearlos a la correspondiente nueva version.
