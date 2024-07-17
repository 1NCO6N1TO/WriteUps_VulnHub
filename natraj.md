- Tags: #wrappers #LFI #sshInject 
____
se comenzo la maquina con un escaneo de nmap en el cual se puede apreciar que hay dos puertos abiertos los cuales son el 22 - 80 (SSH - HTTP) se le aplico los script basicos de reconocimiento de namp.

accedemos a la pagina web para investigar pero nada interesante, por lo que aplicamos fuzzing con gobuster el cual nos revela un directorio llamado **/console/** en el cual se encuentra un archivo file.php el cual no podemos ver el contenido por lo que usaremos wrappers para permitir que nos muestre el contenido pero el base64 y posteriormente lo decodificaremos.
____
![](attachment/37f00b4208b706c4ac27a7873ceb066c.png)
___
una vez utilizamos el wrapper nos aparece lo siguiente
___
![](attachment/c8fbc0b31ded7208f5553067775fc885.png)
___
se puede observar ahí una cadena en base64 generada por el wrapper, una vez decodificada nos aparece lo siguiente.
___
![](attachment/b36c407651d61a36201e05b47baddb60.png)
_____
ese es el contenido del archivo, por lo que con este fichero podemos ejecutar comando pero aun lo tenemos que escalar a un RCE. por lo que exploraremos los logs para ver cuales son accesibles.

el log accesible es el de SSH en la siguiente ruta **/var/log/auth.log**
___
![](attachment/530b6f8075dd1dc58935f4625e27094d.png)
____
por lo que precederemos a usar SSH para envenenar el log, ahora encontraremos un error de ssh puesto que la vulnerabilidad de inyectar comandos por medio del mismo fue parcheada por lo que nos presentara el siguiente error.
____
![](attachment/e9d4d58c671547c1f8d4370cb5760079.png)
____
utilizaremos otra vía para llegar a inyectar el comando, utilizando **msfconsole** podremos llevar acabo la tarea.

utilizando el siguiente comando: `use auxiliary/scanner/ssh/ssh_login` tendremos variedad de opciones, pero solo configuraremos el nombre, contraseña y claro direccion ip de la victima. todo esto no permitirá inyectar el comando y que así podamos ejecutar comandos remotos.
___
![](attachment/e999064c3b0ebea49b7568ba52cfa808.png)
____
ahí tenemos todo configurado, en el apartado de usuario es donde colocaremos nuestra carga util.

una vez configurado todo debemos correrlo con el comando **run**. 

si actualizamos la ruta del log y aplicamos un comando vemos si funciona.
____
![](attachment/84d3fbac518f9e8bbf85943ab59963c3.png)
___
aplicamos el comando whaomi y tenemos respuesta, por lo que ahora debemos entablar una reverse shell y asi obtener acceso a la maquina.

buscamos formas de escalar privilegios, con el comando find \-perm -4000 2>/dev/null no obtenemos resultados, por lo que buscamos que archivos tienen permisos de escritura en el usuario www-data.

para eso usamos el siguiente comando `find / -writable 2>/dev/null` pero debemos filtrar para ver los resultados que queremos, usando **grep -vE** indicamos que no queremos ver lo que le pasemos como argumento, ademas de permitirnos colocar mas de un argumento.
___
![](attachment/8202522ccfb910ab4aeceea8b99bef50.png)
___
nos indica que tenemos permisos de escrituras sobre el archivo **apache2.conf** y si vemos quien esta ejecutando el fichero podemos ver que el usuario root.

de esto nos podemos aprovechar al cambiar el usuario que esta levantando el servicio de apache y si recordamos como accedimos dependiendo de que usuario levante el servicio de apache una vez contaminemos el log y accedamos nuevamente accederemos como el usuario que ha levantado el servicio nuevamente con una excepción de que no nos permite colocar root y ademas tendremos que reiniciar la maquina puesto que no hay un tarea cron que reinicie el servicio automáticamente. 
____
![](attachment/0d34afd5bfcd49117e8248072fb3652d.png)
___
en este apartado con variables de entorno cambiaremos el usuario, el cual debemos conocer con anticipación.
____
![](attachment/ee7273e82262ba4108b763b81a12fd80.png)
____
usaremos el usuario **mahakal**.
___
![](attachment/4e49aaab2efb67a6915461da36312a46.png)
___
guardamos y reiniciamos y vemos que pasara.
____
![](attachment/b2c47d6b65d61a070947a6abf589b083.png)
____
y en efecto iniciamos como el usuario mahakal, ahora vemos nuevas forma de escalar privilegios.
___
![](attachment/87cd02ed05e7187d1b029d79341657e5.png)
____
vemos que podemos ejecutar nmap y ejecutando los siguientes comando obtendremos root.
____
![](attachment/9ac1cf230cd3f7a34b7bef6b0fbcdc21.png)
____
fin...
