# Práctica: Instalación de MariaDB y PHP en Ubuntu 24.04 (completando el LAMP)

| | |
|---|---|
| **Alumno** | Daviti Gvaramia |
| **Ciclo** | Sistemas Microinformáticos y Redes (SMR) |
| **Módulo** | Aplicaciones Web |
| **Fecha** | 22 de octubre de 2026 |

---

## Índice

1. [Preparación del sistema](#apartado-1-preparación-del-sistema)
2. [Instalación de MariaDB](#apartado-2-instalación-de-mariadb)
3. [Comprobación del funcionamiento](#apartado-3-comprobación-del-funcionamiento)
4. [Comandos principales](#apartado-4-comandos-principales)
5. [¿Qué pasa si MariaDB está parado?](#apartado-5-qué-pasa-si-mariadb-está-parado)
6. [Instalación de PHP](#apartado-6-instalación-de-php)
7. [Comprobación final](#apartado-7-comprobación-final)
8. [Conclusiones y problemas encontrados](#conclusiones-y-problemas-encontrados)

---

## Introducción: ¿qué es una base de datos y qué es el LAMP?

Una **base de datos** es un sitio donde un programa guarda su información de forma ordenada para poder encontrarla rápido. Se organiza en **tablas** (como hojas de cálculo), con **columnas** (qué datos se guardan) y **filas** (cada elemento guardado).

**WordPress** guarda en la base de datos todo lo que no son ficheros: entradas, páginas, comentarios, usuarios y la configuración del sitio. Cada vez que alguien visita la web, WordPress le pide los datos a la base de datos.

**MariaDB** es el programa que gestiona esa base de datos: la guarda, la protege y responde a las peticiones.

El **LAMP** es el conjunto de cuatro piezas que necesita WordPress:

| Pieza | Programa | Qué hace |
|---|---|---|
| **L** | Linux (Ubuntu 24.04) | Sistema operativo |
| **A** | Apache | Sirve la web al navegador |
| **M** | MariaDB | Guarda los datos |
| **P** | PHP | Ejecuta WordPress |

Ya tenía Linux y Apache de la práctica anterior. En esta práctica instalo las dos piezas que faltan: **MariaDB** y **PHP**.

---

## Apartado 1. Preparación del sistema

### 1.1. Sitios virtuales de Apache

```bash
sudo a2ensite 000-default.conf
sudo systemctl reload apache2
```

**Qué hace y por qué:** `a2ensite` activa un sitio virtual de Apache (en este caso el sitio por defecto, `000-default.conf`) y `systemctl reload apache2` hace que Apache vuelva a leer su configuración sin tener que pararse. Así dejo el servidor con un único sitio activo, el de por defecto, que es el que usará WordPress más adelante.

Salida relevante: Apache avisa de que hay que recargar la configuración para que el cambio se aplique, y por eso se ejecuta el `reload`.

![Captura 1](img/1.png)

*Captura 1: activación del sitio virtual por defecto (`a2ensite 000-default.conf`) y recarga de Apache con `systemctl reload apache2`.*

### 1.2. Actualizar el sistema y comprobar Apache

```bash
sudo apt update
sudo apt upgrade -y
systemctl is-active apache2
```

**Qué hace y por qué:** `apt update` refresca la lista de paquetes disponibles, `apt upgrade -y` instala las actualizaciones pendientes (la opción `-y` responde "sí" automáticamente) y `systemctl is-active apache2` pregunta si Apache está funcionando. Conviene tener el sistema al día antes de instalar software nuevo, y comprobar que Apache sigue vivo antes de añadirle piezas.

Salida relevante: la última línea responde `active`, así que Apache funciona correctamente.

![Captura 2](img/2.png)

*Captura 2: actualización de la lista de paquetes y de los paquetes del sistema, y comprobación de que Apache está activo (`active`).*

---

## Apartado 2. Instalación de MariaDB

```bash
sudo apt install mariadb-server -y
```

**Qué hace y por qué:** instala el servidor de base de datos MariaDB desde los repositorios oficiales de Ubuntu. Es la pieza **M** del LAMP y es donde WordPress guardará sus datos.

![Captura 3](img/3.png)

*Captura 3: comando de instalación del servidor MariaDB (`sudo apt install mariadb-server -y`).*

Después comprobé la versión instalada:

```bash
mariadb --version
```

Salida relevante:

```text
mariadb  Ver 15.1 Distrib 10.11.14-MariaDB, for debian-linux-gnu (x86_64) using  EditLine wrapper
```

**Qué hace y por qué:** muestra la versión del cliente de MariaDB. Sirve para confirmar que la instalación ha terminado bien y saber con qué versión (10.11) voy a trabajar.

![Captura 4](img/4.png)

*Captura 4: versión de MariaDB instalada, `10.11.14-MariaDB` (`mariadb --version`).*

---

## Apartado 3. Comprobación del funcionamiento

### 3.1. Estado del servicio

```bash
sudo systemctl status mariadb
systemctl is-enabled mariadb
```

**Qué hace y por qué:** `status` muestra si el servicio está funcionando ahora mismo y las últimas líneas de su registro; `is-enabled` indica si arrancará solo cada vez que se encienda el equipo. Ubuntu deja ambas cosas configuradas automáticamente al instalar.

Salida relevante:

- `status`: `active (running)`, con el mensaje "Taking your SQL requests now...", lo que indica que el servicio está funcionando y listo para recibir peticiones.
- `is-enabled`: `enabled`, es decir, arranca automáticamente con el sistema.

![Captura 5](img/5.png)

*Captura 5: estado del servicio MariaDB (`sudo systemctl status mariadb`): cargado y habilitado (`enabled`), `active (running)`, con su registro de arranque. Para salir de esta pantalla se pulsa `q`.*

![Captura 6](img/6.png)

*Captura 6: comprobación de que MariaDB arranca automáticamente con el sistema (`systemctl is-enabled mariadb` responde `enabled`).*

### 3.2. Entrar en la base de datos

```bash
sudo mariadb
```

**Qué hace y por qué:** abre el monitor de MariaDB (su consola). Si aparece el mensaje de bienvenida y el prompt cambia a `MariaDB [(none)]>`, significa que el servidor responde. Con eso queda comprobado que MariaDB funciona; después escribí `exit` para volver al terminal de Linux.

![Captura 7](img/7.png)

*Captura 7: entrada en el monitor de MariaDB (mensaje "Welcome to the MariaDB monitor", versión del servidor `10.11.14-MariaDB-0ubuntu0.24.04.1`) y salida con `exit`.*

---

## Apartado 4. Comandos principales

Estos son los comandos que se usan para manejar el servicio de MariaDB y lo que hace cada uno:

| Comando | Función | Explicación con mis palabras |
|---|---|---|
| `sudo systemctl start mariadb` | Arranca el servicio | Enciende MariaDB si está parado. |
| `sudo systemctl stop mariadb` | Detiene el servicio | Apaga MariaDB; mientras esté parado nadie puede conectarse a la base de datos. |
| `sudo systemctl restart mariadb` | Reinicia el servicio | Lo apaga y lo vuelve a encender de una vez; útil tras cambiar la configuración. |
| `sudo systemctl status mariadb` | Muestra su estado | Dice si está funcionando o parado y enseña las últimas líneas de su registro. |
| `sudo systemctl enable mariadb` | Lo arranca automáticamente al encender el equipo | Hace que MariaDB se inicie solo cada vez que se arranca el servidor, sin tener que hacerlo a mano. |

Los comandos `stop`, `start` y `status` se ven en acción en los apartados 3 y 5 de esta memoria.

---

## Apartado 5. ¿Qué pasa si MariaDB está parado?

Más adelante, cuando WordPress muestre el error *"Error al establecer una conexión con la base de datos"*, lo primero que hay que mirar es si MariaDB está funcionando. Para ver cómo se nota, detuve el servicio a propósito:

```bash
sudo systemctl stop mariadb
systemctl is-active mariadb
sudo mariadb
```

Salida relevante:

```text
inactive
ERROR 2002 (HY000): Can't connect to local server through socket '/run/mysqld/mysqld.sock' (2)
```

**Qué significa:** `inactive` indica que el servicio está parado, y el error 2002 quiere decir "no puedo conectar con MariaDB porque no está en marcha" (el *socket* es el canal por el que se comunica el cliente con el servidor, y no existe si el servidor está apagado).

![Captura 8](img/8.png)

*Captura 8: MariaDB detenido a propósito. `is-active` responde `inactive` y al intentar entrar aparece el `ERROR 2002 (HY000)` porque no se puede conectar con el servidor.*

Después lo volví a arrancar y comprobé que todo vuelve a la normalidad:

```bash
sudo systemctl start mariadb
systemctl is-active mariadb
```

![Captura 9](img/9.png)

*Captura 9: recuperación del servicio. Tras `sudo systemctl start mariadb`, `systemctl is-active mariadb` responde de nuevo `active`.*

### Pregunta de reflexión

> **Si WordPress no pudiera conectar con la base de datos, ¿qué dos comandos de este apartado usarías para saber si el problema es que MariaDB está parado?**

Usaría:

1. `systemctl is-active mariadb`: si responde `inactive`, el servicio está parado.
2. `sudo mariadb`: si MariaDB está parado, da el error `ERROR 2002 (HY000): Can't connect to local server through socket...`; si funciona, entra en el monitor.

Si el problema fuera ese, lo arrancaría con `sudo systemctl start mariadb` y volvería a comprobar con `systemctl is-active mariadb`.

---

## Apartado 6. Instalación de PHP

WordPress está hecho en PHP, así que instalo PHP y las piezas que WordPress necesita:

```bash
sudo apt install php libapache2-mod-php php-mysql php-curl php-gd php-mbstring php-xml php-zip php-intl -y
php -v
```

![Captura 10](img/10.png)

*Captura 10: instalación de PHP y de los módulos necesarios para WordPress (`php`, `libapache2-mod-php`, `php-mysql`, `php-curl`, `php-gd`, `php-mbstring`, `php-xml`, `php-zip` y `php-intl`).*

Para qué sirve cada paquete:

| Paquete | Función |
|---|---|
| `php` | El lenguaje PHP |
| `libapache2-mod-php` | Permite a Apache ejecutar PHP |
| `php-mysql` | Permite a PHP hablar con MariaDB (imprescindible) |
| `php-curl` | Conexiones a otros servidores (actualizaciones, plugins) |
| `php-gd` | Tratamiento de imágenes |
| `php-mbstring` | Manejo de textos con tildes y caracteres especiales |
| `php-xml` | Lectura y escritura de ficheros XML |
| `php-zip` | Instalación y actualización de plugins y temas |
| `php-intl` | Idiomas y formatos regionales |

Salida de `php -v`:

```text
PHP 8.3.6 (cli) (built: Oct  7 2026 18:06:38) (NTS)
```

![Captura 11](img/11.png)

*Captura 11: versión de PHP instalada, `PHP 8.3.6` (`php -v`).*

### Reinicio de Apache y comprobación de PHP

```bash
sudo systemctl restart apache2
sudo apache2ctl -M | grep -E "mpm|php"
```

**Qué hace y por qué:** reinicio Apache para que cargue el módulo de PHP, y con `apache2ctl -M` listo los módulos cargados filtrando los que contienen `mpm` o `php`. Deben aparecer `mpm_prefork_module` y `php_module`. En la página `info.php` de más abajo (captura 15) también se ve `mod_php` y `prefork` entre los módulos cargados de Apache.

### Comprobación de que PHP puede hablar con MariaDB

```bash
php -m | grep -i mysql
```

Salida:

```text
mysqli
mysqlnd
pdo_mysql
```

**Qué hace y por qué:** `php -m` lista los módulos de PHP y `grep -i mysql` se queda solo con los relacionados con MySQL/MariaDB. Aparecen `mysqli`, `mysqlnd` y `pdo_mysql`, que son las piezas que usa WordPress para conectarse a la base de datos.

![Captura 13](img/13.png)

*Captura 13: módulos de PHP para base de datos (`php -m | grep -i mysql`): `mysqli`, `mysqlnd` y `pdo_mysql`.*

### Comprobación de que Apache ejecuta PHP

```bash
echo '<?php phpinfo();' | sudo tee /var/www/html/info.php
```

**Qué hace y por qué:** crea un fichero de prueba `info.php` con una sola instrucción, `phpinfo()`, que muestra toda la información de PHP. Después abrí `http://IP_DE_TU_SERVIDOR/info.php` en el navegador: si Apache muestra la página de información de PHP en lugar de descargar o enseñar el código, es que Apache está ejecutando PHP correctamente.

![Captura 14](img/14.png)

*Captura 14: creación del fichero de prueba `/var/www/html/info.php` con el contenido `<?php phpinfo();`.*

![Captura 15](img/15.png)

*Captura 15: página `info.php` abierta en el navegador. Se ve la información de Apache (`Apache/2.4.58 (Ubuntu)`, usuario `www-data`, sitio virtual, módulos cargados como `mod_php` y `prefork`), lo que confirma que Apache ejecuta PHP.*

### Borrado del fichero de prueba (obligatorio)

```bash
sudo rm /var/www/html/info.php
ls /var/www/html
```

**Qué hace y por qué:** elimina el fichero de prueba y con `ls` compruebo que ya no está; solo quedan `404.html` e `index.html`. Es obligatorio borrarlo por seguridad (ver la pregunta de reflexión).

![Captura 16](img/16.png)

*Captura 16: borrado de `info.php` (`sudo rm`) y comprobación con `ls /var/www/html`, donde solo quedan `404.html` e `index.html`.*

### Preguntas de reflexión

> **¿Por qué es peligroso dejar `info.php` en un servidor real?**

Porque `phpinfo()` muestra mucha información interna del servidor: la versión exacta de PHP y de Apache, los módulos instalados, las rutas de los ficheros, la configuración y las variables de entorno. Cualquiera que abra esa página puede usar esos datos para buscar vulnerabilidades concretas de esas versiones y planear un ataque. Por eso un fichero de prueba como este se borra en cuanto se ha comprobado que PHP funciona.

> **¿Para qué sirve el paquete `php-mysql`?**

Sirve para que PHP pueda comunicarse con MariaDB/MySQL. Sin él, PHP no sabría "hablar" con la base de datos y WordPress no podría guardar ni leer sus datos, por eso es imprescindible. Es el que aporta módulos como `mysqli`, `mysqlnd` y `pdo_mysql`.

---

## Apartado 7. Comprobación final

```bash
systemctl is-active apache2 mariadb
php -v
```

**Qué hace y por qué:** comprueba de una vez que Apache y MariaDB están funcionando (`active` dos veces) y que PHP está instalado con la versión esperada. Es la comprobación de que el servidor está listo para instalar WordPress.

![Captura 17](img/17.png)

*Captura 17: comprobación final. `systemctl is-active apache2 mariadb` responde `active` dos veces y `php -v` muestra `PHP 8.3.6`.*

Estado final del LAMP:

| Pieza del LAMP | Programa | Estado |
|---|---|---|
| Linux | Ubuntu 24.04 | Instalado |
| Apache | Apache2 | Funcionando |
| MariaDB | MariaDB 10.11 | Funcionando |
| PHP | PHP 8.3 | Funcionando y preparado para MariaDB |

Tras esta comprobación se recomienda hacer una instantánea (*snapshot*) de la máquina virtual con el servidor listo, que será el punto de partida seguro para instalar WordPress.

---

## Conclusiones y problemas encontrados

### Conclusiones

- He completado el LAMP en Ubuntu 24.04: a Linux y Apache les he añadido **MariaDB 10.11** y **PHP 8.3**, y he comprobado que los cuatro componentes funcionan.
- Entiendo para qué sirve cada pieza: Apache sirve la web, PHP ejecuta WordPress y MariaDB guarda sus datos. El paquete `php-mysql` es el que conecta PHP con MariaDB.
- He visto cómo se nota que MariaDB está parado (`inactive` y el error 2002), que será lo primero que habrá que mirar si WordPress da el error de conexión con la base de datos.
- Aprendí que ficheros de prueba como `info.php` hay que borrarlos tras usarlos, porque exponen información sensible del servidor.

### Problemas encontrados

- Al ejecutar `sudo apt upgrade -y` se quedaron 8 paquetes sin actualizar (`0 upgraded ... 8 not upgraded`): Ubuntu los aplaza de forma escalonada ("deferred due to phasing"). No es un error y no impide continuar con la práctica.
- El enunciado del Apartado 1 habla de "deshabilitar" los sitios virtuales de la práctica anterior, pero el comando indicado es `a2ensite 000-default.conf`, que lo que hace es **habilitar** el sitio por defecto. Ejecuté el comando tal y como figura en la práctica.
