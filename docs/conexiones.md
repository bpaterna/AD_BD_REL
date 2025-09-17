
# Conexión a un SGBD


De todas las formas posibles de interactuar con una base de datos, nos vamos a centrar en el uso de **conectores**, porque son la forma más directa y habitual de acceder a la base de datos desde un lenguaje de programación, como Kotlin, que es el que estamos utilizando en este módulo.

En la introducción ya vimos que un **conector** (también llamado driver) es una librería software que permite que una aplicación se comunique con un gestor de base de datos (SGBD). Actúa como un puente entre nuestro código y la base de datos, traduciendo las instrucciones SQL a un lenguaje que el gestor puede entender y viceversa. Sin un conector, tu aplicación no podría comunicarse con la base de datos.

- En herramientas gráficas como **DBeaver**, los drivers se gestionan automáticamente.
- En **proyectos en código**, se añaden como dependencia (por ejemplo, en Maven, Gradle, pip…).


Una base de datos puede ser accedida desde diferentes orígenes o herramientas, siempre que tengamos:

- Las credenciales de acceso (usuario y contraseña)
- El host/servidor donde se encuentra la base de datos
- El motor de base de datos (PostgreSQL, MySQL, SQLite, etc.)
- Los puertos habilitados y los permisos correctos



**Principales formas de conectarse a una base de datos**{.azul}

| Medio de conexión                         | Descripción                                                                 |
|-------------------------------------------|-----------------------------------------------------------------------------|
| Aplicaciones de escritorio             | Herramientas gráficas como **DBeaver**, **pgAdmin**, **MySQL Workbench**, **DB Browser for SQLite**. Permiten explorar, consultar y administrar BD de forma visual. |
| Aplicaciones desarrolladas en código   | Programas en **Kotlin**, **Java**, **Python**, **C#**, etc., mediante **conectores**{.verde} como **JDBC**, **psycopg2**, **ODBC**, etc. para acceder a BD desde código. |
| Línea de comandos                      | Clientes como `psql` (PostgreSQL), `mysql`, `sqlite3`. Permiten ejecutar comandos SQL directamente desde terminal. |
| Aplicaciones web                        | Sitios web que acceden a BD desde el backend (por ejemplo, en Spring Boot, Node.js, Django, etc.). |
| APIs REST o servidores intermedios     | Servicios web que conectan la BD con otras aplicaciones, actuando como puente o capa de seguridad. |
| Aplicaciones móviles                   | Apps Android/iOS que acceden a BD locales (como **SQLite**) o remotas (vía **Firebase**, API REST, etc.). |
| Herramientas de integración de datos   | Software como **Talend**, **Pentaho**, **Apache Nifi** para migrar, transformar o sincronizar datos entre sistemas. |


!!!Tip ""
    A continuación se describe cómo trabajar con BD relacionales desde una aplicación desarrollada en **Kotlin**.


Una aplicación (escrita en Kotlin, Java u otro lenguaje) puede leer, insertar o modificar información almacenada en una base de datos relacional (BDR) si previamente se ha conecta conectarse al gestor de base de datos (PostgreSQL, MySQL, SQLite…). Una vez establecida correctamente la conexión podrá:

- Enviar consultas SQL (SELECT, INSERT, UPDATE, DELETE…)

- Recibir y procesar resultados (ResultSet, listas de objetos…)

- Cerrar correctamente los recursos utilizados

**JDBC** (Java Database Connectivity) es la API básica de Java (conector) para conectarse a bases de datos relacionales.

**Sintaxis:**

    jdbc:<gestor>://<host>:<puerto>/<nombre_base_datos>

Gestor de Base de Datos|	URL de conexión
-----------------------|---------------------
PostgreSQL|	jdbc:postgresql://localhost:5432/empresa
MySQL|	jdbc:mysql://localhost:3306/empresa
SQLite|	jdbc:sqlite:empresa.sqlite


Para que la conexión funcione, es necesario **añadir el conector jdbc** correspondiente. Para ello utilizaremos la herramienta **Gradle**, que permite automatizar la gestión de dependencias sin tener que configurar nada a mano.

- **build.gradle.kts** : 

        dependencies {
            implementation("org.postgresql:postgresql:42.7.1") //Postgres
            implementation("mysql:mysql-connector-java:8.3.0") //MySQL
            implementation("org.xerial:sqlite-jdbc:3.43.0.0") //SQLite
        }


!!!Tip ""
    A continuación se describe cómo conectar a una base de datos **SQLite** llamada **plantas.db** que se encuentra en la carpeta **resources** de un proyecto en **Kotlin**.


**conexion_SQLite.kt** 
       
        import java.io.File
        import java.sql.DriverManager

        fun main() {
            // Ruta al archivo de base de datos SQLite
            val dbPath = "src/main/resources/Tienda.sqlite"
            val dbFile = File(dbPath)
            println("Ruta de la BD: ${dbFile.absolutePath}")
            val url = "jdbc:sqlite:${dbFile.absolutePath}"

            // Conexión y prueba
            DriverManager.getConnection(url).use { conn ->
                println("Conexión establecida correctamente con SQLite.")
            }
        }

!!!Tip ""
    No se necesita usuario ni contraseña con SQLite, ya que es una base de datos local y embebida.     


Podemos encapsular la conexión a la base de datos dentro de un objeto para reutilizarla tantas veces como sea necesario. Así evitamos duplicar código y reducimos posibles errores. Por ejemplo, si la base de datos cambia de ubicación, solo habría que actualizar la ruta en el objeto y no en cada uno de los programas.

**conexion_SQLite_obj.kt**
       
        import java.io.File
        import java.sql.DriverManager

        object DatabasePlantas {
            // Ruta al archivo de base de datos SQLite
            val dbPath = "src/main/resources/plantas.db"
            val dbFile = File(dbPath)
            val url = "jdbc:sqlite:${dbFile.absolutePath}"

            fun getConnection() = DriverManager.getConnection(url)
        }

**conexion_objeto.kt**

            import java.io.File
            import java.sql.DriverManager
            import kotlin.use

            fun main() {
                val sql = "SELECT * FROM article"
                DatabaseTienda.getConnection().use { conn ->
                    conn.prepareStatement(sql).use { stmt ->
                        stmt.executeQuery().use { rs ->

                            while (rs.next()) {

                                //aquí iría el código
                            }
                        }
                    }
                }
            }

      
