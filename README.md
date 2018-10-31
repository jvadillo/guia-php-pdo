
# PDO Cheatsheet

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
$sth= $dbh->("INSERT INTO alumnos(nombre, apellidos) values (:nombre, :apellidos)");
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
$sth= $dbh->("INSERT INTO alumnos(nombre, edad) values (:nombre, :edad)");
$stmt->execute($data);
```

## Tratar los resultados de una consulta SELECT

Una vez ejecutada la sentencia `execute()` podremos acceder a los resultados obtenidos de la base de datos. PDO nos ofrece la posiblidad de recibir los resultados en distintos formatos: objetos de una clase, arrays asociativos, etc. Para indicarle cómo queremos recoger los resultados utilizaremos el método `setFetchMode(String mode)`. 

Estos son los 3 valores más utilizados y que nos servirán para cubrir prácticamente todas nuestras necesidades:

 - PDO::FETCH_ASSOC: returns an array indexed by column name
 - PDO::FETCH_CLASS: Assigns the values of your columns to properties of the named class. It will create the properties if matching properties do not exist.
 - PDO::FETCH_OBJ: returns an anonymous object with property names that correspond to the column names

Una vez indicado el cómo queremos los datos, utilizaremos el método `fetch()` para acceder a la información. El método `fetch()` obtiene la siguiente fila de un conjunto de resultados, por lo que podremos iterar por los resultados tal y como se muestra en los ejemplos siguientes:

```php
function fetchAssoc(){
	// Este tipo de fetch crea un array asociativo, indexado por el nombre de la columna.
	
	$data = array( 'nombre' => 'Mikel', 'edad' => 15 );
	$stmt = $dbh->prepare("SELECT nombre, apellidos FROM alumnos WHERE nombre = :nombre AND edad = :edad");
	// Establecemos el modo en el que queremos recibir los datos
	$sth->setFetchMode(PDO::FETCH_ASSOC);
	// Ejecutamos la sentencia
	$stmt->execute($data);
	// Mostramos los resultados obtenidos
	while($row = $sth->fetch()) {
		echo $row['nombre'] . "\n";
		echo $row['apellidos'] . "\n";
		echo $row['edad '] . "\n";
	}
}

function fetch_obj(){
	// Este método crea un objeto por cada fila obtenida de la base de datos.

	$data = array( 'nombre' => 'Mikel', 'edad' => 15 );
	$stmt = $dbh->prepare("SELECT nombre, apellidos FROM alumnos WHERE nombre = :nombre AND edad = :edad");
	// Establecemos el modo en el que queremos recibir los datos
	$sth->setFetchMode(PDO::FETCH_OBJ);
	// Ejecutamos la sentencia
	$stmt->execute($data);
	
	// Mostramos los resultados obtenidos
	while($row = $STH->fetch()) {
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
	$sth->setFetchMode(PDO::FETCH_CLASS, 'Alumno');
	// Ejecutamos la sentencia
	$stmt->execute($data);

	// Mostramos los resultados
	while($obj = $sth->fetch()) {
		echo $obj->addr;
	}
}

```
## Método abreviado query()
En consultas que no reciban parámetros, podemos utilizar el método abreviado `query()` el cual ejecutará la sentencia y nos devolverá el conjunto de resultados directamente. En otras palabras, no es necesario hacer la operación en 2 pasos (`prepare()` y `execute()`) como hacíamos hasta ahora.

```php
	$sth = $dbh->query('SELECT nombre, apellidos, edad from empleado');

	// Establecemos el modo en el que queremos recibir los datos
	$sth->setFetchMode(PDO::FETCH_ASSOC);

	while($row = $sth->fetch()) {
		echo $row['nombre'] . "\n";
		echo $row['apellidos'] . "\n";
		echo $row['edad '] . "\n";
	}
```
Por razones de seguridad (evitar [SQL Injection](https://es.wikipedia.org/wiki/Inyecci%C3%B3n_SQL)) es recomendable evitar el método `query()` cuando la sentencia incluya valores variables. Por razones de rendimiento también se recomienda utilizar `prepare()` y `execute()` en sentencias que vayan a ejecutarse varias veces.

## Método fetchObject()
Existe una alternativa al método `fetch()` la cual devolverá los resultados cómo objetos anónimos (**`PDO::FETCH_OBJ`** ) u objetos de la clase indicada ( **`PDO::FETCH_CLASS`**). Este método se llama `fetchObject()`.

```php
	$sth = $dbh->query('SELECT nombre, apellidos, edad from empleado');

	while($persona = $sth ->fetchObject()) {
		echo $persona->nombre;
		echo $persona->apellido;
	}
```
En caso de que queremos que los objetos pertenezcan a una clase en concreto, es suficiente con indicárselo en la llamada:

```php
	$sth = $dbh->query('SELECT nombre, apellidos, edad from empleado');

	while($persona = $sth ->fetchObject('Alumno')) {
		echo $persona->nombre;
		echo $persona->apellido;
	}
```

## Obtener todos los resultados con fetchAll()
A diferencia del método `fetch()`, `fetchAll()` te trae todos los datos de golpe, sin abrir ningún puntero, almacenándolos en un array. Se recomienda cuando no se esperan demasiados resultados que podrían provocar problemas de memoria al querer guardar de golpe en un array miles de filas provenientes de un SELECT.

```php
	// En este caso $resultado será un array asociativo con todos los datos de la base de datos
	$resultado = $sth->fetchAll(PDO::FETCH_ASSOC);
	
	// Para leer las filas podemos recorrer el array y acceder a la información.
	foreach ($resultado as $row){
	    echo $row["nombre"]." ".$row["apellido"].PHP_EOL;
	}
```
Es casi idéntico que en `fetch()`, sólo que aquí no estamos actuando sobre un recorrido de los datos a través de un puntero, sino sobre los datos ya almacenados en una variable. Si por ejemplo antes de esto nosotros cerramos el statement, ya tenemos los datos en $resultado y podremos leerlos. En `fetch` si cerramos el statement no podremos leer los datos.

## Licencia

Puedes utilizar esta guía para lo que quieras. El objetivo de esta guía es ayudarte a tí y a todas las personas que lo necesiten a utilizar PDO de forma correcta. Puedes utilizar este contenido de la forma que lo creas conveniente, sin ser necesario citar al autor o el origen de la fuente (aunque se agradece ;-)

Happy coding!



