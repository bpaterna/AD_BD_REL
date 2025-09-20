# Conexión a un SGBD

Cuando desarrollamos aplicaciones que trabajan con información persistente, necesitamos acceder a BD para consultar, insertar, modificar o eliminar datos. Existen dos formas principales de hacerlo desde el código: 

  - Acceso mediante ORM (Object-Relational Mapping).
  - Acceso mediante conectores.


📌 **Acceso mediante ORM**

Un **ORM** es una herramienta que permite trabajar con la base de datos como si fuera un conjunto de objetos, evitando tener que escribir directamente SQL. El **ORM** se encarga de mapear las tablas a clases y los registros a objetos, y traduce automáticamente las operaciones del código a consultas SQL. Es ideal para trabajar de forma más productiva en aplicaciones complejas. Sus principales características son:

 - Se trabaja con clases en lugar de tablas SQL.
 - Ahorra mucho código repetitivo.
 - Ideal para proyectos medianos o grandes que requieren mantener muchas entidades.

**Algunos ejemplos de ORMs**

ORM / Framework|	Lenguaje|	Descripción
---------------|---------|-----------------
Hibernate|	Java/Kotlin|	El ORM más utilizado con JPA
Exposed|	Kotlin|	ORM ligero y expresivo creado por JetBrains
Spring Data JPA|	Java/Kotlin|	Abstracción que automatiza el acceso a datos
Room|	Java/Kotlin|	ORM oficial para bases de datos SQLite en Android    

**JPA** (Java Persistence API) es una especificación estándar de Java que define cómo se deben mapear objetos Java (o Kotlin) a tablas de bases de datos relacionales. Es decir, permite gestionar la persistencia de datos de forma orientada a objetos, sin necesidad de escribir SQL directamente. Es el estándar utilizado por las herramientas ORM como Hibernate, EclipseLink, o Spring Data JPA.


📌 **Acceso mediante conectores{.subtitulo}**

En la introducción ya vimos que un **conector** (también llamado driver) es una librería software que permite que una aplicación se comunique con un gestor de base de datos (SGBD). Actúa como un puente entre nuestro código y la base de datos, traduciendo las instrucciones SQL a un lenguaje que el gestor puede entender y viceversa. Sin un conector, tu aplicación no podría comunicarse con la base de datos.

Una base de datos puede ser accedida desde diferentes orígenes o herramientas, siempre que tengamos:

- Las credenciales de acceso (usuario y contraseña)
- El host/servidor donde se encuentra la base de datos
- El motor de base de datos (PostgreSQL, MySQL, SQLite, etc.)
- Los puertos habilitados y los permisos correctos


Las principales formas de conectarse a una base de datos son las siguientes:

| Medio de conexión                         | Descripción                                                                 |
|-------------------------------------------|-----------------------------------------------------------------------------|
| Aplicaciones de escritorio             | Herramientas gráficas como **DBeaver**, **pgAdmin**, **MySQL Workbench**, **DB Browser for SQLite**. Permiten explorar, consultar y administrar BD de forma visual. |
| Aplicaciones desarrolladas en código   | Programas en **Kotlin**, **Java**, **Python**, **C#**, etc., mediante **conectores** como **JDBC**, **psycopg2**, **ODBC**, etc. para acceder a BD desde código. |
| Línea de comandos                      | Clientes como `psql` (PostgreSQL), `mysql`, `sqlite3`. Permiten ejecutar comandos SQL directamente desde terminal. |
| Aplicaciones web                        | Sitios web que acceden a BD desde el backend (por ejemplo, en Spring Boot, Node.js, Django, etc.). |
| APIs REST o servidores intermedios     | Servicios web que conectan la BD con otras aplicaciones, actuando como puente o capa de seguridad. |
| Aplicaciones móviles                   | Apps Android/iOS que acceden a BD locales (como **SQLite**) o remotas (vía **Firebase**, API REST, etc.). |
| Herramientas de integración de datos   | Software como **Talend**, **Pentaho**, **Apache Nifi** para migrar, transformar o sincronizar datos entre sistemas. |

De todas las formas posibles de interactuar con una base de datos, nos vamos a centrar en el uso de **conectores JDBC (Java Database Connectivity)**. **JDBC** es una API estándar de Java (y compatible con Kotlin) que permite conectarse a una BD, enviar instrucciones SQL y procesar los resultados manualmente. Es el método de más bajo nivel, pero ofrece un control total sobre lo que ocurre en la BD. Es ideal para aprender los fundamentos del acceso a datos y tener control total y aprenderlo ayuda a entender mejor lo que hace un ORM por debajo. Sus principales características son:

 - El programador escribe directamente las consultas SQL.
 - Requiere gestionar manualmente conexiones, sentencias y resultados.
 - Se necesita un driver específico (conector) para cada SGBD:


**Algunos ejemplos de conectores**

SGBD|	Conector (Driver JDBC)|	URL de conexión típica
----|-------------------------|-----------------------
PostgreSQL|	org.postgresql.Driver| jdbc:postgresql://host:puerto/basedatos
MySQL / MariaDB|	com.mysql.cj.jdbc.Driver| jdbc:mysql://host:puerto/basedatos
SQLite (embebido)|	org.sqlite.JDBC	|jdbc:sqlite:ruta_al_fichero

Una aplicación (escrita en Kotlin, Java u otro lenguaje) puede leer, insertar o modificar información almacenada en una base de datos relacional si previamente se ha conectado al sitema gestor de base de datos (SGBD). **JDBC** (Java Database Connectivity) es la API básica de Java (conector) para conectarse a bases de datos relacionales y su sintaxis general es:

    jdbc:<gestor>://<host>:<puerto>/<nombre_base_datos>

Aunque puede variar según el SGBD con el que se trabaje. Por ejemplo en SQLite no se necesita usuario ni contraseña ya que es una base de datos local y embebida. 

SGBD|	URL de conexión
-----------------------|---------------------
PostgreSQL|	jdbc:postgresql://localhost:5432/plantas
MySQL|	jdbc:mysql://localhost:3306/plantas
SQLite|	jdbc:sqlite:plantas.sqlite

También dependiendo del SGBD será necesario utilizar un **conector JDBC** u otro. Para ello utilizaremos la herramienta **Gradle**, que permite automatizar la gestión de dependencias sin tener que configurar nada a mano añadiendo las líneas correspondientes en el fichero **build.gradle.kts**. A continuación se muestran las líneas para los SGBD PostgreSQL, MySQL y SQLite.

```
dependencies {
    implementation("org.postgresql:postgresql:42.7.1") //Postgres
    implementation("mysql:mysql-connector-java:8.3.0") //MySQL
    implementation("org.xerial:sqlite-jdbc:3.43.0.0") //SQLite
}
```

## Conexión a SQLite

A continuación se describe cómo conectar a una base de datos **SQLite** llamada **plantas.sqlite** que se encuentra en la carpeta **datos** dentro de un proyecto en **Kotlin**.


``` kotlin
import java.io.File
import java.sql.DriverManager

fun main() {
    // Ruta al archivo de base de datos SQLite
    val dbPath = "src/main/resources/plantas.sqlite"
    val dbFile = File(dbPath)
    println("Ruta de la BD: ${dbFile.absolutePath}")
    val url = "jdbc:sqlite:${dbFile.absolutePath}"

    // Conexión y prueba
    DriverManager.getConnection(url).use { conn ->
        println("Conexión establecida correctamente con SQLite.")
    }
}
```


!!! warning "Práctica 2: Comprobar conexión" 
    1. Añade al archivo **build.gradle.kts** en la sección de dependencias la línea correspondiente al **conector JDBC** de **SQLite**.
    2. Copia el código anterior y modifica lo que necesites para comprobar que el programa establece conexión correctamente con tu BD.


Una buena páctica es cerrar la conexión a la base de datos después de realizar operaciones sobre ella para que permanezca abierta el mínimo tiempo posible. Una forma de conseguir esto es tener las funciones de conexión y desconexión en un objeto separado. 

Además, otra ventaja de trabajar así es que si la base de datos cambia de ubicación, solo habrá que actualizar la ruta dentro del código del objeto que maneja la conexión y no en varios sitios del programa. 

Un ejemplo del código podría ser el siguiente:

``` kotlin
import java.io.File
import java.sql.Connection
import java.sql.DriverManager
import java.sql.SQLException

object DatabaseObj {
    // Ruta al archivo de base de datos SQLite
    private val dbPath = "datos/plantas.sqlite"
    private val dbFile = File(dbPath)
    private val url = "jdbc:sqlite:${dbFile.absolutePath}"

    // Obtener conexión
    fun getConnection(): Connection? {
        return try {
            DriverManager.getConnection(url)
        } catch (e: SQLException) {
            e.printStackTrace()
            null
        }
    }

    // Función de prueba: verificar conexión
    fun testConnection(): Boolean {
        return getConnection()?.use { conn ->
            println("Conexión establecida con éxito a ${dbFile.absolutePath}")
            true
        } ?: false
    }

    // Cerrar conexión
    fun closeConnection(conn: Connection?) {
        try {
            conn?.close()
            println("Conexión cerrada correctamente.")
        } catch (e: SQLException) {
            println("Error al cerrar la conexión: ${e.message}")
        }
    }
}
```

De esta forma, cuando el programa necesite acceder a la BD llamará a la función de conexión y cuando termine llamará a la fución que cierra la conexión. Un ejemplo de estas llamadas podría ser:

**conexion_objeto.kt**
``` kotlin
import java.io.File
import java.sql.DriverManager
import kotlin.use

fun main() {
    val conn = DatabaseObj.getConnection()
    if (conn != null) {
        println("Conectado a la BD correctamente.")
        DatabaseObj.closeConnection(conn)
    }
}

```
      
!!! warning "Práctica 3: Organizar conexión a la BD" 
        1. Crea el archivo con las funciones de conexión y desconexión a la BD.
        2. Comprueba desde el main que el programa se conecta a la BD correctamente y luego cierra la conexión.



## Conexión a PostgreSQL

En entornos reales y profesionales, lo más habitual es trabajar con SGBD más potentes y completos, como **PostgreSQL**.

A continuación, aplicaremos lo aprendido en SQLite, pero ahora trabajando con **PostgreSQL** en **dos contextos distintos**:

- Base de datos **remota**: alojada en un servidor accesible mediante una dirección IP y credenciales.
- Base de datos **local**: replicada en un contenedor **Docker**, lo que resulta ideal para pruebas, desarrollo y aprendizaje en un entorno controlado.

En ambos casos utilizaremos la misma base de datos, llamada geo_ad. Su versión remota estará disponible desde cualquier ubicación, mientras que la local se generará a partir de ella siguiendo unas instrucciones que se os facilitarán. El esquema relacional de `geo_ad` es el siguiente:

**Esquema de la BD geo_ad**

![ref](img/geo_ad.jpg)


!!!Note "Datos de conexión al servido remoto"      
    **Servidor (host)**: 89.36.214.106  
    **Port**: 5432 (és el port per defecte)  
    **Usuari**: geo_ad  
    **Contrasenya**: geo_ad  
    **Base de dades**: geo_ad  


!!!Note "Intrucciones para replicar la BD en local (Docker)"   
    Las instrucciones para replicar la base de datos en Docker las podéis encontrar en el siguiente enlace: [Instrucciones](https://docs.google.com/document/d/1uU5B9MonTf1KhIOP5PkECIfP-NCSkdzDAo2W33P81Js/edit?tab=t.0)



**Configuración de Dependencias (Gradle)**{.azul}

Lo primero será incluir las dependencia necesarias en **build.gradle.kts**

        // build.gradle.kts (para PostgreSQL)
        dependencies {
            implementation("org.postgresql:postgresql:42.6.0")
        }

**Conexión al servidor**{.azul}


**Postgres remoto**{.verde}

**Ejemplo_conexion_Postgres_remota.kt**

        package Postgres
        import java.sql.DriverManager
        object DatabaseRemota {

            private const val URL =  "jdbc:postgresql://89.36.214.106:5432/geo_ad"
            private const val USER = "geo_ad"
            private const val PASSWORD = "geo_ad"

            fun getConnection() = DriverManager.getConnection(URL, USER, PASSWORD)
        }

**Postgress en Docker**{.verde}

**Ejemplo_conexion_Postgres_local.kt**
        
        package Postgres
        import java.sql.DriverManager
        object DatabaseLocal {

            private const val URL =  "jdbc:postgresql://localhost:5432/geo"
            private const val USER = "postgres"
            private const val PASSWORD = "postgres"

            fun getConnection() = DriverManager.getConnection(URL, USER, PASSWORD)
        }

