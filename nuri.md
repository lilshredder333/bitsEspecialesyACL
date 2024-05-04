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
Vemos que necesitamos una "dependencia" o algo asi llamada gcc para poder compliarlo. Como ej1 no es admin no puede instalarla. 

```shell
#me meto en mi usuario para instalarla y vuelvo a probar a compliar desde ej1

nuri@nuriserver:~$ sudo apt install gcc
 
#una vez instalado todo vuelvo a cambiar al user ej1 e intento compliar de nuevo

ej1@nuriserver:~$ gcc delete_dir.c -o delete_dir
delete_dir.c: In function ‘main’:
delete_dir.c:11:13: warning: implicit declaration of function ‘rmdir’ [-Wimplicit-function-declaration]
   11 |         if (rmdir(directorio) == 0)
      |             ^~~~~
ej1@nuriserver:~$ ls -l
total 24
-rwxr-xr-x 1 ej1 users 16264 abr 30 17:32 delete_dir
-rwxr--r-- 1 ej1 users   398 abr 29 13:33 delete_dir.c
drwxr-xr-x 2 ej1 users  4096 abr 30 17:27 esb
ej1@nuriserver:~$ ls
delete_dir  delete_dir.c  esb
ej1@nuriserver:~$ ./delete_dir
Intentando borrar el directorio '/home/ej1/esb' ...
Directorio eliminado con éxito.

# volvemos a crear esb


```

## Descargamos script_delete_dir y comprobamos que tiene la misma funcionalidad que delete_dir

```shell
ej1@nuriserver:~$ curl  https://raw.githubusercontent.com/s-rom/SlidesUsuariosLinux/master/Ejercicio_ACL_SETUID/delete_dir.sh > script_delete_dir
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--    100   190  100   190    0     0    554      0 --:--:-- --:--:-- --:--:--   555
ej1@nuriserver:~$ gcc script_delete_dir -o script_delete_dir.c
script_delete_dir: file not recognized: file format not recognized
collect2: error: ld returned 1 exit status

# puede que script_delete_dir no necesite ser compilado, vamos a intentar ejecutarlo sin mas
ej1@nuriserver:~$ ./script_delete_dir
bash: ./script_delete_dir: Permission denied

# vemos que nos faltan permisos, de ejecucion probablemente. Se los proporciono desde mi usuario

nuri@nuriserver:~$ u+x home/ej1/script_delete_dir
Command 'u+x' not found, did you mean:
  command 'upx' from snap upx (v0.2.3)
  command 'uux' from deb uucp (1.07-27build3)
See 'snap info <snapname>' for additional versions.
nuri@nuriserver:~$ chmod +x /home/ej1/script_delete_dir
chmod: cannot access '/home/ej1/script_delete_dir': Permission denied

#casi xd

nuri@nuriserver:~$ sudo chmod +x /home/ej1/script_delete_dir
[sudo] password for nuri:
nuri@nuriserver:~$ su ej1
Password:
ej1@nuriserver:/home/nuri$ cd ../ej1
ej1@nuriserver:~$ ./script_delete_dir
Directorio eliminado '/home/ej1/esb' eliminado!
ej1@nuriserver:~$

# efectivamente vemos que era por tema de permisos de ejecucion del script. 

```

## Intentamo instalar comandos delete_dir y script_delete_dir en /usr/bin

```shell
ej1@nuriserver:~$ cp script_delete_dir /usr/bin/
cp: cannot create regular file '/usr/bin/script_delete_dir': Permission denied
ej1@nuriserver:~$

# vemos que al intentar copiar script_delete_dir nos da error, igual puede ser porque no esta compilado; vamos a ver con delete_script. 

#delete_dir no compilado
ej1@nuriserver:~$ cp delete_dir /usr/bin/
cp: cannot create regular file '/usr/bin/delete_dir': Permission denied
ej1@nuriserver:~$

#delete_dir compilado
ej1@nuriserver:~$ cp delete_dir.c /usr/bin/
cp: cannot create regular file '/usr/bin/delete_dir.c': Permission denied
ej1@nuriserver:~$

#Descartamos que sea por motivos de compilacion.Como nos aparece permission denied podría ser que no nos dejara con el usuario ej1 porque no es sudoer y no podemos "forzar" la copia. Vamos a intentarlo en mi usuario.

nuri@nuriserver:/home$ sudo cp /home/ej1/delete_dir /usr/bin/
nuri@nuriserver:/home$ sudo cp /home/ej1/script_delete_dir /usr/bin/
nuri@nuriserver:/home$

# comprobacion de /usr/bin/ para ver si se han copiado correctamente

nuri@nuriserver:/usr/bin$ ls -l | grep delete_dir
-rwxr-xr-x 1 root root       16264 may  4 16:59 delete_dir
-rwxr-xr-x 1 root root         190 may  4 17:00 script_delete_dir
nuri@nuriserver:/usr/bin$

```

## Añadido de excepción acl para que el usuario ej1 pueda copiar ficheros en /usr/bin/ únicamente. 