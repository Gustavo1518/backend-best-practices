Backend development best practices
==================================


<!-- START doctoc generado TOC por favor mantenga el comentario aquí para permitir la actualización automática ->
<!-- NO EDITE ESTA SECCIÓN, EN LUGAR RE-RUN doctoc PARA ACTUALIZAR -->

**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Traducciones de este documento](#traducciones-de-este-documento)
- [N Mandamientos](#n-mandamientos)
- [Puntos generales sobre las directrices](#puntos-generales-sobre-las-directrices)
- [Configuración del entorno de desarrollo en README.md](#configuración-del-entorno-de-desarrollo-en-readme.md)
- [Persistencia de datos](#persistencia-de-datos)
  - [Consideraciones Generales](#consideraciones-Generales)
  - [SaaS, cloud-hosted(alojamiento en la nube) o self-hosted(autohospedado)?](#alojamiento-en-la-nube-autohospedado)
  - [Soluciones de persistencia](#soluciones-de-persistencia)
    - [RDBMS](#rdbms)
    - [NoSQL](#nosql)
      - [Almacenamiento de documentos](#almacenamiento-de-documentos)
      - [Key-value store(tienda clave-valor)](#key-value-store)
      - [Base de datos de grafico](#base-de-datos-de-grafico)
- [Entornos](#entornos)
  - [Entorno de desarrollo local](#entorno-de-desarrollo-local)
  - [Entorno de integración continua](#entorno-de-integración-continua)
  - [ Entorno de prueba](#entorno-de-prueba)
  - [Entorno de estadificación](#entorno-de-estadificación)
  - [Entorno de producción](#entorno-de-producción)
- [Lista de materiales](#lista-de-materiales)
- [Seguridad](#seguridad)
  - [Docker](#docker)
  - [Credenciales](#credenciales)
  - [Secretos](#secretos)
  - [Regulación de inicio de sesión](#regulación-de-inicio-de-sesión)
  - [Almacenamiento de contraseña de usuario](#almacenamiento-de-contraseña-de-usuario)
  - [Registro de auditoría](#registro-de-auditoría)
  - [Acción sospechosa Regulación y / o bloqueo](#acción-sospechosa-Regulación-y/o-bloqueo)
  - [Datos anonimizados](#datos-anonimizados)
  - [Almacenamiento temporal de archivos](#almacenamiento-temporal-de-archivos)
  - [Entorno de servidor dedicado vs compartido](#entorno-de-servidor-dedicado-vs-compartido)
- [Monitoreo de la aplicación](#monitoreo-de-la-aplicación)
  - [Página de estado](#página-de-estado)
  - [Formato de la página de estado](#formato-de-la-página-de-estado)
    - [Formato plano](#formato-plano)
    - [Formato JSON](#formato-JSON)
  - [HTTP códigos de estado](#http-códigos-de-estado)
  - [Comprobaciones de estado del equilibrador de carga](#comprobaciones-de-estado-del-equilibrador-de-carga)
  - [Control de acceso](#control-de-acceso)
- [ Listas de verificación](#listas-de-verificación)
  - [Lista de verificación de responsabilidad](#lista-de-verificación-de-responsabilidad)
  - [ Lista de verificación de lanzamiento](#lista-de-verificación-de-lanzamiento)
- [Preguntas generales a considerar](#preguntas-generales-a-considerar)
- [Herramientas útiles generalmente probadas](#herramientas-útiles-generalmente-probadas)
- [Licencia](#licencia)

<!-- END doctoc generó TOC por favor mantenga un comentario aquí para permitir la actualización automática -->

# Traducciones de este documento

Estas son traducciones de este documento proporcionadas por la comunidad. Si tiene comentarios sobre una traducción en particular, por favor diríjase al responsable de la traducción.

- [Turkish](https://github.com/umutphp/backend-best-practices) traducción por [umutphp](https://github.com/umutphp)

# N Mandamientos

1. README.md en la raíz del repositorio están los documentos
2. Ejecución de comando único
3. Despliegue de comando único
4. Construcciones repetibles y re-creables
5. Construir artefactos paquete a ["Bill of Materials"](#bill-of-materials)
6. Use [UTC as the timezone](http://yellerapp.com/posts/2015-01-12-the-worst-server-setup-you-can-make.html) all around

# Puntos generales sobre las directrices

No queremos limitarnos a ciertos conjuntos de tecnología o marcos de trabajo. Diferentes problemas requieren diferentes soluciones, y por lo tanto estas directrices son válidas para varias arquitecturas de backend.

# Configuración del entorno de desarrollo en README.md

Documente todas las partes del entorno de desarrollo / servidor. Esfuércese por utilizar la misma configuración y versiones en todos los entornos, comenzando por las computadoras portátiles de los desarrolladores y terminando con el entorno de producción real. Esto incluye la base de datos, el servidor de aplicaciones, el servidor proxy (nginx, Apache, ...), las versiones de SDK, gemas / bibliotecas / módulos.

Automatice el proceso de configuración tanto como sea posible. Por ejemplo, [Docker Compose](https://docs.docker.com/compose/) podría usarse tanto en producción como en desarrollo para establecer un entorno completo, donde[Dockerfiles](https://docs.docker.com/articles/dockerfile_best-practices/) recuperar todas las partes del software, y contener las secuencias de comandos necesarias para configurar el entorno y todas las partes del mismo. Considere el uso de copias archivadas de los instaladores, en caso de que los paquetes de la línea de producción no estén disponibles. Una precaución mínima es mantener una suma de comprobación SHA-1 de los paquetes, y asegurarse de que la suma de comprobación coincide cuando se instalen los paquetes.

Traducción realizada con la versión gratuita del traductor www.DeepL.com/Translator

Considere la posibilidad de almacenar cualquier parte relevante del entorno de desarrollo y sus dependencias en algún almacenamiento persistente. Si el entorno puede ser construido usando Docker, una posible manera de hacerlo es usar [docker export](http://docs.docker.com/reference/commandline/cli/#export).

# Persistencia de datos

## Consideraciones Generales

Independientemente de la solución de persistencia que use su proyecto, existen consideraciones generales que debe seguir:

* Tener copias de seguridad verificadas para funcionar
* Tener scripts u otras herramientas para copiar datos persistentes de un entorno a otro, desde la producción hasta la puesta en escena para depurar algo
* Disponer de planes para la implementación de actualizaciones de la solución de persistencia (por ejemplo, actualizaciones de seguridad del servidor de la base de datos)
* Tenga planes establecidos para ampliar la solución de persistencia
* Tener planes o herramientas para administrar los cambios de esquema.
* Implemente un monitoreo para verificar el estado de la solución de persistencia.

## SaaS, cloud-hosted(alojamiento en la nube) o self-hosted(autohospedado)?

Una opción importante con respecto a cualquier solución es dónde ejecutarla.

* SaaS -- rápido para comenzar, fácil de escalar, se requiere trabajo de infraestructura para permitir el acceso desde cualquier lugar, etc.
* Self-hosted in the cloud -- permite ajustar la base de datos más que SaaS y probablemente más barato a escala en términos de alojamiento, pero requiere más mano de obra
* Self-hosted on own hardware -- capaz de modificar todo y administrar la seguridad física, pero el más costoso y laborioso

## Soluciones de persistencia

Esta sección tiene como objetivo proporcionar una guía para seleccionar el tipo de solución de persistencia. Sin embargo, la elección siempre debe adaptarse al problema y ninguno de estos es una bala de plata.

### RDBMS

Elija un sistema de base de datos relacional como PostgreSQL cuando la integridad de los datos y las transacciones sea una preocupación importante o cuando se requiera un gran número de análisis de datos. El [ACID compliance](https://en.wikipedia.org/wiki/ACID), Las funciones de agregación y transformación del RDBMS ayudarán.
### NoSQL

Elija una base de datos NoSQL cuando espere escalar horizontalmente y cuando no necesite ACID. Elija un sistema que se adapte a su modelo.

#### Almacenamiento de documentos

Almacena documentos que pueden abordarse y buscarse fácilmente por contenido o por inclusión en una colección. Esto es posible porque la base de datos comprende el formato de almacenamiento. Úselo solo para eso: almacenar grandes cantidades de documentos estructurados. Ejemplos notables:

* CouchDB
* ElasticSearch

> Tenga en cuenta que desde 9.4, PostgreSQL también se puede usar para almacenar JSON de forma nativa

#### Key-value store(tienda clave-valor)

Almacena valores, o a veces grupos de pares clave-valor, accesibles por clave. Considera que los valores son simplemente blobs, por lo que no proporciona las capacidades de consulta de los almacenes de documentos. Escalable a inmensos tamaños. Ejemplos notables:

* Cassandra
* Redis

#### Base de datos de grafico

Las bases de datos de gráficos generales almacenan nodos y bordes de un gráfico, proporcionando búsquedas libres de índices de los vecinos de cualquier nodo. Para aplicaciones en las que las consultas de tipo gráfico, como el camino más corto o el diámetro, son cruciales. También existen bases de datos gráficas especializadas para almacenar, por ejemplo [RDF triples](https://en.wikipedia.org/wiki/Resource_Description_Framework).

# Entornos

Esta sección describe los entornos que debe tener, como mínimo. Puede parecer mucho, [pero hay un propósito para cada uno](http://futurice.com/blog/five-environments-you-cannot-develop-without).

- [Local development](#local-development-environment)
- [Continuous integration](#continuous-integration-environment)
- [Testing](#testing-environment)
- [Staging](#staging-environment)
- [Production](#production-environment)

## Entorno de desarrollo local

Este es su entorno de desarrollo local. Probablemente no debería tener un entorno de desarrollo externo compartido. En su lugar, debe trabajar para que sea posible ejecutar todo el sistema localmente, tropezando o burlándose de servicios de terceros según sea necesario.

## Entorno de integración continua

CI es (entre otras cosas) para asegurarse de que las compilaciones de software y las pruebas automatizadas pasan después de cada cambio.

## Entorno de prueba

Este es un entorno compartido donde el código se implementa con la mayor frecuencia posible, preferiblemente cada vez que el código se confirma en la rama principal. Puede romperse de vez en cuando, especialmente en la fase de desarrollo activo. Es un entorno canario importante y es lo más similar posible a la producción. Todas las integraciones externas se configuran para usar versiones de otros servicios a nivel provisional.

## Entorno de estadificación

La puesta en escena se configura exactamente como la producción. No hay cambios en el entorno de producción ocurren antes de haber sido ensayado aquí en primer lugar. Aquí se puede depurar cualquier problema de producción misterioso.

## Entorno de producción

El gran hierro. Registrado, monitoreado, limpiado periódicamente, ajustado y asegurado.

# Lista de materiales

Este documento debe incluirse en cada artefacto de compilación y debe contener lo siguiente:

1. Qué versión (es) de un SDK y herramientas críticas se usaron para producirlo
1. Qué dependencias se han incluido
1. Un número de revisión globalmente único de la compilación (es decir, un hash git SHA-1)
1. El entorno y las variables utilizadas al construir el paquete.
1. Una lista de pruebas o verificaciones fallidas


# Seguridad

Tenga en cuenta las posibles amenazas y problemas de seguridad. Al menos deberías estar familiarizado con el[Las 10 vulnerabilidades principales de OWASP](https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project), y debe monitorear las vulnerabilidades en cualquier software de terceros que use.

Las buenas pautas de seguridad genéricas serían:

## Docker

**Usar Docker no hará que su servicio sea más seguro.** En general, debe considerar al menos seguir las siguientes cosas si usa Docker:

- No ejecute ningún binario no confiable dentro de los contenedores Docker
- Cree usuarios sin privilegios dentro de los contenedores de Docker y ejecute archivos binarios con usuarios sin privilegios en lugar de root siempre que sea posible
- Periódicamente reconstruya y vuelva a implementar sus contenedores con bibliotecas y dependencias actualizadas
- Actualice periódicamente (o reconstruya) sus hosts Docker con las últimas actualizaciones de seguridad
- Por defecto, varios contenedores que se ejecutan en el mismo host tendrán cierto nivel de acceso a otros contenedores y al propio host. Asegure adecuadamente todos los hosts y ejecute contenedores con un conjunto mínimo de capacidades, por ejemplo, evitando el acceso a la red si no lo necesitan.

## Credenciales

Nunca envíe credenciales sin cifrar a través de la red pública. Utilice siempre el cifrado (como HTTPS, SSL, etc.).
## Secretos

¡Nunca almacene secretos (contraseñas, claves, etc.) en las fuentes en el control de versiones! Es muy fácil olvidar que están allí y la fuente del proyecto tiende a terminar en muchos lugares (máquinas de desarrollo, servidores de prueba de desarrollo, etc.), lo que aumenta innecesariamente el riesgo de que un secreto importante se vea comprometido. Además, el control de versiones tiene la característica desagradable de sobrescribir los permisos de los archivos, por lo que, incluso si asegura los permisos del archivo de configuración, la próxima vez que revise la fuente, los permisos se sobrescribirán en la lectura pública predeterminada.

Probablemente, la forma más fácil de manejar secretos es ponerlos en un archivo separado en los servidores que los necesitan, y ser ignorado por el control de versiones. Puedes mantener un archivo `.sample` en el control de versión, con valores falsos para ilustrar lo que debe ir allí en el archivo real. En algunos casos, no es fácil incluir un archivo de configuración separado de la configuración principal. Si esto sucede, considere usar variables de entorno o escribir el archivo de configuración desde una plantilla controlada por versión en la implementación.

## Regulación de inicio de sesión

Establezca límites en la cantidad de intentos de inicio de sesión permitidos por cliente por unidad de tiempo. Bloquee una cuenta de usuario durante un tiempo específico después de un número determinado de intentos fallidos (por ejemplo, bloquee durante 5 minutos después de 20 intentos fallidos de inicio de sesión).
El objetivo de estas medidas es hacer que los ataques de fuerza bruta en línea contra nombres de usuario / contraseñas no sean factibles

## Almacenamiento de contraseña de usuario

> ¡NUNCA almacene contraseñas en texto plano!

Nunca almacene contraseñas en forma cifrada reversible, a menos que la aplicación / sistema lo requiera absolutamente. Aquí hay un buen artículo sobre qué y qué no hacer:https://crackstation.net/hashing-security.htm

Si necesita poder obtener contraseñas de texto sin formato de la base de datos, aquí hay algunas sugerencias a seguir.

Si las contraseñas no se convertirán de nuevo a texto sin formato a menudo (por ejemplo, se requiere un procedimiento especial), mantenga las claves de descifrado lejos de la aplicación que accede a la base de datos regularmente.

Si las contraseñas aún necesitan ser descifradas regularmente, separe la funcionalidad de descifrado de la aplicación principal tanto como sea posible, por ejemplo. un servidor separado acepta solicitudes para descifrar una contraseña, pero impone un mayor nivel de control, como limitación, autorización, etc.

Siempre que sea posible (debería ser en la gran mayoría de los casos), almacene las contraseñas utilizando un buen hash unidireccional con una buena sal aleatoria. Y no, SHA-1 no es una buena opción para una función hash en este contexto. Las funciones de hash diseñadas con contraseñas en mente son deliberadamente más lentas, lo que hace que los ataques de fuerza bruta fuera de línea sean más lentos y, por lo tanto, menos factibles. Vea esta publicación para más detalles: http://security.stackexchange.com/questions/211/how-to-securely-hash-passwords/31846#31846

## Registro de auditoría

Para las aplicaciones que manejan datos confidenciales, especialmente donde a ciertos usuarios se les permite un acceso o control relativamente amplio, es bueno mantener algún tipo de registro de auditoría, almacenando una secuencia de acciones / eventos que tuvieron lugar en el sistema, junto con el origen del evento / fuente (usuario, trabajo de automatización, etc.). Esto puede ser, por ejemplo:

    2012-09-13 03:00:05 Job "daily_job" acción realizada" "eliminar elementos antiguos".
    2012-09-13 12:47:23 User "admin_user" acción realizada "eliminar elemento 123".
    2012-09-13 12:48:12 User "admin_user" acción realizada "cambiar la contraseña del usuario foobar".
    2012-09-13 13:02:11 User "sneaky_user" acción realizada "ver página confidencial 567".
    ...

El registro puede ser un archivo de texto simple o almacenado en una base de datos. Es bueno tener al menos estos tres elementos: una marca de tiempo exacta, el creador de la acción / evento (quién hizo esto) y la acción / evento real (lo que se hizo). Las acciones exactas a registrar dependen de lo que es importante para la aplicación en sí, por supuesto.

El registro de auditoría puede ser parte del registro normal de la aplicación, pero el énfasis aquí está en registrar quién hizo qué y no solo en que se realizó una determinada acción. Si es posible, el registro de auditoría debe hacerse a prueba de manipulaciones, solo será accesible por un usuario o proceso de registro dedicado y no directamente por la aplicación.

## Acción sospechosa Regulación y / o bloqueo

Esto puede verse como una generalización de la limitación de inicio de sesión, esta vez presentando mecanismos similares para acciones arbitrarias que se consideran "sospechosas" en el contexto de la aplicación. Por ejemplo, un sistema ERP que permite a los usuarios normales acceder a una cantidad sustancial de información, pero espera que los usuarios se preocupen solo por un pequeño subconjunto de esa información, puede limitar los intentos de acceder a conjuntos de datos más grandes de lo esperado demasiado rápido. P.ej. evitar que los usuarios descarguen la lista de todos los clientes, si se supone que los usuarios deben trabajar en uno o dos clientes a la vez. Tenga en cuenta que esto es diferente de limitar el acceso por completo: los usuarios aún pueden recuperar información sobre cualquier cliente, pero no todos a la vez. Dependiendo del sistema, la limitación podría no ser suficiente, por ejemplo. cuando se invoca una acción en todos los recursos con una sola solicitud. Entonces puede ser necesario el bloqueo. Tenga en cuenta la diferencia entre realizar 1000 solicitudes en 10 segundos para recuperar la información completa del cliente, un cliente a la vez, y realizar una sola solicitud para recuperar esa información a la vez.

Lo sospechoso aquí depende en gran medida del uso esperado de la aplicación. en un sistema, eliminar 10000 registros puede ser una acción completamente legítima, pero no lo es en otro.

## Datos anonimizados

Siempre que se exporten grandes conjuntos de datos a terceros, los datos deben ser anonimizados tanto como sea posible, dado el uso previsto de los datos. Por ejemplo, si un servicio de un tercero proporcionará un análisis estadístico general en una base de datos de clientes, probablemente no necesite conocer los nombres, direcciones u otra información personal para clientes individuales. Incluso un número de identificación de cliente genérico puede ser demasiado revelador, dependiendo del conjunto de datos. Mira este articulo: http://arstechnica.com/tech-policy/2009/09/your-secrets-live-online-in-databases-of-ruin/.

Evite el registro de información de identificación personal, por ejemplo, el nombre del usuario.

Si sus registros contienen información confidencial, asegúrese de saber cómo están protegidos los registros y dónde están ubicados también en el caso de los sistemas de administración de registros alojados en la nube

Si debe registrar información confidencial, intente el hash antes de iniciar sesión para poder identificar la misma entidad entre diferentes partes del procesamiento.

## Almacenamiento temporal de archivos

Asegúrese de saber dónde almacena su aplicación archivos temporales. Si está utilizando directorios de acceso público (que probablemente sean los predeterminados) como `/ tmp` y` / var / tmp`, asegúrese de crear sus archivos con el modo 600, de modo que solo el usuario pueda leerlos. corriendo como. Alternativamente, tenga un directorio protegido para almacenar archivos temporales (directorio accesible solo para el usuario de la aplicación).

## Entorno de servidor dedicado vs compartido

Las amenazas de seguridad pueden ser bastante diferentes dependiendo de si la aplicación se ejecutará en un entorno compartido o dedicado. Compartido aquí significa que hay otras aplicaciones (no necesariamente de terceros) que se ejecutan en el mismo servidor. En ese caso, tener permisos de archivo apropiados se vuelve crítico, de lo contrario, el código fuente de la aplicación, los archivos de datos, los archivos temporales, los registros, etc., podrían ser accesibles para usuarios no deseados. Luego, una violación de la seguridad en una aplicación de terceros puede hacer que su aplicación se vea comprometida.

Nunca puede estar seguro de qué tipo de entorno ejecutará su aplicación durante toda su vida útil; puede comenzar en un servidor dedicado, pero a medida que pasa el tiempo, las aplicaciones de terceros pueden agregarse al mismo sistema. Es por eso que es mejor planificar desde el primer momento en que su aplicación se ejecuta en un entorno compartido y tomar todas las precauciones. Aquí hay una lista no exhaustiva de los archivos / directorios en los que debe pensar:

* código fuente de la aplicación
* directorios de datos
* directorios de almacenamiento temporal (a menudo, de manera predeterminada, se puede usar el sistema wide / tmp - ver arriba)
* Archivos de configuración
* directorios de control de versiones - .git, .hg, .svn, etc.
* secuencias de comandos de inicio (pueden contener variables de inicialización, secretos, etc.)
* archivos de registro
* vertederos de emergencia
* claves privadas (SSL, SSH, etc)
* etc.

A veces, algunos archivos necesitan ser accesibles por diferentes usuarios (por ejemplo, contenido estático servido por apache). En ese caso, tenga cuidado de permitir el acceso sólo a lo que realmente se necesita.

Tenga en cuenta que, en un sistema de archivos UNIX / Linux, el acceso de escritura a un directorio es,en términos de permisos, muy poderoso: le permite eliminar archivos en ese directorio y volver a crearlos (lo que da como resultado un archivo modificado). / tmp y / var / tmp están a salvo de este efecto por defecto, debido al bit fijo que debería establecerse en ellos.

Además, como se menciona en la sección de secretos, los permisos de los archivos podrían no preservarse en el control de versiones, por lo que incluso si los configura una vez, la siguiente comprobación/actualización/lo que sea podría anularlos. Una buena idea es entonces tener un Makefile, un script, un gancho de control de versiones o algo similar que establezca los permisos correctos al actualizar las fuentes.

Traducción realizada con la versión gratuita del traductor www.DeepL.com/Translator

# Monitoreo de la aplicación

La supervisión del estado completo de un servicio requiere que se realicen comprobaciones de supervisión específicas del nivel del sistema operativo y de la aplicación. Las comprobaciones a nivel del sistema operativo incluyen, por ejemplo, uso de CPU, disco o memoria, procesos en ejecución, puertos abiertos, etc. Sin embargo, las comprobaciones específicas de la aplicación son las más importantes desde el punto de vista del servicio en ejecución. Estos pueden ser desde "responde este URL y devuelve el estado HTTP 200", hasta verificar la conectividad de la base de datos, la consistencia de los datos, etc.

Esta sección describe una forma de implementar las verificaciones específicas de la aplicación, lo que facilitaría el monitoreo del estado general de la aplicación y otorgaría un control total a los desarrolladores de la aplicación para determinar qué verificaciones son significativas en el contexto de la aplicación concreta.

En esencia, la idea es tener un único punto final (una URL de la aplicación) que pueda proporcionar una buena visión general del estado de toda la aplicación. Esto se implementa dentro de la aplicación y requiere el trabajo del equipo del proyecto, pero por otro lado, el equipo del proyecto es el que realmente puede definir qué es un estado correcto de la aplicación y qué se considera un estado de ERROR.

La aplicación podría implementar cualquier número de comprobaciones de "subsistema". Por ejemplo,

* la conexión a la base de datos está activa
* los datos están en un estado consistente (por ejemplo, una lista de elementos en una determinada       tabla de base de datos es significativa)
* Los servicios de terceros a los que se integra la aplicación son accesibles
* Los índices de ElasticSearch están en un estado consistente
* cualquier otra cosa que tenga sentido para la aplicación

La aplicación debe proporcionar un estado de resumen combinado, agregando la información de las diversas verificaciones del subsistema. La idea es que un sistema de monitoreo externo solo pueda rastrear esta visión general combinada, de modo que no sea necesario volver a configurar el monitoreo externo cuando se agrega o modifica una nueva verificación de la aplicación. Además, los desarrolladores son los que pueden decidir en qué se basa el estado general con respecto a las verificaciones del subsistema (es decir, cuáles son críticos, mientras que otros no, etc.).

## Página de estado

Todas las comprobaciones de estado DEBEN ser accesibles bajo `/status` URLs  de la siguiente manera:

* `/status` - la página de estado general (obligatorio)
* `/status/subsystem1` - una verificación del estado del subsistema específico (opcional)
* ...

La página principal `/ status` debe dar como mínimo un estado general del sistema, como se describe en la siguiente sección. Esto significa que la página principal `/ status` debe ejecutar TODAS las comprobaciones del subsistema e informar el estado general del sistema agregado. Depende de los desarrolladores decidir cómo se determina el estado general del sistema en función de los subsistemas. Por ejemplo, un estado `ERROR` de algún subsistema no crítico solo puede generar un estado general de` ADVERTENCIA`.

Por razones de rendimiento, algunas verificaciones del subsistema pueden excluirse de esta página general de `/ estado` - por ejemplo, cuando la verificación causa un mayor uso de recursos, tarda más tiempo en completarse, etc. En general, la página de estado principal debe ser lo suficientemente clara como para que puede sondearse con relativa frecuencia (cada 1-3 minutos) y no causar demasiada carga en el sistema. Las verificaciones del subsistema que están excluidas de la verificación del estado general deben tener sus propias URL, como se muestra arriba. Naturalmente, monitorearlos requeriría modificaciones en la configuración del sistema de monitoreo. Para superar esto, se puede adoptar un enfoque diferente: la aplicación podría realizar las verificaciones pesadas del subsistema en un proceso en segundo plano a una velocidad aceptable y almacenar el estado internamente. Esto permitiría que la página de estado principal refleje también estas verificaciones pesadas (por ejemplo,recuperaría el último estado de verificación realizado). Este enfoque debe usarse, a menos que su implementación sea demasiado difícil.

## Formato de la página de estado

Proponemos dos formatos alternativos para las páginas de estado.- `plain` y `JSON`.

### Formato plano

El formato plano tiene un estado por línea en la forma "clave: valor". La clave es un nombre de subsistema/verificación y el valor es el valor de status. El valor de status puede ser uno de:

* `OK`
* `WARN Message`
* `ERROR Message`

donde "Mensaje" puede ser un texto significativo, que puede ayudar a identificar rápidamente el problema. El mensaje es de una sola línea, sin restricción de longitud especificada, pero use el sentido común - por ejemplo, probablemente no debe tener más de 200 caracteres.

La página principal de status DEBE tener una clave llamada "status" que muestre el status global de la aplicación agregada. Las líneas de status de verificación del subsistema individual son opcionales. Las claves de status del subsistema deberían tener un sufijo "status". A continuación se muestran algunos ejemplos:

Cuando todo esté bien:

```
status: OK
database_status: OK
elastic_search_status: OK
```

Cuando algún cheque falla:

```
status: ERROR Database is not accessible
database_status: ERROR Connection failed
elastic_search_status: OK
```

Múltiples fallas al mismo tiempo:

```
status: ERROR failed subsystems: base de datos, búsqueda elástica. Para más detalles, vér https://myapp.example.com/status
database_status: ERROR Connection failed
elastic_search_status: WARN Too few entries in index A.
```

Además de las líneas de estado, una página de estado puede tener claves que no sean de estado. Por ejemplo, esos pueden mostrar algunas métricas (que pueden o no ser monitoreadas). Las claves adicionales deben tener como prefijo el nombre del subsistema.

```
status: OK
database_status: OK
database_customers: 378
database_items: 8934748
elastic_search_status: OK
elastic_search_shards: 20
```

El estado general puede basarse naturalmente en algunas de las métricas:

```
status: WARN Muy pocos elementos en la base de datos.
database_status: WARN Muy pocos clientes en la base de datos
database_customers: 378
database_items: 1
elastic_search_status: OK
elastic_search_shards: 20
```

Las comprobaciones de subsistemas que tienen su propia URL (`/ status / subsystemX`) deben seguir un formato similar, con una clave obligatoria` status` y varias claves adicionales opcionales. Ejemplo por ej. `/ status / database`:

```
status: OK
connection_pool: 30
latency: 2
```

### Formato JSON

El formato JSON de las páginas de estado a menudo puede ser preferible, por ejemplo, cuando las herramientas o la integración a otros sistemas es más fácil de lograr a través de un formato de datos común.

Los valores de estado siguen el mismo formato que el descrito anteriormentew- `OK`, `WARN Message` y `ERROR Message`.

El equivalente a la clave de estado del formato plano es una clave de `status` en el objeto JSON raíz.
Los subsistemas deben usar objetos anidados que también tengan una clave de `status` obligatoria. Aquí hay algunos ejemplos:

Todo está bien:

```json
{
    "status": "OK",
    "database": {
        "status": "OK"
    },
    "elastic_search": {
        "status": "OK"
    }
}
```

Página de estado con métricas adicionales:

```json
{
    "status": "OK",
    "uptime": 18234,
    "database": {
        "status": "OK",
        "connection_pool": 30
    },
    "elastic_search": {
        "status": "OK",
        "multinode": false
    }
}
```

Algo está fallando:

```json
{
    "status": "ERROR La base de datos no es accesible. ver https://myapp.example.com/status para detalles.",
    "database": {
        "status": "ERROR La conexión falló",
        "connection_timeout": 30
    },
    "elastic_search": {
        "status": "OK"
    }
}
```

## HTTP códigos de estado

Siempre que el estado general de la aplicación sea correcto, el código de estado HTTP en la respuesta de la página de estado DEBE establecerse en 200 (correcto). De lo contrario, DEBE establecerse un código de error 5XX. Por ejemplo, se podría usar el código 500 (Error interno del servidor). Opcionalmente, el estado WARN no crítico aún puede responder con 200.

## Comprobaciones de estado del equilibrador de carga

A menudo, la aplicación se ejecuta detrás de un equilibrador de carga. Los equilibradores de carga generalmente pueden monitorear los servidores de aplicaciones sondeando una URL determinada. La comprobación de estado se utiliza para que el equilibrador de carga pueda detener el enrutamiento del tráfico a los servidores de aplicaciones que fallan.

La página general `/ status` es un buen candidato para la URL de comprobación de estado del equilibrador de carga. Sin embargo, una página de estado dedicada separada para una comprobación del estado del equilibrador de carga proporciona un beneficio importante. Dicha página se puede ajustar para cuando la aplicación se considera saludable desde la perspectiva del equilibrador de carga. Por ejemplo, un error en un subsistema aún puede considerarse un error crítico para el estado general de la aplicación, pero no necesariamente tiene que hacer que el servidor de aplicaciones se elimine del grupo de equilibradores de carga. Un buen ejemplo es una verificación de estado de integración de terceros. La página de comprobación del estado del equilibrador de carga solo debe devolver un código de estado que no sea 200 cuando la instancia de la aplicación debe considerarse no operativa.

La página de comprobación del estado del equilibrador de carga debe colocarse en una URL `/status/ health`. Dependiendo de su balanceador de carga, el formato de esa página puede diferir del formato de estado general descrito aquí. Algunos equilibradores de carga pueden incluso observar solo el código de estado HTTP devuelto.

## Control de acceso

Las páginas de estado pueden necesitar una autorización adecuada, especialmente en caso de que expongan información de depuración en mensajes de estado o métricas de aplicaciones. La autenticación básica HTTP o las restricciones basadas en IP suelen ser candidatos suficientemente buenos para considerar.

# Listas de verificación

Para evitar olvidar las cosas más importantes, aquí hay algunas listas de verificación útiles para sus proyectos actuales o futuros.

## Lista de verificación de responsabilidad

En proyectos más grandes, especialmente cuando hay múltiples partes involucradas, es crucial hacer un seguimiento de todos los diferentes aspectos y sus responsabilidades. La siguiente tabla ilustra cómo se vería una lista de verificación para lanzar un sitio web:

| Aspecto   | Tarea                             | persona responsable/ parte | Fecha limite | Estado           |
|---        |---                                |---                         |---           |---                |
| Frontend  | Wireframes del sitio web          | e.g. Company B / Person X  | e.g. 17.6.   |  e.g. en progreso |
| Frontend  | Diseño de sitios web              | e.g. Company A / Person Z  | e.g. 23.7.   |  e.g. waiting     |
| Frontend  | Plantillas de sitios web          |   |   |   |
| Frontend  | Creación de contenido y población.|   |   |   |
| Backend   | Configurar CMS                    |   |   |   |
| Backend   | Configurar entorno de preparación |   |   |   |
| Backend   | Configurar entorno de producción  |   |   |   |
| Backend   | Migrar servicios de alojamiento a cuentas de clientes |   |   |   |
| Backend   | DNS configuracion                 |   |   |   |
| Backend   | Configurar análisis de sitios web |   |   |   |
| Backend   | Integrar la automatización del marketing.    |   |   |   |
| Backend   | Licencia de fuentes Web           |   |   |   |
| Dates     | Tiempo de vida de la página web/producto      |   |   |   |
| Dates     |Publicar el sitio web              |   |   |   |

## Lista de verificación de lanzamiento

Cuando esté listo para lanzar, recuerde marcar todo en su lista de verificación de lanzamiento. La tranquilidad mental, la repetibilidad y la fiabilidad resultantes son una gran bendición.

Usted * tiene * uno, ¿verdad? Si no lo hace, aquí hay un buen punto de partida genérico para usted:

* [ ] La implementación funciona igual sin importar en qué entorno se esté implementando
* [ ] Todos los entornos tienen nombres bien definidos, y se hace referencia a ellos utilizando esos nombres.
* [ ] Todos los entornos tienen la misma pila de software subyacente.
* [ ]Toda la configuración del entorno está controlada por la versión (configuración del servidor web, scripts de compilación de CI, etc.)
* [ ] El producto ha sido probado desde las redes desde donde se utilizará (por ejemplo, Internet público, LAN del cliente)
* [ ] El producto ha sido probado con todos los dispositivos específicos.
* [ ] Hay una manera simple de averiguar qué código se está ejecutando en un entorno determinado
* [ ] Se ha definido un esquema de versiones.
* [ ] Cualquier versión del producto debe poder mapearse fácilmente a un estado de la base del código
* [ ] Revertir un despliegue es posible
* [ ] Las copias de seguridad se están ejecutando
* [ ] La restauración desde una copia de seguridad ha sido probada
* [ ] No se almacenan secretos en el control de versiones
* [ ] El registro está activado
* [ ] Existe un proceso bien definido para acceder y buscar en los registros.
* [ ] El registro incluye excepciones y seguimientos de pila cuando corresponda
* [ ] Los errores se pueden asignar a los rastros de la pila
* [ ] Se han escrito notas de liberación
* [ ] Los entornos de servidores están actualizados
* [ ] Existe un plan para actualizar los entornos del servidor
* [ ] El producto ha sido probado en carga
* [ ]Existe un método para replicar el estado de un entorno en otro (por ejemplo, copiar prod a QA para reproducir un error)
* [ ] Todos los procesos de lanzamiento repetido han sido automatizados

# Preguntas generales a considerar

* ¿Cuál es la vida útil esperada / requerida del proyecto?
* ¿El proyecto es único o habrá un desarrollo continuo?
* ¿Cuál es el ciclo de lanzamiento de una versión del servicio?
* ¿Qué entornos (desarrollo, prueba, puesta en escena, producción, ...) se van a configurar?
* ¿Cómo afectará el tiempo de inactividad del servicio de producción al valor del servicio?
* ¿Qué tan madura es la tecnología? ¿Se esperan grandes cambios que rompan la compatibilidad con versiones anteriores?

# Herramientas útiles generalmente probadas

* [HTTPie](https://github.com/jakubroztocil/httpie) es una gran herramienta para probar API en la línea de comando. Es simple pasar encabezados y cookies personalizados, e incluso tiene soporte de sesión.
* [jq](http://stedolan.github.io/jq/) 
es un procesador CLI JSON. Masajee los datos JSON procedentes de cURL (¡o, por supuesto, HTTPie!) A voluntad. Otra gran herramienta para pruebas de API o exploración.

# Licencia

[Futurice Oy](http://www.futurice.com)
Creative Commons Reconocimiento 4.0 Internacional (CC BY 4.0)