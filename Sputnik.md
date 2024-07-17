____
se comienza como siempre, realizando escaneo de puertos con nmap.

se nos presentan 3 puerto abiertos los cuales son los siguiente:
____
![](attachment/dba3ee95e121484b3c3b0f27f6a8c42a.png)
____
vemos que ya de primera se nos reporta que hay un repositorio activo en la ruta indicada, por lo que exploramos y con el comando `wget -r <url>` descargamos todo el contenido y usando `git log --oneline` podemos ver todo el historial de commits, y se nos presentan los siguientes commits:
___
![](attachment/945ae2c514ac50a47c0305420cd90094.png)
___
exploramos todos, pero el marcado tiene algo interesante y es que si aplicamos el siguiente comando `git ls-tree <id_del_commit>` podremos ver un secret.
____
![](attachment/ea6562b92c7c6cd02abd9f984437ba3d.png)
___
pero si queremos acceder al mismo nos dará un error.
_____
![](attachment/fb1e625164ac4bd79885ef306298f0d9.png)
___
por lo que debemos explorar diferentes opciones, si nos vamos a la ruta donde se encuentra el .git, veremos que podemos navegar por directorios.
___
![](attachment/1822b0c206a14a5214d3a4e7a6f5a206.png)
____
si entramos al directorio logs, podemos encontrar lo siguiente.
___
![](attachment/614811f9ae54ab9a6429dc003d124295.png)
____
lo que nos indica que el proyecto real es el que se encuentra en ese repositorio por que empleando git clone, podremos traerlo a nuestra area de trabajo e investigar los logs para poder comparar y ver si son los mismos.
____
![](attachment/fffdb91a3bedfaf92ff86a58cb19815f.png)
____
los commits del proyecto son lo siguientes por lo que veremos el primer commit que nos indica que se creo un nuevo archivo, no son el mismo commit pero veamos la estructura del mismo.
___
![](attachment/ed7ceebb7754ca3ccea127e264e6fb03.png)
___
el identificador del secret si que es el mismo que el anterior por lo que si lo abrimos nos aparece lo siguiente:

**sputnik:ameer_says_thank_you_and_good_job**

son credenciales al parecer, si nos dirigimos al puerto 61337 veremos que tenemos un panel de login y si ingresamos las credenciales veremos que sucede.
____
![](attachment/cd588046de714e7f976a672560c50a2c.png)
____
nos logueamos como administrador, por lo que ahora tenemos que buscar la forma de ganar acceso a la maquina, por medio de esta web.

si buscamos el nombre de la web y le colocamos exploit nos aparecerán paginas de github explicando la vulnerabilidad que esta pagina presenta.

se nos dice que debemos cargar un archivo que contiene una carga util ya diseñada, y que el archivo debe de tener la extensión **.spl** también nos indica que al momento de cargar el archivo ya debemos estar a la escucha por medio de netcat.

ya después que se carga el archivo este se ejecuta automáticamente, dándonos acceso a la maquina victima.
____
![](attachment/d79d19e54a35205d8f90f3c8f206bc50.png)
____
una vez dentro de la maquina victima, buscaremos el medio para escalar privilegios. si aplicamos sudo -l veremos que nos solicita contraseña por que probaremos con contraseña antes encontrada. 
____
![](attachment/8666552cc8286823f7e8c251d8ecc8ff.png)
___
funciona, y nos dice que podemos ejecutar el **ed** para ver como podemos llegar a escalar privilegios por medio del mismo buscaremos en la pagina https://gtfobins.github.io/ y vemos que nos dice.
____
![](attachment/8542fa1cd7057ec674eb2f1eb590d0d7.png)
____
ejecutando ese simple comando podemos llegar a ser root, así que lo intentaremos.
___
![](attachment/687ea6cc202ab28ff6a90b2e477621a5.png)
____
tenemos root, damos por concluida la maquina.
____
![](attachment/29e9418f20ac95d24db217120a959b27.png)

