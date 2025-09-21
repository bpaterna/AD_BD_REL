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



<span class="mi_h2">Buenas prácticas</span>

**Liberación de recursos**

Cuando una aplicación accede a una base de datos, abre varios recursos internos que consumen memoria y conexiones activas en el sistema:

- La conexión con el servidor de base de datos (Connection).
- Las sentencias SQL preparadas (Statement o PreparedStatement).
- El resultado de la consulta (ResultSet).

Estos recursos no se liberan automáticamente cuando se termina su uso (especialmente en Java o Kotlin con JDBC). Si no se cierran correctamente, se pueden producir problemas como:

- Fugas de memoria.
- Bloqueo de conexiones (demasiadas conexiones abiertas).
- Degradación del rendimiento.
- Errores inesperados en la aplicación.

En Kotlin, cuando usas **.use { ... }** sobre un objeto que implementa AutoCloseable (como Connection, Statement o ResultSet en JDBC), se cierra automáticamente al salir del bloque, aunque haya excepciones en el interior.

**Ejemplo con .use (sin necesidad de closeConnection):** Este ejemplo utiliza la función `getConnection` declarada en el fichero `BD.kt` para abrir la conexión de forma que:

- **conn.use { ... }** cierra la conexión automáticamente al final del bloque.

- **stmt.use { ... }** cierra el Statement automáticamente.

- **ResultSet** se cierra cuando cierras el Statement.

``` kotlin
fun main() {
    PlantasBD.getConnection()?.use { conn ->
        println("Conectado a la BD")

        conn.createStatement().use { stmt ->
            val rs = stmt.executeQuery("SELECT * FROM plantas")
            while (rs.next()) {
                println("${rs.getString("nombre_comun")}")
            }
        }
    } ?: println("No se pudo conectar")
}
```

!!! success "Realiza lo siguiente" 
    Prueba el código de ejemplo y verifica que funciona correctemente.


Si no utilizas **use {}** en Kotlin (o try-with-resources en Java), entonces debes cerrar manualmente cada uno de los recursos abiertos (ResultSet, Statement y Connection) usando .**close()**, y normalmente deberías hacerlo dentro de un bloque **finally** para garantizar su cierre incluso si ocurre un error. El orden correcto de cierre es del más interno al más externo.



FALTA ejemplo

!!! success "Realiza lo siguiente" 
    Prueba el código de ejemplo y verifica que funciona correctemente.


Otra buena práctica es separar en objetos de acceso a datos (Data Access Object o DAO) las diferentes operaciones CRUD:
- C → Create (Crear / Insertar un nuevo registro)
- R → Read (Leer / Consultar registros)
- U → Update (Actualizar un registro existente)
- D → Delete (Eliminar un registro)

**Ventajas de usar DAO**
- Organización: todo el código SQL está en un único lugar.
- Reutilización: puedes llamar a PlantaDAO.listarPlantas() desde distintos sitios sin repetir la consulta.
- Mantenibilidad: si cambia la base de datos, solo tocas el DAO.
- Claridad: el resto de tu app se lee mucho más limpio, sin SQL mezclado.


<span class="mi_h2">Ejemplo DAO en SQlite</span>

El siguiente ejemplo es el DAO para la tabla `plantas` de la BD `plantas.sqlite` en la que se utiliza el código de conexión del objeto **PlantasBD.kt**.

Se declaran funciones para leer la información de la tabla, añdir registros nuevos, modificar la información existenete y borrarla. Para ello se utiliza un data class **Planta.kt** con la estructura siguiente:

``` kotlin
data class Planta(
    val id: Int? = null, // lo genera SQLite automáticamente
    val nombreComun: String,
    val nombreCientifico: String,
    val frecuenciaRiego: Int,
    val altura: Double
)
```

El código del archivo **PlantasDAO.kt** es el siguiente:
``` kotlin
object PlantaDAO {

    fun listarPlantas(): List<Planta> {
        val lista = mutableListOf<Planta>()
        DatabasePlantas.getConnection()?.use { conn ->
            conn.createStatement().use { stmt ->
                val rs = stmt.executeQuery("SELECT * FROM plantas")
                while (rs.next()) {
                    lista.add(
                        Planta(
                            id = rs.getInt("id"),
                            nombreComun = rs.getString("nombre_comun"),
                            nombreCientifico = rs.getString("nombre_cientifico"),
                            frecuenciaRiego = rs.getInt("frecuencia_riego"),
                            altura = rs.getDouble("altura")
                        )
                    )
                }
            }
        } ?: println("❌ No se pudo establecer la conexión.")
        return lista
    }

    fun consultarPlantaPorId(id: Int): Planta? {
        var planta: Planta? = null
        PlantasBD.getConnection()?.use { conn ->
            conn.prepareStatement("SELECT * FROM plantas WHERE id = ?").use { pstmt ->
                pstmt.setInt(1, id)
                val rs = pstmt.executeQuery()
                if (rs.next()) {
                    planta = Planta(
                        id = rs.getInt("id"),
                        nombreComun = rs.getString("nombre_comun"),
                        nombreCientifico = rs.getString("nombre_cientifico"),
                        frecuenciaRiego = rs.getInt("frecuencia_riego"),
                        altura = rs.getDouble("altura")
                    )
                }
            }
        } ?: println("No se pudo establecer la conexión.")
        return planta
    }

    fun insertarPlanta(planta: Planta) {
        DatabasePlantas.getConnection()?.use { conn ->
            conn.prepareStatement(
                "INSERT INTO plantas(nombre_comun, nombre_cientifico, frecuencia_riego, altura) VALUES (?, ?, ?, ?)"
            ).use { pstmt ->
                pstmt.setString(1, planta.nombreComun)
                pstmt.setString(2, planta.nombreCientifico)
                pstmt.setInt(3, planta.frecuenciaRiego)
                pstmt.setDouble(4, planta.altura)
                pstmt.executeUpdate()
                println("🌱 Planta '${planta.nombreComun}' insertada con éxito.")
            }
        } ?: println("❌ No se pudo establecer la conexión.")
    }

    fun actualizarPlanta(planta: Planta) {
        if (planta.id == null) {
            println("⚠️ No se puede actualizar una planta sin id.")
            return
        }
        DatabasePlantas.getConnection()?.use { conn ->
            conn.prepareStatement(
                "UPDATE plantas SET nombre_comun = ?, nombre_cientifico = ?, frecuencia_riego = ?, altura = ? WHERE id = ?"
            ).use { pstmt ->
                pstmt.setString(1, planta.nombreComun)
                pstmt.setString(2, planta.nombreCientifico)
                pstmt.setInt(3, planta.frecuenciaRiego)
                pstmt.setDouble(4, planta.altura)
                pstmt.setInt(5, planta.id)
                val filas = pstmt.executeUpdate()
                if (filas > 0) {
                    println("✏️ Planta con id=${planta.id} actualizada con éxito.")
                } else {
                    println("⚠️ No se encontró ninguna planta con id=${planta.id}.")
                }
            }
        } ?: println("❌ No se pudo establecer la conexión.")
    }

    fun eliminarPlanta(id: Int) {
        DatabasePlantas.getConnection()?.use { conn ->
            conn.prepareStatement("DELETE FROM plantas WHERE id = ?").use { pstmt ->
                pstmt.setInt(1, id)
                val filas = pstmt.executeUpdate()
                if (filas > 0) {
                    println("🗑️ Planta con id=$id eliminada correctamente.")
                } else {
                    println("⚠️ No se encontró ninguna planta con id=$id.")
                }
            }
        } ?: println("❌ No se pudo establecer la conexión.")
    }
}
```

La llamada a estas funciones desde **main.kt** podría ser:

``` kotlin
fun main() {

    // Listar todas las plantas
    println("Lista de plantas:")
    PlantasDAO.listarPlantas().forEach {
        println(" - [${it.id}] ${it.nombreComun} (${it.nombreCientifico}), riego cada ${it.frecuenciaRiego} días, altura: ${it.altura} m")
    }

    // Consultar planta por ID
    val planta = PlantasDAO.consultarPlantaPorId(3)
    if (planta != null) {
        println("Planta encontrada: [${planta.id}] ${planta.nombreComun} (${planta.nombreCientifico}), riego cada ${planta.frecuenciaRiego} días, altura: ${planta.altura} m")
    } else {
        println("No se encontró ninguna planta con ese ID.")
    }

    // Insertar plantas
    PlantasDAO.insertarPlanta(
        Planta(
            nombreComun = "Palmera",
            nombreCientifico = "Arecaceae",
            frecuenciaRiego = 2,
            altura = 8.5
        )
    )

    // Actualizar planta con id=1
    PlantasDAO.actualizarPlanta(
        Planta(
            id = 1,
            nombreComun = "Aloe Arborescens",
            nombreCientifico = "Aloe barbadensis miller",
            frecuenciaRiego = 5,
            altura = 0.8
        )
    )

    // Eliminar planta con id=2
    PlantasDAO.eliminarPlanta(2)
}
```

!!! success "Realiza lo siguiente" 
    Prueba el código de ejemplo y verifica que funciona correctemente.

!!! warning "Práctica 4: Trabaja con tu base de datos" 
    Replica el ejemplo anterior para que funcione con tu base de datos.







<!-- 
**Ejemplo 2 - Consulta con parámetros:** El siguiente ejemplo consulta la información de la planta cuyo ID coincide con el pasado como parámetro.

``` kotlin
fun consultarPlantaPorId(id: Int) {
    val conn = BD.getConnection()
    if (conn != null) {
        val sql = "SELECT * FROM plantas WHERE id = ?"
        try {
            conn.prepareStatement(sql).use { stmt ->
                stmt.setInt(1, id)

                stmt.executeQuery().use { rs ->
                    if (rs.next()) {
                        val idPlanta = rs.getInt("id")
                        val nombreComun = rs.getString("nombre_comun")
                        val nombreCientifico = rs.getString("nombre_cientifico")
                        val frecuenciaRiego = rs.getInt("frecuencia_riego")
                        val altura = rs.getDouble("altura")

                        println("Planta encontrada:")
                        println("- $idPlanta: $nombreComun ($nombreCientifico). Frecuencia de riego: $frecuenciaRiego días, altura: $altura m")
                    } else {
                        println("No se encontró ninguna planta con id=$id")
                    }
                }
            }
        } catch (e: Exception) {
            println("Error: ${e.message}")
        } finally {
            BD.closeConnection(conn)
        }
    } else {
        println("No se pudo establecer la conexión a la base de datos.")
    }
}
``` 

!!! success "Realiza lo siguiente" 
    Prueba el código de ejemplo y verifica que funciona correctemente.

!!! warning "Práctica 5: Conecta a tu base de datos" 
    Replica el ejemplo anterior para que funcione con tu base de datos.


**Ejemplo 3 - INSERT:** El siguiente ejemplo añade un registro a la tabla de la BD.

``` kotlin
fun insertarPlanta() {
    val conn = BD.getConnection()
    if (conn != null) {
        //datos de ejemplo a insertar
        val nombreComun = "Palmera"
        val nombreCientifico = "Arecaceae"
        val frecuenciaRiego = 2
        val altura = 8.5

        val sql = "INSERT INTO plantas (nombre_comun, nombre_cientifico, frecuencia_riego, altura) VALUES (?, ?, ?, ?)"
        try {
            conn.prepareStatement(sql).use { stmt ->
                stmt.setString(1, nombreComun)
                stmt.setString(2, nombreCientifico)
                stmt.setInt(3, frecuenciaRiego)
                stmt.setDouble(4, altura)
                stmt.executeUpdate()
            }
        } catch (e: Exception) {
            println("Error: ${e.message}")
        } finally {
            println("Planta insertada correctamente")
            BD.closeConnection(conn)
        }
    } else {
        println("No se pudo establecer la conexión a la base de datos.")
    }
}
``` 

!!! success "Realiza lo siguiente" 
    Prueba el código de ejemplo y verifica que funciona correctemente.

!!! warning "Práctica 6: Conecta a tu base de datos" 
    Replica el ejemplo anterior para que funcione con tu base de datos.


**Ejemplo 4 - UPDATE:**  El siguiente ejemplo actualiza la `altura` de una planta identificada por su `id` (el id de la planta a actualizar y la nueva altura se piden por consola antes de llamar a la función).

``` kotlin
fun actualizarPlanta(id: Int, altura: Double) {
    val conn = BD.getConnection()
    if (conn != null) {
        val sql = "UPDATE plantas SET altura = ? WHERE id = ?"
        try {
            conn.prepareStatement(sql).use { stmt ->
                stmt.setDouble(1, altura)
                stmt.setInt(2, id)

                stmt.executeUpdate()
            }
        } catch (e: Exception) {
            println("Error: ${e.message}")
        } finally {
            println("Planta actualizada correctamente")
            BD.closeConnection(conn)
        }
    } else {
        println("No se pudo establecer la conexión a la base de datos.")
    }
}
```

!!! success "Realiza lo siguiente" 
    Prueba el código de ejemplo y verifica que funciona correctemente.

!!! warning "Práctica 7: Conecta a tu base de datos" 
    Replica el ejemplo anterior para que funcione con tu base de datos.


**Ejemplo 5 - DELETE:** El siguiente ejemplo elimina una planta identificada por su `id` que se pasa por parámetro a la función.

``` kotlin
fun eliminarPlanta(id: Int) {
    val conn = BD.getConnection()
    if (conn != null) {
        val sql = "DELETE FROM plantas WHERE id = ?"
        try {
            conn.prepareStatement(sql).use { stmt ->
                stmt.setInt(1, id)
                stmt.executeUpdate()
            }
        } catch (e: Exception) {
            println("Error: ${e.message}")
        } finally {
            println("Planta eliminada correctamente")
            BD.closeConnection(conn)
        }
    } else {
    println("No se pudo establecer la conexión a la base de datos.")
    }
}
```

!!! success "Realiza lo siguiente" 
    Prueba el código de ejemplo y verifica que funciona correctemente.

!!! warning "Práctica 8: Conecta a tu base de datos" 
    Replica el ejemplo anterior para que funcione con tu base de datos.


-->








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




**Ejemplos** para cerrar recursos abiertos sin **use()**, de forma manual y con el bloque **try-catch-finally**

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
