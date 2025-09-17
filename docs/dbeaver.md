# DBeaver

**DBeaver** es una herramienta gráfica y gratuita que permite gestionar múltiples bases de datos de forma visual. Algunas de las acciones que podemos realizar con esta herramienta son las siguientes:

- Explorar la estructura de la base de datos (tablas, vistas, claves, relaciones…).

- Consultar datos.

- Modificar tablas, añadir registros o ejecutar scripts SQL sin salir del proyecto.

- Probar consultas antes de implementarlas en el programa.

Los siguientes pasos ilustran como configurar esta herramienta para conectarnos a la BD de ejemplo **plantas.db** que ya tenemos copiada a la carpeta **resources** de nuestro proyecto.

![Imagen 1](img/conexiones_01.png)


Los pasos para conectarse a la BD  Factura.sqlite, disponible en la sección de recursos de Aules, son los siguientes:


**1. Abre DBeaver**{.azul}

Inicia el programa **DBeaver**. Aparecerá la ventana principal con el panel lateral de conexiones.

![ref](img/dbeaver0.jpg)

Haz clic en el botón **"Nueva conexión"** (ícono de enchufe) o ve al menú `Archivo > Nueva conexión`.

![ref](img/dbeaver2.jpg)

---


**2. Selecciona el tipo de base de datos**{.azul}

En la ventana de selección, elige **SQlite** y pulsa **Siguiente**.

![ref](img/dbeaver3.jpg)


---

**3. Introduce la ruta donde se encuentra la BD**{.azul}


![ref](img/dbeaver4.jpg)


---

**4. Prueba la conexión**{.azul}

Haz clic en **"Probar conexión"**. Si todo está correcto, verás un mensaje de éxito.  
Si DBeaver necesita un controlador (driver), te lo ofrecerá para descargar automáticamente.


![ref](img/dbeaver5.jpg)


---

**5. Finaliza y explora**{.azul}

Haz clic en **"Finalizar"**. La nueva conexión aparecerá en el panel lateral izquierdo.  
Desde allí puedes:

- Ver tablas, vistas, funciones y procedimientos
- Ejecutar sentencias SQL
- Consultar y modificar registros
- Exportar datos en distintos formatos

![ref](img/dbeaver6.jpg)

