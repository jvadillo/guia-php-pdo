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
