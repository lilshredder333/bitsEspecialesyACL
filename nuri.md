# Ejercicio bits especiales y ACL Nuri

## Curl script 
``` shell 
curl https://raw.githubusercontent.com/s-rom/SlidesUsuariosLinux/master/Ejercicio_ACL_SETUID/create_user.sh > ejercicioBitsEspecialess.sh

```
## Añadir permisos de ejecucion al archivo

``` shell
chmod u+x ejercicioBitsEspecialess.sh
```

## Ejecutar el script

``` shell
./ejercicioBitsEspecialess.sh
```
Respuesta
``` shell 
Creating user ej1
useradd: Permission denied.
useradd: cannot lock /etc/passwd; try again later.
User ej1 was not created
```
Esto me pasa por tonta jeje, volvemos a ejecutar el script con permisos sudo


``` shell
nuri@nuriserver:~$ sudo ./create_user.sh
[sudo] password for nuri:
Creating user ej1
User ej1 (password:123) created successfully
nuri@nuriserver:~$
```

## Cambiamos al usuario ej1
Nos pide que introduzcamos la contraseña y ya estamos en usuario ej1.

``` shell
nuri@nuriserver:~$ su ej1
Password:
ej1@nuriserver:/home/nuri$
```

## Intento eliminar el archivo esb
```shell
ej1@nuriserver:/home/nuri$ rm esb
rm: cannot remove 'esb': Permission denied
ej1@nuriserver:/home/nuri$ ls -l
ls: cannot open directory '.': Permission denied
ej1@nuriserver:/home/nuri$ sudo ls -l
[sudo] password for ej1:
ej1 is not in the sudoers file.  This incident will be reported.
```

Vemos que no podemos eliminar la carpeta ya que el usuario no es el propietario de esta y ademas no esta en sudoers.

## Creo la carpeta como ej1 y cambio a mi user

```shell
ej1@nuriserver:~$ mkdir esb
ej1@nuriserver:~$ ls -l
total 4
drwxr-xr-x 2 ej1 users 4096 abr 29 13:25 esb
ej1@nuriserver:~$ exit
exit 

#desde mi usuario
nuri@nuriserver:~$ rmdir /home/ej1/esb
rmdir: failed to remove '/home/ej1/esb': Permission denied

#de primeras me deniega eliminarlo ya que no soy la propietaria de la carpeta, pero con sudo nos dejará eliminarla: 

nuri@nuriserver:~$ sudo rmdir /home/ej1/esb
nuri@nuriserver:~$

nuri@nuriserver:~$ su ej1
Password:
ej1@nuriserver:/home/nuri$ cd ../home/ej1
ej1@nuriserver:~$ ls -l
total 0
ej1@nuriserver:~$
```
Como podemos comprobar, hemos sido capaces de eliminar el archivo. 

## Descargamos delete_dir.c con curl

```shell
ej1@nuriserver:~$ ls -l
total 4
-rw-r--r-- 1 ej1 users 398 abr 29 13:33 delete_dir.c
ej1@nuriserver:~$ gcc delete_dir.c -o delete_dir
Command 'gcc' not found, but can be installed with:
apt install gcc
Please ask your administrator.
```
Vemos que necesitamos una "dependencia o algo asi llamada gcc para poder compliarlo. Como ej1 no es admin no puede instalarla. 