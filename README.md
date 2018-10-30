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

```php
function fetch_assoc(){
	#This fetch type creates an associative array, indexed by column name.

	# using the shortcut ->query() method here since there are no variable

	# values in the select statement.

	$STH = $DBH->query('SELECT name, addr, city from folks');

	# setting the fetch mode
	$STH->setFetchMode(PDO::FETCH_ASSOC);

	while($row = $STH->fetch()) {
		echo $row['name'] . "\n";
		echo $row['addr'] . "\n";
		echo $row['city'] . "\n";
	}
}

function fetch_obj(){

	#This fetch type creates an object of std class for each row of fetched data.

	# creating the statement

	$STH = $DBH->query('SELECT name, addr, city from folks');

	# setting the fetch mode

	$STH->setFetchMode(PDO::FETCH_OBJ);

	# showing the results

	while($row = $STH->fetch()) {
		echo $row->name . "\n";
		echo $row->addr . "\n";
		echo $row->city . "\n";
	}

}

function fetch_class(){

	#This fetch method allows you to fetch data directly into a class of your choosing.

	#the properties of your object are set BEFORE the constructor is called.

	#If properties matching the column names do not exist, those properties will be created (as public) for you.

	#So if your data needs any transformation after it comes out of the database,

	# it can be done automatically by your object as each object is created. (via __consruct() method )

	class secret_person {

		public $name;
		public $addr;
		public $city;
		public $other_data;

		function __construct($other = '') {
			// will be called after object has valid data in its properties
			$this->address = preg_replace('/[a-z]/', 'x', $this->address);
			$this->other_data = $other;
		}

	}

	$STH = $DBH->query('SELECT name, addr, city from folks');

	$STH->setFetchMode(PDO::FETCH_CLASS, 'secret_person');

	while($obj = $STH->fetch()) {
		echo $obj->addr;
	}
}

```

## Licencia

El objetivo de esta guía es ayudarte a utilizar PDO de forma correcta. Puedes utilizar este contenido de la forma que lo creas conveniente, sin ser necesario citar al autor (aunque se agradece ;-)



