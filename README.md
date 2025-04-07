# semp-p5-dios-v2
Este programa facilitará mucho la practica 5 de sistemas empotrados

## Uso
Primero ejecutemos la configuración para integrar valores al entorno. Puede ejecutarse desde rutas absolutas
```bash
source ./setup
source /ruta/hasta/el/fichero/setup
```
Una vez configurado dispondremos en todo el sistema de los comandos. El más interesante es `exec_command` que se encargará de toda la tarea automatizada. Pero también se disponen de otros comandos útiles.
### `exec_command`
```bash
exec_command [dest_path]
```
Genera una estructura básica de ficheros a partir del actual para una ruta concreta, en caso de no ser especificada se usará `/opt-A`.

El comando copiará los directorios `/dev` y `/etc` si detecta que alguno de estos no está en el destino, se deberá tener esto en cuenta si se desea repetir todo el proceso.

De manera similar con la estructura de directorios que **no** se rellenará inicialmente (`/bin` `/lib` `/lib64` `/proc` `/root` `/sys` `/tmp` `/sbin` `/usr/bin` `/usr/sbin` `/var/state/dhcp` `/var/tmp` `/var/log` `/dev/shm`), en este caso la condición para volver a ejecutar la creación y configuración es la existencia del directorio `/root` en el destino.

Una vez establecido lo básico del sistema de ficheros se empezará a copiar los comandos respetando sus enlaces, copiando por tanto tanto enlaces como los ficheros a los que apuntan y sus directorios si no existen, y las librerías dinámicas de las que dependen asi mismo también se aplica a las dependencias de las propias librerías.

> [!warning]
> En caso de que ya exista el ejecutable en el destino se interpretará como que ya ha sido copiado y se pasará al siguiente comando.


> [!note]
> Estos comandos incluyen los comandos del sistema y usuario juntos con unos mínimos para la ejecución de /sbin/init

También copiará librerías dinámicas imprescindibles para el nuevo sistema de ficheros, puesto que afecta directamente entre otras a la información de los usuarios, y una librería imprescindible para los comandos con dependencias de librerías dinámicas `/lib64/ld-linux-x86-64.so.2`. Se verifica primero que no existan en el destino antes de iniciar el proceso.

> [!warning]
> En caso de que se produzca cualquier problema y se desee ejecutar todo de nuevo se recomienda encarecidamente borrar antes de ejecutar la parte afectada en el destino o todo el contenido del destino.
> 
> Ya que debido a las comprobaciones automáticas este proceso se saltará pasos.
> 
> O, por el contrario (**no recomendado**), modificar el código para ejecutar siempre todos los pasos

### `copy_command`
Copia un comando, los enlaces a los que apunta, las librerías de las que depende y las librerías de las que cada una de estas librerías depende.
```bash
copy_command <command> [ <dest_path> ]
```
### `copy_libraries`
Copia una librería y las librerías de las que depende.
```bash
copy_libraries <path_to_library> [ <dest_path> ]
```
### `copy_link_file`
Copia un fichero manteniendo la estructura de enlaces en caso de tenerla. En caso de que en el destino no exista el destino se generará la estructura de directorios oportuna.
```bash
copy_link_file <path_to_file> [ <dest_path> ]
```
### `cpo`
Copia los ficheros pasados por parámetros respetando la ruta.
```bash
cpo [-p <prefix_path_dest> ] file1 [file2 ...]
```