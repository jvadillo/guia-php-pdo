
# Guía rápida de PDO

Esta es una guía rápida para aprender a utilizar   **PDO**. Los ejemplos que encontrarás en esta guía están pensados para   **MySQL** pero eres libre de adaptarlos a cualquier otro gestor de bases de datos soportado por PDO.

## Establecer conexión con la base de datos
El ejemplo a continuación muestra como establecer la conexión en distintos tipos de bases de datos.

```php
function connect(){
  try {
    # MS SQL Server
    $dbh= new PDO("mssql:host=$host;dbname=$dbname, $user, $pass");
   
    # MySQL
    $dbh= new PDO("mysql:host=$host;dbname=$dbname", $user, $pass);
   
    # SQLite Database
    $dbh= new PDO("sqlite:my/database/path/database.db");
  }
  catch(PDOException $e) {
      echo $e->getMessage();
  }
}
```

En caso de ocurrir un error al intentar establecer una conexión a base de datos, este siempre producirá una **excepción**. Por lo tanto, cualquier intento de creación de conexión a base de datos deberá realizarse dentro de un bloque **try/catch**.

## Cerrar la conexión
Una conexión a base de datos con PDO permanecerá abierta mientras exista el objeto PDO creado. Por lo tanto, para cerrarla, bastará con asignarle al objeto el valor `null`. De lo contrario, PHP mantendrá la conexión abierta hasta que finalice la ejecución del script.
```php
function close(){
    $dbh = null;
}
```

## Ejecutar sentencias SQL
Los pasos para ejecutar una sentencia SQL son:

 1. **Establecer** conexión con la base de datos (ver punto anterior)
 2. **Preparar** la sentencia
 3. **Ejecutar** sentencia (asociando parámetros si fuese necesario)
 4. Opcional: tratar el resultado de la sentencia ejecutada.

### Preparar la sentencia
Podemos preparar la sentencia a ejecutar de forma sencilla 
```php
$stmt = $dbh->prepare("SELECT nombre, apellidos FROM alumnos");
```
En caso de necesitar incluir parámetros en la sentencia, podemos realizarlo utilizando la sintaxis `:nombre`

```php
$stmt = $dbh->prepare("SELECT nombre, apellidos FROM alumnos WHERE edad > :edad");
```

De la misma forma que hemos preparado una consulta de tipo SELECT, también podemos preparar un INSERT, UPDATE, etc.

```php
$stmt= $dbh->prepare("INSERT INTO alumnos(nombre, apellidos) values (:nombre, :apellidos)");
```

### Ejecutar la sentencia
Para ejecutar una sentencia simple que no requiera de parámetros, podemos invocar directamente el método `execute()`. 
```php
$stmt = $dbh->prepare("SELECT nombre, apellidos FROM alumnos");
$stmt->execute();
```
En caso de necesitar incluir parámetros en la sentencia, pasaremos al método `execute()` la información en un array asociativo:

```php
$data = array( 'nombre' => 'Mikel', 'edad' => 15 );
$stmt = $dbh->prepare("SELECT nombre, apellidos FROM alumnos WHERE nombre = :nombre AND edad = :edad");
$stmt->execute($data);
```
De la misma forma que hemos preparado una consulta de tipo SELECT, también podemos preparar un INSERT, UPDATE, etc.

```php
$data = array( 'nombre' => 'Mikel', 'edad' => 15 );
$stmt = $dbh->prepare("INSERT INTO alumnos(nombre, edad) values (:nombre, :edad)");
$stmt->execute($data);
```

## Tratar los resultados de una consulta SELECT

Una vez ejecutada la sentencia `execute()` podremos acceder a los resultados obtenidos de la base de datos. PDO nos ofrece la posiblidad de recibir los resultados en distintos formatos: objetos de una clase, arrays asociativos, etc. Para indicarle cómo queremos recoger los resultados utilizaremos el método `setFetchMode(String mode)`. 

Estos son los 3 valores más utilizados y que nos servirán para cubrir prácticamente todas nuestras necesidades:

 - PDO::FETCH_ASSOC: devuelve un array cuyos índices serán los nombres de las columnas.
 - PDO::FETCH_CLASS: Asigna los valores de las columnas a las propiedades de la clase. Creará nuevas propiedades en caso de que no existan para las columnas.
 - PDO::FETCH_OBJ: devuelve objetos anónimos con propiedades que corresponden a los nombres de columna.

Una vez indicado el cómo queremos los datos, utilizaremos el método `fetch()` para acceder a la información. El método `fetch()` obtiene la siguiente fila de un conjunto de resultados, por lo que podremos iterar por los resultados tal y como se muestra en los ejemplos siguientes:

```php
function fetchAssoc(){
	// Este tipo de fetch crea un array asociativo, indexado por el nombre de la columna.
	
	$data = array( 'nombre' => 'Mikel', 'edad' => 15 );
	$stmt = $dbh->prepare("SELECT nombre, apellidos, edad FROM alumnos WHERE nombre = :nombre AND edad = :edad");
	// Establecemos el modo en el que queremos recibir los datos
	$stmt->setFetchMode(PDO::FETCH_ASSOC);
	// Ejecutamos la sentencia
	$stmt->execute($data);
	// Mostramos los resultados obtenidos
	while($row = $stmt->fetch()) {
		echo $row['nombre'] . "\n";
		echo $row['apellidos'] . "\n";
		echo $row['edad '] . "\n";
	}
}

function fetch_obj(){
	// Este método crea un objeto por cada fila obtenida de la base de datos.

	$data = array( 'nombre' => 'Mikel', 'edad' => 15 );
	$stmt = $dbh->prepare("SELECT nombre, apellidos, edad FROM alumnos WHERE nombre = :nombre AND edad = :edad");
	// Establecemos el modo en el que queremos recibir los datos
	$stmt->setFetchMode(PDO::FETCH_OBJ);
	// Ejecutamos la sentencia
	$stmt->execute($data);
	
	// Mostramos los resultados obtenidos
	while($row = $stmt->fetch()) {
		echo $row->nombre . "\n";
		echo $row->apellidos . "\n";
		echo $row->edad . "\n";
	}

}

function fetch_class(){
	// Este método devuelve los datos como objetos de la clase que nosotros le hayamos indicado.
	// Las propiedades del objeto se inicializarán con los datos de la base de datos antes de llamar al constructor.

	// Si hubiese nombres de columnas que no tienen una propiedad en la clase, se crearán como propiedades de tipo public

	// Se pueden realizar transformaciones sobre esos datos en el constructor de la clase.

	class Alumno {

		public $nombre;
		public $apellidos;
		public $edad;
		public $otraInformacion;

		function __construct($otraInformacion= '') {
			// El constructor se ejecutará después de asociar los valores obtenidos de la base de datos al objeto. Por lo tanto, podemos tratar esos valores dentro del constructor.
			$this->nombre = strtoupper($this->nombre);
			$this->otraInformacion = $otraInformacion;
		}
	}

	$data = array( 'nombre' => 'Mikel', 'edad' => 15 );
	$stmt = $dbh->prepare("SELECT nombre, apellidos FROM alumnos WHERE nombre = :nombre AND edad = :edad");
	// Establecemos el modo en el que queremos recibir los datos
	$stmt->setFetchMode(PDO::FETCH_CLASS, 'Alumno');
	// Ejecutamos la sentencia
	$stmt->execute($data);

	// Mostramos los resultados
	while($obj = $stmt->fetch()) {
		echo $obj->nombre;
	}
}

```
## Método abreviado query()
En consultas que no reciban parámetros, podemos utilizar el método abreviado `query()` el cual ejecutará la sentencia y nos devolverá el conjunto de resultados directamente. En otras palabras, no es necesario hacer la operación en 2 pasos (`prepare()` y `execute()`) como hacíamos hasta ahora.

```php
	$stmt = $dbh->query('SELECT nombre, apellidos, edad from empleado');

	// Establecemos el modo en el que queremos recibir los datos
	$stmt->setFetchMode(PDO::FETCH_ASSOC);

	while($row = $stmt->fetch()) {
		echo $row['nombre'] . "\n";
		echo $row['apellidos'] . "\n";
		echo $row['edad '] . "\n";
	}
```
Por razones de seguridad (evitar [SQL Injection](https://es.wikipedia.org/wiki/Inyecci%C3%B3n_SQL)) es recomendable evitar el método `query()` cuando la sentencia incluya valores variables. Por razones de rendimiento también se recomienda utilizar `prepare()` y `execute()` en sentencias que vayan a ejecutarse varias veces.

## Método fetchObject()
Existe una alternativa al método `fetch()` la cual devolverá los resultados cómo objetos anónimos (**`PDO::FETCH_OBJ`** ) u objetos de la clase indicada ( **`PDO::FETCH_CLASS`**). Este método se llama `fetchObject()`.

```php
	$stmt = $dbh->query('SELECT nombre, apellidos, edad from empleado');

	while($persona = $stmt ->fetchObject()) {
		echo $persona->nombre;
		echo $persona->apellidos;
	}
```
En caso de que queremos que los objetos pertenezcan a una clase en concreto, es suficiente con indicárselo en la llamada:

```php
	$stmt = $dbh->query('SELECT nombre, apellidos, edad from empleado');

	while($persona = $stmt ->fetchObject('Alumno')) {
		echo $persona->nombre;
		echo $persona->apellidos;
	}
```

## Obtener todos los resultados con fetchAll()
A diferencia del método `fetch()`, `fetchAll()` te trae todos los datos de golpe, sin abrir ningún puntero, almacenándolos en un array. Se recomienda cuando no se esperan demasiados resultados que podrían provocar problemas de memoria al querer guardar de golpe en un array miles de filas provenientes de un SELECT.

```php
	// En este caso $resultado será un array asociativo con todos los datos de la base de datos
	$resultado = $stmt->fetchAll(PDO::FETCH_ASSOC);
	
	// Para leer las filas podemos recorrer el array y acceder a la información.
	foreach ($resultado as $row){
	    echo $row["nombre"]." ".$row["apellido"].PHP_EOL;
	}
```
Es casi idéntico que en `fetch()`, sólo que aquí no estamos actuando sobre un recorrido de los datos a través de un puntero, sino sobre los datos ya almacenados en una variable. Si por ejemplo antes de esto nosotros cerramos el statement, ya tenemos los datos en $resultado y podremos leerlos. En `fetch` si cerramos el statement no podremos leer los datos.

## Ejemplo de inserción múltiple y visualización en HTML

En esta sección, veremos un ejemplo en el que insertamos varios registros en una base de datos utilizando un bucle y luego los mostramos en una página HTML.

### 1. Crear la tabla en la base de datos

Antes de insertar registros, necesitamos una tabla en la base de datos. Supongamos que tenemos la siguiente estructura en MySQL:

```sql
CREATE TABLE alumnos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50),
    apellidos VARCHAR(50),
    edad INT
);
```

### 2. Insertar varios registros mediante un bucle en PHP

El siguiente código muestra cómo insertar varios registros en la tabla `alumnos` utilizando un bucle con PDO:

```php
function insertMultipleRecords($dbh) {
    $alumnos = [
        ['nombre' => 'Juan', 'apellidos' => 'Pérez', 'edad' => 20],
        ['nombre' => 'Ana', 'apellidos' => 'García', 'edad' => 22],
        ['nombre' => 'Luis', 'apellidos' => 'Martínez', 'edad' => 21]
    ];

    $stmt = $dbh->prepare("INSERT INTO alumnos (nombre, apellidos, edad) VALUES (:nombre, :apellidos, :edad)");
    
    foreach ($alumnos as $alumno) {
        $stmt->execute($alumno);
    }
}
```

### 3. Mostrar los registros en una página HTML

Una vez insertados los registros, podemos recuperarlos y mostrarlos en una página web:

```php
function fetchAllRecords($dbh) {
    $stmt = $dbh->query("SELECT * FROM alumnos");
    return $stmt->fetchAll(PDO::FETCH_ASSOC);
}

$alumnos = fetchAllRecords($dbh);
```

A continuación, mostramos los registros en una tabla HTML:

```php
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Lista de Alumnos</title>
</head>
<body>
    <h2>Lista de Alumnos</h2>
    <table border="1">
        <tr>
            <th>ID</th>
            <th>Nombre</th>
            <th>Apellidos</th>
            <th>Edad</th>
        </tr>
        <?php foreach ($alumnos as $alumno): ?>
            <tr>
                <td><?php echo htmlspecialchars($alumno['id']); ?></td>
                <td><?php echo htmlspecialchars($alumno['nombre']); ?></td>
                <td><?php echo htmlspecialchars($alumno['apellidos']); ?></td>
                <td><?php echo htmlspecialchars($alumno['edad']); ?></td>
            </tr>
        <?php endforeach; ?>
    </table>
</body>
</html>
```

### Explicación del código

1. **Insertar registros**: Se usa `prepare()` y `execute()` dentro de un bucle para insertar varios registros de forma segura.
2. **Recuperar registros**: Se usa `fetchAll(PDO::FETCH_ASSOC)` para obtener todos los registros como un array asociativo.
3. **Mostrar en HTML**: Se genera una tabla en la que iteramos sobre los datos obtenidos para mostrarlos en pantalla.

Este método asegura una manipulación segura de datos y evita problemas de seguridad como la inyección SQL.

## Uso de transacciones en PDO

Las transacciones permiten agrupar varias operaciones en una sola unidad de trabajo, asegurando que todas se completen correctamente o ninguna se aplique.

### 1. Iniciar, confirmar y deshacer transacciones

```php
function transactionExample($dbh) {
    try {
        $dbh->beginTransaction();

        $stmt = $dbh->prepare("INSERT INTO alumnos (nombre, apellidos, edad) VALUES (:nombre, :apellidos, :edad)");

        $stmt->execute(['nombre' => 'Carlos', 'apellidos' => 'Fernández', 'edad' => 23]);
        $stmt->execute(['nombre' => 'Lucía', 'apellidos' => 'Gómez', 'edad' => 21]);

        // Confirmar la transacción
        $dbh->commit();
    } catch (Exception $e) {
        // Deshacer la transacción en caso de error
        $dbh->rollBack();
        echo "Error: " . $e->getMessage();
    }
}
```

### Explicación
1. **`beginTransaction()`**: Inicia la transacción.
2. **Ejecución de consultas**: Se ejecutan varias consultas dentro de la transacción.
3. **`commit()`**: Confirma los cambios si todas las consultas son exitosas.
4. **`rollBack()`**: Revierte los cambios si ocurre un error.

El uso de transacciones es ideal para garantizar la integridad de los datos en operaciones que involucran varias consultas dependientes entre sí.

## Licencia: CC BY-SA

Puedes utilizar esta guía para lo que quieras. El objetivo de esta guía es ayudarte a tí y a todas las personas que lo necesiten a utilizar PDO de forma correcta. Puedes utilizar este contenido de la forma que lo creas conveniente, siempre y cuando cites al autor.

Happy coding!



