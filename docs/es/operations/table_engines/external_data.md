---
machine_translated: true
---

# Datos externos para el procesamiento de consultas {#external-data-for-query-processing}

ClickHouse permite enviar a un servidor los datos necesarios para procesar una consulta, junto con una consulta SELECT. Estos datos se colocan en una tabla temporal (consulte la sección “Temporary tables”) y se puede utilizar en la consulta (por ejemplo, en operadores IN).

Por ejemplo, si tiene un archivo de texto con identificadores de usuario importantes, puede cargarlo en el servidor junto con una consulta que utilice la filtración de esta lista.

Si necesita ejecutar más de una consulta con un gran volumen de datos externos, no utilice esta función. Es mejor cargar los datos a la base de datos con anticipación.

Los datos externos se pueden cargar mediante el cliente de línea de comandos (en modo no interactivo) o mediante la interfaz HTTP.

En el cliente de línea de comandos, puede especificar una sección de parámetros en el formato

``` bash
--external --file=... [--name=...] [--format=...] [--types=...|--structure=...]
```

Puede tener varias secciones como esta, para el número de tablas que se transmiten.

**–externo** – Marca el comienzo de una cláusula.
**–file** – Ruta al archivo con el volcado de tabla, o -, que hace referencia a stdin.
Solo se puede recuperar una sola tabla de stdin.

Los siguientes parámetros son opcionales: **–nombre**– Nombre de la tabla. Si se omite, se utiliza \_data.
**–formato** – Formato de datos en el archivo. Si se omite, se utiliza TabSeparated.

Se requiere uno de los siguientes parámetros:**–tipo** – Una lista de tipos de columnas separadas por comas. Por ejemplo: `UInt64,String`. Las columnas se llamarán \_1, \_2, …
**–estructura**– La estructura de la tabla en el formato`UserID UInt64`, `URL String`. Definir los nombres y tipos de columna.

Los archivos especificados en ‘file’ se analizará mediante el formato especificado en ‘format’ utilizando los tipos de datos especificados en ‘types’ o ‘structure’. La mesa será cargado en el servidor y accesibles, como una tabla temporal con el nombre de ‘name’.

Ejemplos:

``` bash
$ echo -ne "1\n2\n3\n" | clickhouse-client --query="SELECT count() FROM test.visits WHERE TraficSourceID IN _data" --external --file=- --types=Int8
849897
$ cat /etc/passwd | sed 's/:/\t/g' | clickhouse-client --query="SELECT shell, count() AS c FROM passwd GROUP BY shell ORDER BY c DESC" --external --file=- --name=passwd --structure='login String, unused String, uid UInt16, gid UInt16, comment String, home String, shell String'
/bin/sh 20
/bin/false      5
/bin/bash       4
/usr/sbin/nologin       1
/bin/sync       1
```

Cuando se utiliza la interfaz HTTP, los datos externos se pasan en el formato multipart/form-data. Cada tabla se transmite como un archivo separado. El nombre de la tabla se toma del nombre del archivo. El ‘query\_string’ se pasa los parámetros ‘name\_format’, ‘name\_types’, y ‘name\_structure’, donde ‘name’ es el nombre de la tabla a la que corresponden estos parámetros. El significado de los parámetros es el mismo que cuando se usa el cliente de línea de comandos.

Ejemplo:

``` bash
$ cat /etc/passwd | sed 's/:/\t/g' > passwd.tsv

$ curl -F 'passwd=@passwd.tsv;' 'http://localhost:8123/?query=SELECT+shell,+count()+AS+c+FROM+passwd+GROUP+BY+shell+ORDER+BY+c+DESC&passwd_structure=login+String,+unused+String,+uid+UInt16,+gid+UInt16,+comment+String,+home+String,+shell+String'
/bin/sh 20
/bin/false      5
/bin/bash       4
/usr/sbin/nologin       1
/bin/sync       1
```

Para el procesamiento de consultas distribuidas, las tablas temporales se envían a todos los servidores remotos.

[Artículo Original](https://clickhouse.tech/docs/es/operations/table_engines/external_data/) <!--hide-->
