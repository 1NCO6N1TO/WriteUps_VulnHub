- Tags: #pivoting 
_____
seguimos con la maquina analizando el reporte de nmap para ver que podemos explotar.
___
![](attachment/b5eb978458e0a54eda6750bf3554be0c.png)
_____
vemos que tenemos 5 puertos abiertos, al igual que vemos versiones, también tenemos un servicio de samba activo. 

empezaremos empleando smbmap para escanear el servicio de samba para ver si hay recursos en los que tengamos permiso de lectura.
____
![](attachment/bdeccc4f1f2ef99f0b1d343128986b58.png)
____
vemos que tenemos un directorio **anonymous** el cual tenemos permisos de lectura, veremos que tiene dentro utilizando el comando `proxychains smbmap -H <direccion_ip> -r <direcctorio>`  
___
![](attachment/222c6169912e4247939bfafacb554b04.png)
___
vemos que tenemos otro directorio llamado **backups** el cual examinaremos.
___
![](attachment/2a16e485159ef346950bae2548330c5f.png)
_____
ahora tenemos un archivo de texto el cual descargaremos y veremos su contenido.
___
![](attachment/6c6fc3613b373f9ce0a33c95a7af23b9.png)
___
vemos que la ruta donde se encuentra el archivo **shadow.bak**.

contexto de shadow.bak:

El archivo `shadow.bak` es una copia de seguridad del archivo `/etc/shadow`, que es fundamental para la seguridad de un sistema Unix o Linux. Este archivo contiene la información de las contraseñas de los usuarios del sistema, en un formato cifrado.

es por ello que conocer su ruta podría suponer una vulnerabilidad puesto que si encontramos la forma de llegar al mismo podríamos tratar de descifrar su contenido.

si vemos nuevamente el contenido del reporte de nmap veremos que tenemos el puerto 21 (FTP) con una versión algo vieja, en concreto es la versión **proFTP 1.3.5** la cual al buscarla con searchsploit nos muestra lo siguiente.
____
![](attachment/792345b8c3cfd0ff102fd3490e164e2a.png)
___
al parecer tiene una vulnerabilidad que nos permite copiar archivos desde una ruta conocida, hasta otra ruta del sistema. 

por lo que anteriormente la ruta que vimos del archivo shadow.bak, trataremos de copiar el archivo a la carpeta o ruta que nos comparte el servicio de ftp, pero para eso necesitamos su ruta.
____
![](attachment/0149dc367bf34a9579bb49999fed71e4.png)
____
se observa la ruta de la carpeta anonymous, ya teniendo esto podemos proceder y ver si funciona.
___
![](attachment/6b5683db24916021217c711ebcf7d971.png)
usando john y un diccionario para descifrar la contraseña obtenemos lo siguiente.

son credenciales que usaremos para acceder por ssh.

una vez accedemos por ssh debemos buscar formas para escalar privilegios.

aplicando comando como `find \-perm -4000 2>/dev/null` para listar binarios y `cut -d: -f1 /etc/passwd` para listar los usuarios del sistema. 

en este caso en particular veremos que puertos se encuentran abiertos puesto que puede que hayan puerto que no podamos ver a través de nuestro tunel con proxychains, por lo que aplicamos el comando `ss -nltp`.
____
![](attachment/1b8b20d157a52e54dcd935d3d60cd4e2.png)
____
vemos que tenemos un puerto **8080** abierto pero desde nuestra maquina atacante no es accesible por lo que utilizaremos **SSH** para aplicar un **Local Port Forwarding** y el puerto 8080 de la maquina victima traerlo a nuestra maquina y así poder ver su contenido.

por lo aplicamos el siguiente comando con ssh:

```shell
proxychains ssh aeolus@192.168.20.4 -L 8080:127.0.0.1:8080
```

una vez aplicado el comando ya podremos acceder al puerto 8080 por medio de nuestro local host.
____
![](attachment/f00c1d10fafe886ae8bd6bbf808c1bf8.png)
____
vemos que tenemos accesibilidad al puerto, recordemos que tenemos credenciales por lo que probaremos a ver si nos autentica.
____
![](attachment/718961bf0a6dbceb80f78275bd29f3bf.png)
____
estamos dentro, por lo que ahora debemos buscar vulnerabilidades asociadas a **libreNMS** utilizando searchsploit.
____
![](attachment/9fe74e5b69ac7714f66e0eb004a894a9.png)
____
### Forma local
nos encontramos con una vulnerabilidad que nos permite ejecutar codigo, y nos da instrucciones ademas del payload a utilizar, lo pegamos y establecemos nuestra ip y puerto.

esta es una forma de cambiar de usuario a uno con mas privilegios. y es la forma local por asi decirlo de hacerlo.

hacemos referencia a local porque la reverse shell es enviada a la misma maquina symfonos 2 y no a nuestra maquina de atacante.
_____
![](attachment/8f6774718ed57a5337403b436fb8b6fb.png)
_____
antes eramos el usuario aeolus y ahora somos el usuario cronus. 

### forma con socat
_____
con esta forma redirigiremos el flujo nos mande la maquina symfonos 2 a la symfonos 1, pensemos que la maquina symfonos 1 nos hará de puente nuevamente para que ese trafico llegue a nuestra maquina de atacante, a continuación un diagrama que lo explica.
_____
![](attachment/86885194d472f90a14b2d18a8386e450.png)
_____
![](attachment/c842deab6b0798ea1a48cf471d3eafdf.png)
_____
y es así como manejamos los paylodad si lo queremos enviar desde una 3era maquina a nuestra maquina atacante.

ahora a elevar privilegios, si aplicamos un `sudo -l` vemos que podemos ejecutar mysql como root.
____
![](attachment/b5ad99f3fd96a333a2bec8ab0f1105a2.png)
____
por lo que buscamos como elevar privilegios y nos aparece el siguiente comando.

```shell
sudo mysql -e '\! /bin/sh'
```

de esa forma ganaremos root en la maquina synfonos 2.
____
![](attachment/f966c355c85e3829ff0b65f7d1b7bfd4.png)
