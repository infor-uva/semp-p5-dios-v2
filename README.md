# semp-p5-dios-v2
Este programa facilitará mucho la practica 5 de sistemas empotrados

> [!note]
> Si se dispone del pdf el comando [`exec_command`](#exec_command) abarca los puntos del 3 al 6, parte del 11 y se ofrece apoyo para los puntos 9 y 13 a través del resto de comandos ([`copy_command`](#copy_command), [`copy_library`](#copy_library), [`copy_link_file`](#copy_link_file) y [`cpo`](#cpo)).


> [!Important]
> Todos los comandos requieren de disponer en el entorno la configuración planteada en [`setup`](setup) ya que tras integrarlo aporta acceso a los comandos de manera global y variables de entorno.
>
> Además, en cualquiera de los comandos si no se especifica el destino se usará la variable `DEFAULT_DEST` definida en el setup, por lo que si no se ha cargado los comandos fallarán.
>
> Se recomienda que el uso de los mismos se haga, sobre todo al comienzo, con directorios destino vacíos para evitar problemas indeseados.
## Uso
Primero ejecutemos la configuración para integrar valores al entorno. Puede ejecutarse desde rutas absolutas
```bash
source ./setup
source /ruta/hasta/el/fichero/setup
```
Una vez configurado dispondremos en todo el sistema de los comandos. El más interesante es `exec_command` que se encargará de toda la tarea automatizada. Pero también se disponen de otros comandos útiles.
### `exec_command`
```bash
exec_command [ dest_path ]
```
> [!note]
> En caso de que no se fie del funcionamiento se recomienda generar un directorio en `/tmp` del sistema de ficheros actual, ejecutar el comando con ese destino y probarlo mediante el comando `chroot <ruta_al_directorio>`.
>
> ```bash
> ruta=/tmp/prueba/repo
> mkdir -p "$ruta"
> exec_command "$ruta"
> chroot "$ruta"
> ```
> Muy importante, revise que el sistema de resolución de usuarios funcione correctamente.

Genera una estructura básica de ficheros a partir del actual para una ruta concreta (`dest_path`).

El comando copiará los directorios `/dev` y `/etc` si detecta que alguno de estos directorios **no está** en el destino, se deberá tener esto en cuenta si se desea repetir todo el proceso.

De manera similar con la estructura de directorios que **no** se rellenará inicialmente (`/bin` `/lib` `/lib64` `/proc` `/root` `/sys` `/tmp` `/sbin` `/usr/bin` `/usr/sbin` `/var/state/dhcp` `/var/tmp` `/var/log` `/dev/shm`), en este caso la condición para volver a ejecutar la creación y configuración es la existencia del directorio `/root` o `/tmp` en el destino.

Una vez establecido lo básico del sistema de ficheros se empezará a copiar los comandos respetando sus enlaces, copiando por tanto tanto enlaces como los ficheros a los que apuntan y sus directorios si no existen (esto se hará a través del comando [`copy_command`](#copy_command)) y las librerías dinámicas de las que dependen asi mismo también se aplica a las dependencias de las propias librerías (a través del comando [`copy_library`](#copy_library)).
> [!note]
> Estos comandos incluyen los comandos del sistema y usuario juntos con unos mínimos para la ejecución de /sbin/init.

> [!important]
> En caso de que ya exista el ejecutable en el destino se interpretará como que ya ha sido copiado y se pasará al siguiente comando. Lo mismo con las librerías. 
> 
> Si un comando o librería existe se interpreta que las librerías de las que depende también


También copiará librerías dinámicas imprescindibles para el nuevo sistema de ficheros, puesto que afecta directamente entre otras a la información de los usuarios, y una librería imprescindible para los comandos con dependencias de librerías dinámicas `/lib64/ld-linux-x86-64.so.2`. Se verifica primero que no existan en el destino antes de iniciar el proceso.

> [!important]
> En caso de que se produzca cualquier problema y se desee ejecutar todo de nuevo se recomienda encarecidamente borrar antes de ejecutar la parte afectada en el destino o todo el contenido del destino.
> 
> Ya que debido a las comprobaciones automáticas este proceso se saltará pasos.
> 
> O, por el contrario modificar el código para ejecutar siempre todos los pasos (**no recomendado**)

### `copy_command`
```bash
copy_command <command> [ <dest_path> ]
```
Copia un comando, los enlaces a los que apunta, las librerías de las que depende y las librerías de las que cada una de estas librerías depende. 

Se puede especificar a que **ruta raíz** se desea copiar, ya que respeta la ruta del anterior sistema:
```bash
copy_command bash /tmp/prueba/repo
# El comando obtendrá la ruta del comando a través del comando `which` 
# dando como resultado /usr/bin/bash y como hemos especificado la ruta se copiará
# en /tmp/prueba/repo/usr/bin/bash haciendo lo mismo con el resto de librerías 
# de las que depende este comando
```
> [!warning]
> Este comando ejecuta un which dentro, no es necesario hacerlo antes de llamarlo, solo indique el nombre del programa y asegúrese de que está disponible a través del path
>
> Además este comando, por ser usado dentro de [`exec_command`](#exec_command), **no copia la librería que carga las librerías dinámicas**. 
> 
> En caso de que no haya ejecutado el anterior comando se puede resolver ejecutando el comando:
> ```bash
> copy_library /lib64/ld-linux-x86-64.so.2 [ <dest_path> ]
> ```

### `copy_library`
Copia una librería y las librerías de las que depende.
```bash
copy_library <path_to_library> [ <dest_path> ]
# O usar which si sabemos que las librerías están en el PATH del entorno ($PATH)
# Verificar que solo devuelve un resultado, de otra manera el comando podrá 
# interpretar que la otra ruta es el dest_path
copy_library $(which <library>) [ <dest_path> ]
```
### `copy_link_file`
Copia un fichero manteniendo la estructura de enlaces en caso de tenerla. 
```bash
copy_link_file <path_to_file> [ <dest_path> ]
```
En caso de que en el destino no exista el directorio destino se generará la estructura de directorios oportuna:

```bash
$ tree /tmp/repo
/tmp/repo
|-- prueba
|-- test
|   |-- ln1 -> test.sh
|   |-- ln2 -> ../test/ln1
|   |-- ln3 -> /tmp/repo/test/ln2
|   `-- test.sh
`-- test_2
    `-- ln4 -> ../test/ln3

$ copy_link_file /tmp/repo/test_2/ln4 /tmp/prueba


```

### `cpo`
Copia los ficheros pasados por parámetros respetando la ruta.
```bash
cpo [ -p <prefix_path_dest> ] file1 [ file2 ... ]
```