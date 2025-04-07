# semp-p5-dios-v2
Este programa facilitará mucho la practica 5 de sistemas empotrados

## Uso
Primero ejecutemos la configuración para integrar valores al entorno
```bash
source ./setup
```
Una vez configurado dispondrémos en todo el sistema de los comandos. El más interesante es `exec_command` que se encargará de toda la tarea automatizada.

Pero también se disponen de los siguientes comandos:
### `copy_command`
Copia un comando, los enlaces a los que apunta y librerías de las que depende
```bash
copy_command <command> [ <dest_path> ]
```
### `copy_libraries`
Copia una librería y las librerías de las que depende
```bash
copy_libraries <path_to_library> [ <dest_path> ]
```
### `copy_link_file`
Copia un fichero manteniendo la estructura de enlaces en caso de tenerla
```bash
copy_link_file <path_to_file> [ <dest_path> ]
```
### `cpo`
Copia los ficheros pasados por parámetros respetando la ruta.
```bash
cpo [-p <prefix_path_dest> ] file1 [file2 ...]
```