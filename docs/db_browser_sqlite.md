# 2. DB Browser for SQLite

Siempre que desarrollamos aplicaciones que acceden a una base de datos es recomendable disponer de un visor / editor adecuado al sistema gestor de bases de datos con el que estamos trabajando para poder realizar comprobaciones. Con la herramienta DB Browser for SQLite podemos crear, modificar y consultar la estructura de una base de datos.

Se puede descargar desde el siguiente enlace

Para Linux puedes descargar la AppImage que no necesita instalación desde:

(https://download.sqlitebrowser.org/DB.Browser.for.SQLite-v3.13.1-x86.64-v2.AppImage)

Una vez descargada, para poder ejecutarla con doble clic, sigue estos pasos:

Haz clic con el botón derecho y entra en Propiedades.

Entra en la pestaña Permisos.

Marca la casilla Permite la ejecución del fichero como programa.

A continuación se describen los pasos a seguir para crear una base de datos llamada fruteria.db con una tabla llamada productos con la siguiente estructura:

Crear la base de datos:

Crear la tabla y los campos

Añadir información a la tabla:

| Campo | Tipo de Dato | Descripción |
| --- | --- | --- |
| id | INTEGER (PK, AUTOINCREMENT) | Identificador único. |
| nombre | CHAR(15) | Nombre. |
| precio | REAL | Precio (con decimales). |
| procedencia | CHAR(3) | Código de país de procedencia. |

![Imagen 1](image9.png)

![Imagen 2](image6.png)

![Imagen 3](image12.png)

![Imagen 4](image5.png)

![Imagen 5](image7.png)

![Imagen 6](image4.png)

![Imagen 7](image11.png)

![Imagen 8](image3.png)

![Imagen 9](image1.png)

![Imagen 10](image2.png)

![Imagen 11](image10.png)

![Imagen 12](image8.png)

