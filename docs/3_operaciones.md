# Operaciones sobre la BD

En **JDBC** (Java Database Connectivity), las operaciones sobre la base de datos se realizan  utilizando los siguientes objetos y métodos:

- **Connection**, establece el canal de comunicación con el SGBD (PostgreSQL, MySQL, etc.)

- Los objetos **PreparedStatement** y **CreateStatement** se utlizan para enviar consultas SQL desde el programa a la base de datos. A continuación se muestra una tabla con el uso de cada uno:


| Si necesitas...                                     | Usa...            |
|-----------------------------------------------------|-------------------|
| Consultas sin parámetros                            | `CreateStatement`       |
| Consultas con datos del usuario                     | `PreparedStatement` |
| Seguridad frente a inyecciones SQL                  | `PreparedStatement` |
| Ejecutar muchas veces con distintos valores         | `PreparedStatement` |
| Crear tablas o sentencias SQL complejas que no cambian | `CreateStatement`


- Los métodos **executeQuery()**, **executeUpdate()** y **execute()** se utilizan para ejecutar sentencias SQL, pero se usan en contextos diferentes. A continuación se muestra una tabla con el uso de cada uno:


Método|	Uso principal|	Tipo de sentencia SQL|	Resultado que devuelve
------|--------------|-----------------------|------------------------
**executeQuery()**|	Realizar consultas|	SELECT|	Objeto  **ResultSet** con el resultado de la consulta SQL. Permite recorrer fila a fila el conjunto de resultados, accediendo a cada campo por nombre o por posición
**executeUpdate()**|Realizar modificaciones|	INSERT, UPDATE, DELETE, DDL (CREATE, DROP, etc.)|	Entero con el número de filas afectadas
**execute()**|No se sabe de antemano qué tipo de sentencia SQL se va a ejecutar (consulta o modificación)| Sentencias SQL que pueden devolver varios resultados| Booleano **true** si el resultado es un ResultSet (SELECT) y **false** si el resultado es un entero (INSERT, UPDATE, DELETE,CREATE, ALTER)

**CRUD** se refiere a las siglas de: **C**reate (crear), **R**ead (Leer), **U**pdate (Actualizar) y **D**elete (Borrar). 


<span class="mi_h2">Ejemplos en SQlite</span>

Los siguientes ejemplos se han realizado sobre la tabla `plantas`de la BD `plantas.sqlite`

**Ejemplo 1: Consulta sin parámetros** El siguiente ejemplo consulta toda la infromación de las plantas y la mustra por consola de forma ordenada:

``` kotlin
fun consultarPlantas() {
    val conn = BD.getConnection()
    if (conn != null) {
        val sql = "SELECT * FROM plantas"
        conn.prepareStatement(sql).use { stmt ->

            stmt.executeQuery().use { rs ->
                println("Plantas encontradas:")

                while (rs.next()) {
                    val idPlanta = rs.getString("id")
                    val nombreComun = rs.getString("nombre_comun")
                    val nombreCientifico = rs.getString("nombre_cientifico")
                    val frecuenciaRiego = rs.getInt("frecuencia_riego")
                    val altura = rs.getDouble("altura")

                    println("- $idPlanta: $nombreComun ($nombreCientifico). Frecuencia de riego: $frecuenciaRiego, altura: $altura")
                }
            }
        }
        BD.closeConnection(conn)
    }
}
```


**Ejemplo 2: Consulta con parámetros** El siguiente ejemplo consulta la infromación de la planta cuyo ID coincide con uno proporcionado por el usuario:
``` kotlin

``` 






**Ejemplo 3: INSERT** El siguiente ejemplo añade un registro a la tabla de la BD.

``` kotlin
fun insertarPlanta() {
    val conn = BD.getConnection()
    if (conn != null) {
        //datos a insertar
        val nombreComun = "Palmera"
        val nombreCientifico = "Arecaceae"
        val frecuenciaRiego = 2
        val altura = 8.5

        val sql = "INSERT INTO plantas (nombre_comun, nombre_cientifico, frecuencia_riego, altura) VALUES (?, ?, ?, ?)"

        conn.prepareStatement(sql).use { stmt ->
            stmt.setString(1, nombreComun)
            stmt.setString(2, nombreCientifico)
            stmt.setInt(3, frecuenciaRiego)
            stmt.setDouble(4, altura)
            stmt.executeUpdate()
        }
        BD.closeConnection(conn)
        println("Planta insertada correctamente")
    }
}
``` 



**Ejemplo 4: UPDATE**  El siguiente ejemplo actualiza la `altura` de una planta identificada por su `id` (el id de la planta a actualizar y la nueva altura se piden por teclado al usuario).

``` kotlin
fun actualizaPlanta() {
    var idPlanta = 0
    var nuevaAltura = 0.0

    print("Introduce un ID: ")
    val pideID = readLine()
    if (pideID != null && pideID.isNotBlank()) {
        idPlanta = pideID.toInt()

        print("Introduce la nueva altura: ")
        val pideAltura = readLine()
        if (pideAltura != null && pideAltura.isNotBlank()) {
            nuevaAltura = pideAltura.toDouble()
        }
    }

    val conn = BD.getConnection()
    if (conn != null) {
        val sql = "UPDATE plantas SET altura = ? WHERE id = ?"

        conn.prepareStatement(sql).use { stmt ->
            stmt.setDouble(1, nuevaAltura)
            stmt.setInt(2, idPlanta)

            stmt.executeUpdate()
        }
        BD.closeConnection(conn)
        println("Planta actualizada correctamente")
    }
}
```




**Sentencias DELETE**

Las sentencias **DELETE** permiten eliminar registros de una tabla.




**Ejemplo_Delete.kt**: Este fragmento elimina el articulo "00001"

       package SQLite
       import java.sql.DriverManager

        fun main() {
            val dbPath = "src/main/resources/Tienda.sqlite"
            val dbFile = java.io.File(dbPath)
            val url = "jdbc:sqlite:${dbFile.absolutePath}"

            DriverManager.getConnection(url).use { conn ->

                val sql = "DELETE FROM article WHERE cod_a = ?"
                conn.prepareStatement(sql).use { stmt ->
                stmt.setString(1, "00001")
                stmt.executeUpdate()
                }
            }
        }


FALTA a PARTIR DE AQUÍ

**Consultas complejas: JOIN, filtros y ordenaciones**

**Ejemplo_join.kt**: Este ejemplo obtiene las líneas de factura con nombre del artículo y ordenado por numero de factura y línea.

        package SQLite
        import java.sql.DriverManager

        fun main() {
            val dbPath = "src/main/resources/Tienda.sqlite"
            val dbFile = java.io.File(dbPath)
            val url = "jdbc:sqlite:${dbFile.absolutePath}"

            DriverManager.getConnection(url).use { conn ->

                val sql = """
                    SELECT lf.num_f, lf.num_l, lf.cod_a, a.descrip, lf.quant, lf.preu
                    FROM linia_fac lf
                    JOIN article a ON lf.cod_a = a.cod_a
                    ORDER BY lf.num_f, lf.num_l
                """.trimIndent()

                conn.prepareStatement(sql).use { stmt ->
                    stmt.executeQuery().use { rs ->
                        println("Líneas de factura:")
                        println("Factura | Línea | Artículo | Descripción | Cantidad | Precio")

                        while (rs.next()) {
                            val numF = rs.getInt("num_f")
                            val numL = rs.getInt("num_l")
                            val codA = rs.getString("cod_a")
                            val descrip = rs.getString("descrip")
                            val quant = rs.getInt("quant")
                            val preu = rs.getDouble("preu")

                            println("$numF\t$numL\t$codA\t$descrip\t$quant\t$preu")
                        }
                    }
                }
            }
        }



<span class="mi_h2">Liberación de recursos</span>


Cuando una aplicación accede a una base de datos, abre varios recursos internos que consumen memoria y conexiones activas en el sistema:

- La conexión con el servidor de base de datos (Connection)
- Las sentencias SQL preparadas (Statement o PreparedStatement)
- El resultado de la consulta (ResultSet)

Estos recursos no se liberan automáticamente cuando se termina su uso (especialmente en Java o Kotlin con JDBC). Si no se cierran correctamente, se pueden producir problemas como:

- Fugas de memoria
- Bloqueo de conexiones (demasiadas conexiones abiertas)
- Degradación del rendimiento
- Errores inesperados en la aplicación

En Kotlin, puedes usar **use {}** para cerrar recursos automáticamente al finalizar el bloque.

Si no utilizas **use {}** en Kotlin (o try-with-resources en Java), entonces debes cerrar manualmente cada uno de los recursos abiertos (ResultSet, Statement y Connection) usando .**close()**, y normalmente deberías hacerlo dentro de un bloque **finally** para garantizar su cierre incluso si ocurre un error. El orden correcto de cierre es del más interno al más externo.

**Ejemplos**{.azul} para cerrar recursos abiertos sin **use()**, de forma manual y con el bloque **try-catch-finally**

**Ejemplo_cierre_manual.kt:** Cierra los recurso con close()

        package SQLite
        import java.sql.DriverManager
        import java.sql.Connection
        import java.sql.PreparedStatement
        import java.sql.ResultSet

        fun main() {
            val dbPath = "src/main/resources/Tienda.sqlite"
            val dbFile = java.io.File(dbPath)
            val url = "jdbc:sqlite:${dbFile.absolutePath}"

            val conn: Connection = DriverManager.getConnection(url)
            val sql = "SELECT cod_a, descrip, preu, stock, stock_min FROM article"
            val stmt: PreparedStatement = conn.prepareStatement(sql)
            val rs: ResultSet = stmt.executeQuery()

            println("Artículos:")
            println("Código\tDescripción\tPrecio\tStock\tStock mín.")

            while (rs.next()) {
                val codA = rs.getString("cod_a")
                val descrip = rs.getString("descrip")
                val preu = rs.getDouble("preu")
                val stock = rs.getInt("stock")
                val stockMin = rs.getInt("stock_min")

                println("$codA\t$descrip\t$preu\t$stock\t$stockMin")
            }

            rs.close()
            stmt.close()
            conn.close()
        }


**Ejemplo_cierre_try_catch.kt:** Cierra los reursos con try-catch-finally

        package SQLite
        import java.sql.Connection
        import java.sql.DriverManager
        import java.sql.PreparedStatement
        import java.sql.ResultSet

        fun main() {
            val dbPath = "src/main/resources/Tienda.sqlite"
            val dbFile = java.io.File(dbPath)
            val url = "jdbc:sqlite:${dbFile.absolutePath}"

            var conn: Connection? = null
            var stmt: PreparedStatement? = null
            var rs: ResultSet? = null

            try {
                conn = DriverManager.getConnection(url)
                val sql = "SELECT cod_a, descrip, preu, stock, stock_min FROM article"
                stmt = conn.prepareStatement(sql)
                rs = stmt.executeQuery()

                println("Artículos:")
                println("Código\tDescripción\tPrecio\tStock\tStock mín.")

                while (rs.next()) {
                    val codA = rs.getString("cod_a")
                    val descrip = rs.getString("descrip")
                    val preu = rs.getDouble("preu")
                    val stock = rs.getInt("stock")
                    val stockMin = rs.getInt("stock_min")

                    println("$codA\t$descrip\t$preu\t$stock\t$stockMin")
                }

            } catch (e: Exception) {
                println("Error al acceder a la base de datos: ${e.message}")
            } finally {
                try { rs?.close() } catch (e: Exception) { /* Ignorar */ }
                try { stmt?.close() } catch (e: Exception) { /* Ignorar */ }
                try { conn?.close() } catch (e: Exception) { /* Ignorar */ }
            }
        }






<!--

## Ejemplos en PostgreSQL


!!!Tip "Kotlin - Instrucciones"
    En el proyecto `BDRelacionales` crearemos un **paquete** llamado `Postgres`, donde incluiremos los ejemplos de este apartado.   
   
    ![ref](img/carpeta_postgres.png)






**Read (SELECT)**   

**Ejemplo_Select.kt**

            package Postgres
            fun main(args: Array<String>) {
            val sql = "SELECT * FROM institut"

            DatabaseLocal.getConnection().use { conn ->    // DatabaseRemota si se conecta al servidor del instituto
                    conn.prepareStatement(sql).use { stmt ->
                    stmt.executeQuery().use { rs ->

                        while (rs.next()) {
                            print("" + rs.getString(1) + "\t")
                            println(rs.getString(2))
                        }
                    }
                }
                }
             }



---

**Create (INSERT)**{.verde}  
El siguiente ejemplo inserta un istituto de prueba.

**Ejemplo_Insert.kt**

        package Postgres
        fun main(args: Array<String>) {

            val sql ="INSERT INTO institut (codi,nom,adreca,numero,codpostal,cod_m) VALUES(?,?,?,?,?,?)"

            DatabaseLocal.getConnection().use { conn ->

                conn.prepareStatement(sql).use { stmt ->
                    stmt.setString(1, "00000000")
                    stmt.setString(2, "IES PRUEBA")
                    stmt.setString(3, "CASTELLÓN")
                    stmt.setString(4, "S/N")
                    stmt.setInt(5, 12560)
                    stmt.setInt(6, 12040)
                    stmt.executeUpdate()
                }
            }
        }



**Update (UPDATE)**{.verde}    
El siguiente ejemplo actualiza el campo nombre del instituto de prueba insertado. 

**Ejemplo_Update.kt**

        package Postgres
        fun main() {
            val sql = "UPDATE institut SET nom = ? WHERE codi = ?"

            DatabaseLocal.getConnection().use { conn ->

                conn.prepareStatement(sql).use { stmt ->
                    stmt.setString(1, "IES PRUEBA 2")
                    stmt.setString(2, "00000000")
                    stmt.executeUpdate()
                }
            }
        }


**Delete (DELETE)**{.verde}   
El siguiente ejemplo elimina el instituto de prueba insertado.  

**Ejemplo_Delete.kt**
        
        package Postgres
        fun main() {
            val sql = "DELETE FROM institut WHERE codi = ?"

            DatabaseLocal.getConnection().use { conn ->

                conn.prepareStatement(sql).use { stmt ->
                    stmt.setString(1, "00000000")
                    stmt.executeUpdate()
                }
            }
        }


-->
