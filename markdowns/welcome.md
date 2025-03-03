# Bases de Datos Relaciones y SQL con MySQL
::: Clase 1: Introducción a bases de datos relacionales

En esta sección encontrarás ejemplos de consultas SQL, clasificas en los 3 tipos que existen, DDL, DCL y DML.
Los queries estan preparados para funcionar en una base de datos **MySQL**, puedes copiar los queries y ejecutarlos en tu propia base de datos.

Vamos a crear una base de datos llamada Autenticación, la cual tendra el siguiente diagrama ER (Entidad Relacion).

![ER](https://raw.githubusercontent.com/mayracmg/playground-sql-facilito/master/markdowns/Autenticacion%20ER.png)

::: DDL (Data Definition Language)
Creación de una base de datos llamada Autenticación.

```sql
CREATE DATABASE Autenticacion;
```

## Creación de las tablas del diagrama ER
::: Scripts
La tabla **Usuario** tendrá como llave primaria el campo ID, los campos ID y Nombre tendran un **constraint** de tipo **NOT NULL** el cual sirve para garantizar que esos campos siempre tendrán valor.

La sentencia **AUTO_INCREMENT** sirve para indicar que el campo ID, será un correlativo automatico, manejado por la base de datos, es decir no es necesario especificar el valor cuando se inserten valores.

La sentencia **PRIMARY KEY** sirve para indicar que campo o campos serán utilizados como llave primaria.

+ El tipo de dato INT permite almacenar números desde -2147483648 hasta 2147483647.
+ El tipo de dato VARCHAR permite almacenar una cadena de caracteres de longitud 50 y/o 100 segun se especifique.
+ El tipo de dato DATE permite almacenar fechas (sin hora).
+ El tipo de dato TINYINT permite almacenar números desde -128 hasta 127.
+ **ALTER TABLE** permite modificar la tabla Usuario y agregarle un contraint del tipo **UNIQUE** de esa forma, los valores almacenados en el campo Email no podrán repetirse.
```sql
CREATE TABLE Usuario (
	ID INT NOT NULL AUTO_INCREMENT, 
	Nombre VARCHAR(50) NOT NULL, 
	Apellido VARCHAR(50), 
	Email VARCHAR(100), 
	Fecha_Nacimiento DATE,
	Activo TINYINT,
	PRIMARY KEY (ID)
);
ALTER TABLE Usuario ADD CONSTRAINT UK_Email UNIQUE (Email);
```
___
La tabla **Historial_Conexion** tendrá como llave primaria el campo ID, los campos ID, ID_Usuario y Fecha_Hora tendrán un **constraint** de tipo **NOT NULL** el cual sirve para garantizar que esos campos siempre tendrán valor.

La sentencia **AUTO_INCREMENT** sirve para indicar que el campo ID, será un correlativo automatico, manejado por la base de datos, es decir no es necesario especificar el valor cuando se inserten valores.

La sentencia **PRIMARY KEY** sirve para indicar que campo o campos serán utilizados como llave primaria.

La sentencia **FOREIGN KEY** sirve parea indicar que campo o campos será uitilizados como llave foranea o referencia a otra tabla. El campo ID_Usuario corresponde al campo ID de la tabla Usuario.
+ El tipo de dato DATETIME permite almacenar fecha y hora.
```sql
CREATE TABLE Historial_Conexion (
	ID INT NOT NULL AUTO_INCREMENT,
	ID_Usuario INT NOT NULL,
	Fecha_Hora DATETIME NOT NULL,
	IP VARCHAR(15), 
	Navegador VARCHAR(50), 
	PRIMARY KEY (ID),
	FOREIGN KEY (ID_Usuario) REFERENCES Usuario(ID)
);
```
---
La tabla **Menu** tendrá como llave primaria el campo ID, los campos ID, Titulo y Activo tendrán un **constraint** de tipo **NOT NULL** el cual sirve para garantizar que esos campos siempre tendrán valor.

El campo Activo tendrá un **contraint** de tipo **DEFAULT** el cúal asignará el valor 0 de forma predeterminada cuando en la instrucción insert no se especifique el campo Activo. Ademas el campo Activa tendrá un **CHECK CONTRAINT** el cúal solo permitirá que el campo acepte los valores 0 o 1, ningún otro valor podrá ser almacenado en ese campo.

La sentencia **AUTO_INCREMENT** sirve para indicar que el campo ID, será un correlativo automatico, manejado por la base de datos, es decir no es necesario especificar el valor cuando se inserten valores.

La sentencia **PRIMARY KEY** sirve para indicar que campo o campos serán utilizados como llave primaria.

```sql
CREATE TABLE Menu (
	ID INT NOT NULL AUTO_INCREMENT,
	Titulo VARCHAR(50) NOT NULL, 
	Descripcion VARCHAR(150), 
	URL VARCHAR(150), 
	Activo TINYINT DEFAULT 0 NOT NULL,
	PRIMARY KEY (ID),
	CONSTRAINT CHK_Menu_Activo CHECK (Activo IN (0, 1))
);
```
---
El script de creación de la tabla **Menu_Usuario** no contiene ningun contraint, ni llave primaria y el tipo de dato DATE "es incorrecto". La tabla será modificada con scripts separados para llegar a la estructura correcta.

+ La sentencia **ALTER TABLE** permite modificar la estructura de la tabla Menu_Usuario.
+ **MODIFY** permite modificar el tipo de dato del campo Fecha_Habilitado.
+ **ADD CONSTRAINT** permite agregar la llave primaria a la tabla, entre los parentesis se agregan los 2 campos ID_Usuario y ID_Menu, que juntos forman la llave primaria.
+ **ADD FOREIGN KEY** permite agregar la llave foranea a la tabla, se crean dos queries debido a que son dos refencias a dos tablas disintas.

```sql
CREATE TABLE Menu_Usuario (
	ID_Usuario INT,
	ID_Menu INT,
	Fecha_Habilitado DATE
);

ALTER TABLE Menu_Usuario MODIFY Fecha_Habilitado DATETIME;
ALTER TABLE Menu_Usuario ADD CONSTRAINT PK_Menu_Usuario PRIMARY KEY (ID_Usuario, ID_Menu);
ALTER TABLE Menu_Usuario ADD FOREIGN KEY (ID_Usuario) REFERENCES Usuario(ID);
ALTER TABLE Menu_Usuario ADD FOREIGN KEY (ID_Menu) REFERENCES Menu(ID);
```
[Ver Diagrama ER con scripts](https://github.com/mayracmg/playground-sql-facilito/blob/f5753c08814bb14e3ecf1db49f46995fde1dbcc9/markdowns/ER%20con%20Scripts.png)
:::

## Creación de otros objetos
::: Triggers
Script para la _creación_ de un _trigger_ el cual va a ejecutar automaticamente justo _antes de insertar_ un registro en la _tabla Usuario_, va a verificar si en el registro que se esta insertado, el campo Email esta vacio (NULL) o no, si esta vacio entonces va a asignarle un estado inactivo (Activo = 0).
+ **DELIMITER** Sirve para indicarle a MySQL donde finaliza el bloque de código.
```sql
DELIMITER $$

CREATE TRIGGER BI_Usuario
    BEFORE INSERT ON Usuario 
	FOR EACH ROW
BEGIN
    IF (NEW.Email IS NULL) THEN
		SET NEW.Activo = 0;
	ELSE
		SET NEW.Activo = 1;
	END IF;
END$$
```

El funcionamento del trigger puede ser probado con la ejecución de estos scripts. Al no especificar el campo Email, el trigger va a asignar un valor 0 al campo Activo. Cuando el campo Email si se especifica entonces el campo Activo tendrá valor 1.
```sql
INSERT INTO Usuario (Nombre, Apellido)
VALUES ('Tu_Nombre', 'Tu_Apellido')

INSERT INTO Usuario (Nombre, Apellido, Email)
VALUES ('Tu_Nombre', 'Tu_Apellido', 'nombre@gmail.com')
```
Luego se comprobar el resultado con un query **SELECT** en la tabla Usuario.
```sql
SELECT *
FROM Usuario;
```
El cual generará un resultado similar al siguiente:

| ID | Nombre | Apellido | Email | Fecha_Nacimiento | Activo |
| ------ | ----------- | ----------- | ----------- | ----------- | ----------- |
| 1   | Tu_Nombre | Tu_Apellido | NULL | NULL | 0 |
| 2 | Tu_Nombre | Tu_Apellido | nombre@gmail.com | NULL | 1 |

:::

::: Funciones

Script para la _creación_ de una _función_ la cual va a concatenar el nombre y apellido del usuario en un solo campo y lo convertirá a mayusculas. El _parametro de entrada_ es el ID de la tabla Usuario. 

La función podrá ser invocada desde cualquier consulta DML, por ejemplo una consulta <span style="color:blue">SELECT</span>.

+ **DELIMITER** Sirve para indicarle a MySQL donde finaliza el bloque de código.
+ **DROP FUNCTION IF EXISTS** borrará la función si es que ya existe, para luego crearla nuevamente. Sin esta instrucción, el script **CREATE FUNCTION** solo funcionaría una vez y para modificaciones posteriores sería necesario utilizar un **ALTER FUNCTION**.
+ **RETURNS VARCHAR(101)** indica que el tipo de dato del resultado de la función será una cadena de texto de longitud maxima 101 caracteres.
+ **READS SQL DATA** sin esta instrucción la instrucción <span style="color:blue">SELECT</span> daría error.
+ El parametro de entrada va dentro de parentesis justo despues del nombre la función.
+ **DECLARE** permite declarar la variable que será utilizada para almacenar el nombre completo del usuario.
+ **CONCAT** concatena el primer nombre, un espacio en blanco y el appelido del usuario en un solo campo.
+ **UPPER** convierte el resultado de la concatenación en mayusculas.
+ **INTO** permiate almacenar el resultado del select en la variable declarada.
+ **RETURN** la última instrucción de una función debe ser devolver el resultado generado.
```sql
DELIMITER $$

DROP FUNCTION IF EXISTS Fn_Nombre_Completo;
CREATE FUNCTION Fn_Nombre_Completo(
    ID_Usuario INT
)
RETURNS VARCHAR(101)
READS SQL DATA
BEGIN
	DECLARE Nombre_Completo VARCHAR(101);
	
	SELECT UPPER(CONCAT(Nombre, ' ', Apellido))
	INTO Nombre_Completo
	FROM Usuario
	WHERE ID = ID_Usuario;

	RETURN Nombre_Completo;
END $$
```
En el siguiente ejemplo, la función es invocada desde la instrucción <span style="color:blue">SELECT</span> y el resultado se verá desplegado como cualquier campo.
+ _Nombre_Completo_ es un alias, los alias son opcionales, pero mejoran la visualización de los datos.
```sql
SELECT ID, Fn_Nombre_Completo(id) Nombre_Completo
FROM Usuario;
```
El cual generará un resultado similar al siguiente:

| ID | Nombre_Completo |
| ------ | ----------- |
| 1   | MAY CODE |
:::

::: Procedimientos (Stored Procedures)
Script para la creación de un procedimiento, parentesis vacios indica que no recibe parametros. Cada vez que se ejecute, leerá la tabla _Historial_conexion_, filtará las filas que tengan el campo _IP_ con valor _NULL_ y les asignará el valor '0.0.0.0'.
+ **DELIMITER** Sirve para indicarle a MySQL donde finaliza el bloque de código.
+ **DROP PROCEDURE IF EXISTS** borrará el procedimiento si es que ya existe, para luego crearla nuevamente. Sin esta instrucción, el script **CREATE PROCEDURE** solo funcionaría una vez y para modificaciones posteriores sería necesario utilizar un **ALTER PROCEDURE**.

```sql
DELIMITER //

DROP PROCEDURE IF EXISTS SP_Actualizar_IP_Vacia;
CREATE PROCEDURE SP_Actualizar_IP_Vacia()
BEGIN
	UPDATE Historial_conexion
	SET IP = '0.0.0.0'
	WHERE IP IS NULL;
END //
```
+ **CALL** ejecuta el procedimiento, entre los parentesis se especificarían los parametros, si los hubiera.

```sql
CALL SP_Actualizar_IP_Vacia();
```
:::
:::

::: DCL (Data Control Language)

Creación de 3 usuarios, el proposito del usuario _WebSite_ es ser utilizado desde un API o sistema que se conecte a la base de datos.
El proposito de los usuarios _Developer1_ y _Developer2_ es ser utilizados por 2 personas distintas y de acuerdo a sus responsabilidades serán asignados sus permisos.
*CREATE USER* son sentencias DDL, es necesario que existan usuarios previo a asignarles permisos.
```sql
CREATE USER WebSite IDENTIFIED BY 'Contraseña#1';
CREATE USER Developer1 IDENTIFIED BY 'Contraseña#2';
CREATE USER Developer2 IDENTIFIED BY 'Contraseña#3';
```

## Asignación de permisos:
+ Al usuario _WebSite_, el usuario tendrá todos los permisos sobre las tablas, excepto la tabla _Usuario_, sobre la tabla _Usuario_ no podrá eliminar usuario.
+ Al usuario _Developer1_ se la asignará todos los permisos sobre las tablas _Menu_ y _Menu_Usuario_, sobre las tablas _Usuario_ y _Historial_Conexion_ solo tendrá permiso de lectura, para evitar que Developer1 haga modificaciones sobre esas tablas.
+ Al usuario _Developer2_ solo se le asignará permiso de lectura sobre las tablas _Menu_ y _Menu_Usuario_, sobre las tablas _Usuario_ y _Historial_Conexion_ no tendrá ningun acceso.

+ <span style="color:blue">GRANT</span> sentencia para otorgar permisos.
+ <span style="color:blue">SELECT</span> para asignar permiso de lectura sobre la tabla especificada.
+ <span style="color:blue">INSERT</span> para asignar permiso de inserción sobre la tabla especificada.
+ <span style="color:blue">UPDATE</span> para asignar permiso de actualización sobre la tabla especificada.
+ <span style="color:blue">DELETE</span> para asignar permiso de eliminación sobre la tabla especificada.
+ <span style="color:blue">ON</span> luego de este texto se indica el nombre de la tabla a la cual se le quiere asignar los permisos.
+ <span style="color:blue">TO</span> luego de este texto se indica el nombre del usuario al cual se le quiere asignar los permisos.

```sql
GRANT SELECT, INSERT, UPDATE ON Usuario TO WebSite;
GRANT SELECT, INSERT, UPDATE, DELETE ON Historial_Conexion TO WebSite;
GRANT SELECT, INSERT, UPDATE, DELETE ON Menu TO WebSite;
GRANT SELECT, INSERT, UPDATE, DELETE ON Menu_Usuario TO WebSite;

GRANT SELECT ON Usuario TO Developer1;
GRANT SELECT ON Historial_Conexion TO Developer1;
GRANT SELECT, INSERT, UPDATE, DELETE ON Menu TO Developer1;
GRANT SELECT, INSERT, UPDATE, DELETE ON Menu_Usuario TO Developer1;

GRANT SELECT ON Menu TO Developer2;
GRANT SELECT ON Menu_Usuario TO Developer2;
```

+ Al usuario _WebSiste_ se le quitará el acceso de eliminar sobre la tabla _Menu_.

+ <span style="color:blue">REVOKE</span> sentencia para revocar o quitar permisos.
+ <span style="color:blue">SELECT</span> para quitar permiso de lectura sobre la tabla especificada.
+ <span style="color:blue">INSERT</span> para quitar permiso de inserción sobre la tabla especificada.
+ <span style="color:blue">UPDATE</span> para quitar permiso de actualización sobre la tabla especificada.
+ <span style="color:blue">DELETE</span> para quitar permiso de eliminación sobre la tabla especificada.
+ <span style="color:blue">ON</span> luego de este texto se indica el nombre de la tabla a la cual se le quiere quitar los permisos.
+ <span style="color:blue">FROM</span> luego de este texto se indica el nombre del usuario al cual se le quiere quitar los permisos.
```sql
REVOKE DELETE ON Menu FROM WebSite;
```
:::

::: DML (Data Manipulation Language) 
::: SELECT

Queries que permiten leer todos los campos para cada una de la tablas creadas previamente.
+ <span style="color:blue">SELECT *</span> Indica que se leerán todos los campos de la tabla.
+ <span style="color:blue">FROM</span> Luego de este texto se especifica el nombre de la tabla (o tablas) a leer.

```sql
SELECT *
FROM Usuario;

SELECT *
FROM Historial_Conexion;

SELECT *
FROM Menu;

SELECT *
FROM Menu_Usuario;
```

+ <span style="color:blue">SELECT Nombre, Apellido</span> para leer unicamente los campos Nombre y Apellido. Los campos se separan por comas.

```sql
SELECT Nombre, Apellido
FROM Usuario;
```

+ <span style="color:blue">Nombre Primer_Nombre</span> _Nombre_ es el nombre del campo en la tabla, _Primer_Nombre es un *alias* para el campo, los alias es otra forma de referise al campo, son opcionales.
+ <span style="color:blue">Usuario U</span> La letra _U_ es un alias para la tabla _Usuario_, los alias son muy comunes cuando en una misma consulta hay varias tablas.
+ <span style="color:blue">WHERE</span> Aca se especifican los filtros, para solo obtener las filas que cumplan las condiciones especificadas.
+ **Activo = 1** para filtrar solo las filas que en la columna _Activo_ tengan valor igual a 1.
+ <span style="color:blue">AND</span> para unir varios filtros dentro de la clausula <span style="color:blue">WHERE</span>.
+ **Fecha_Nacimiento IS NULL** para filtrar solo las filas que en la columna Fecha_Nacimiento tengan valor igual a _NULL_.

```sql
SELECT Nombre Primer_Nombre, Apellido
FROM Usuario U
WHERE Activo = 1
AND Fecha_Nacimiento IS NULL;
```

+ <span style="color:blue">ORDER BY</span> Luego de este texto se especifica el nombre del campo (o campos) para aplicar el ordenamiento.
Se leer el hsitorial de conexion ordenados por dirección IP de forma ascendente.
```sql
SELECT *
FROM Historial_Conexion
ORDER BY IP;
```

+ <span style="color:blue">DESC</span> Indica que el ordenamiento será descendente.
+ <span style="color:blue">ASC</span> Indica que el ordenamiento será ascendente, si no se especifica **ASC** o **DESC**, por default el ordenamiento será ascendente.
Se leer el hsitorial de conexion ordenados por Fecha y Hora de forma descendente.
```sql
SELECT *
FROM Historial_Conexion
ORDER BY Fecha_Hora DESC;
```

Podemos leer por ejemplo unicamente 3 filas de la tabla _Menu_ por medio de la instrucción <span style="color:blue">LIMIT</span>.
```sql
SELECT *
FROM Menu
LIMIT 3;
```

+ <span style="color:blue">CREATE VIEW</span> Para crear una vista, luego de <span style="color:blue">AS</span> se coloca el query de tipo <span style="color:blue">SELECT</span> que es lo que la vista mostrará.
+ La vista se puede utilizar de la misma forma que cualquier tabla. Se le puede agregar mas filtros en el <span style="color:blue">WHERE</span> si asi se deseara.
```sql
CREATE VIEW Usuarios_Activos AS
    SELECT *
    FROM Usuario
    WHERE Activo = 1;

SELECT *
FROM Usuarios_Activos;
```
:::

::: INSERT
Distintos ejemplos de como usar la inserción de datos.

+ <span style="color:blue">INSERT INTO</span> Luego de este texto debe de ir el nombre de la tabla.
+ **Entre los parentesis** se incluye la lista de campos a insertar.
+ <span style="color:blue">VALUES</span> Luego de este texto deben de ir los valores de los campos que se especificarón antes, en el mismo orden.
+ El campo _ID_ no se especifica, entonces la clausula <span style="color:blue">AUTO_INCREMENT</span> creará el valor correspondiente de form automatica.

```sql
INSERT INTO Menu 
(Titulo, Descripcion, URL, Activo)
VALUES ('Configuración', 'Menu de configuración', '/Configuración', 1);
```

+ El campo _ID_ si se especifica con un valor que ya existe, por lo que se generará un <span style="color:red">error</span>.
```sql
INSERT INTO Menu 
(ID, Titulo, Descripcion, URL, Activo)
VALUES (1, 'Contraseña', 'Configuración de contraseña', '/Configuración/Contraseña', 1);
```

+ El campo _ID_ si se especifica, entonces la clausula se guardará con ese valor. No se tomará en cuenta el <span style="color:blue">AUTO_INCREMENT</span>.
```sql
INSERT INTO Menu 
(ID, Titulo, Descripcion, URL, Activo)
VALUES (5, 'Contraseña', 'Configuración de contraseña', '/Configuración/Contraseña', 1);
```

+ Como se estan insertando valores para **todas las columnas** de la tabla, no es obligatorio especificar la lista de campos.
+ Los valores deben de ir en el mismo orden en que se especificaron los campos en el <span style="color:blue">CREATE TABLE</span>.

```sql
INSERT INTO Menu 
VALUES (3, 'Tema', 'Configuración de tema oscuro o claro', '/Configuración/Tema', 1);
```

+ Como se estan insertando un valor **5** en el campo _Activo_ el cúal no es valido segun el <span style="color:blue">CHECK CONSTRAINT</span> que se creó, el query dará un <span style="color:red">error</span>.
```sql
INSERT INTO Menu 
(Titulo, Descripcion, URL, Activo)
VALUES ('Tipo Cuenta', 'Configuración de tipo de cuenta', '/Configuración/TipoCuenta', 5);
```

+ Luego de corregir el valor del campo _Activo_ el query debera ejecutarse satisfactoriamente.
```sql
INSERT INTO Menu 
(Titulo, Descripcion, URL, Activo)
VALUES ('Tipo Cuenta', 'Configuración de tipo de cuenta', '/Configuración/TipoCuenta', 0);
```

Creación de una tabla temporal, la cual será una copia exacta de la tabla _Menu_, sobre la tabla temporal se puede hacer cualquier operación, podemos eliminar la tabla con <span style="color:blue">DROP TABLE</span>, o al finalizar la sesión, en automatico la tabla será eliminada.
+ Inserción de 2 filas nuevas, sobre la tabla temporal Menu_Copia.

```sql
CREATE TEMPORARY TABLE Menu_Copia SELECT * FROM Menu;

INSERT INTO Menu_Copia 
(Titulo, Descripcion, URL, Activo)
VALUES ('Asociar Facebook', 'Asociar cuenta de Facebook', '/Configuración/Facebook', 1);

INSERT INTO Menu_Copia 
(Titulo, Descripcion, URL, Activo)
VALUES ('Asociar Google', 'Asociar cuenta de Google', '/Configuración/Google', 1);
```

+ La tabla Menu_Copia contiene todos los valores de la tabla _Menu_ y adicional las 2 filas recien insertadas.
+ Se filtran las filas que el texto del campo _Titulo_ comienza con el texto _Asociar_, para eso se usa la clasula <span style="color:blue">LIKE</span>.
+ <span style="color:blue">%</span> Indica que alli puede ir cualquier texto.
```sql
SELECT Titulo, Descripcion, URL, Activo
FROM Menu_Copia
WHERE Titulo LIKE 'Asociar%'
```

+ Ahora la clausula <span style="color:blue">INSERT</span> ya no contiene la instrucción <span style="color:red">VALUES</span>, en lugar de eso, se insertarán todas filas que sean resultado de la clasula <span style="color:blue">SELECT</span>.
```sql
INSERT INTO Menu (Titulo, Descripcion, URL, Activo)
SELECT Titulo, Descripcion, URL, Activo
FROM Menu_Copia
WHERE Titulo LIKE 'Asociar%'
```
:::
::: UPDATE

Actualización de todos los registros de la tabla _Menu_, el campo _Activo_ se le asigna el valor 1.
```sql
UPDATE Menu
SET Activo = 1;
```

Actualización de todos los registros de la tabla _Menu_, el campo _Activo_ se le asigna el valor 0 y el campo _URL_ se cambian todos sus caracteres a minusculas con la funcion <span style="color:blue">LOWER</span>.
```sql
UPDATE Menu
SET Activo = 0, URL = LOWER(URL);
```

Actualización de un único registro de la tabla _Menu_, el campo _Activo_ se le asigna el valor 1 solo para la fila que tiene el campo _ID_ igual a 1.
```sql
UPDATE Menu
SET Activo = 1
WHERE ID = 1;
```

Actualización de ningun registro de la tabla _Menu_, el campo _Activo_ se trata de asignar el valor 1 solo para la fila que tiene el campo _ID_ igual a 500, pero no hay ninguna fila con ese valor, el query se ejecuta satisfactoriamente dando como resultado ninguna fila actualizada.
```sql
UPDATE Menu
SET Activo = 1
WHERE ID = 500;
```

Actualización de varios registros de la tabla _Menu_, el campo _Activo_ se le asigna el valor 1 solo para la fila que el campo _ID_ esta contenido en la lista de valores (2, 3, 4).
```sql
UPDATE Menu
SET Activo = 1
WHERE ID IN (2, 3, 4)
```

Actualización de todos los registros de la tabla _Menu_, al campo _Activo_ se le trata de asignar el valor 5, pero como la tabla tiene un <span style="color:blue">CHECK CONSTRAINT</span> que no permite esos valores, entonces se generará un <span style="color:red">error</span>.
```sql
UPDATE Menu
SET Activo = 5;
```
:::

::: DELETE
Eliminación de todos los registros de la tabla Historial_Conexion.
<span style="color:red">*</span> Es muy importante ser cuidados al momento de ejecutar un query <span style="color:red">DELETE</span> para no eliminar registros que no se debian eliminar.
```sql
DELETE 
FROM Historial_Conexion;
```

Eliminación de todos los registros de la tabla _Historial_Conexion_.

<span style="color:red">*</span> Es muy importante ser cuidados al momento de ejecutar un query <span style="color:red">DELETE</span> para no eliminar registros que no se debian eliminar.
<span style="color:blue">WHERE 1 = 1</span> es equivalente a que no haya ninguna condición.
```sql
DELETE 
FROM Historial_Conexion;

DELETE 
FROM Historial_Conexion
WHERE 1 = 1;
```

<span style="color:blue">TRUNCATE TABLE</span> Borra todos los datos de una tabla, es parecido a ejecutar un <span style="color:blue">DROP TABLE</span> seguido de un <span style="color:blue">CREATE TABLE</span>.
+ El <span style="color:blue">DELETE</span> borrar las filas una por una, <span style="color:blue">TRUNCATE TABLE</span> borra todas las filas de un solo, por lo que es mas rapido.
```sql
TRUNCATE TABLE Historial_Conexion;
```

Eliminación de varios registros de la tabla _Menu_.
```sql
DELETE 
FROM Menu
WHERE ID >= 7
AND ID <= 8;
```
:::
:::

:::

::: Clase 2: Diseño y normalización de base de datos

::: Ejercicio
**Diseñar y normalizar la base de datos para la siguiente tabla con datos:**<br>
[Script para crear la tabla](https://github.com/mayracmg/Curso-SQL/blob/main/credit_card_fraud_detection.sql)

Crear el diagrama Entidad Relación (Modelo Físico) y generar los scripts para normalizar la BD.
:::
:::

::: Clase 3: Relaciones en base de datos
1. Descargar la base de datos de ejemplo: [Descargar](https://github.com/mayracmg/Curso-SQL/blob/main/credit_card_db.sql)
2. Copiar, pegar y ejecutar los queries en MySQL.

### Diagrama ER de la BD descargada.
![ER](https://raw.githubusercontent.com/mayracmg/playground-3s1cjv5r/refs/heads/master/CreditCardDB.png)

::: Cláusula WHERE
Define una condición (o varias) que debe cumplirse para que los datos sean devueltos.
Los operadores utilizados en la cláusula WHERE (o cualquier condición definida en la cláusula) no tienen efecto en los datos almacenados en las tablas. 
Sólo afectan a los datos devueltos cuando se invoca la vista.
Se puede incluir en una instrucción <span style="color:blue">SELECT</span>, <span style="color:blue">UPDATE</span> o <span style="color:blue">DELETE</span>.

::: Datos
![ER](https://raw.githubusercontent.com/mayracmg/playground-sql-facilito/master/markdowns/EjemploDatos1.png)
:::

::: Aplicar un filtro
![ER](https://raw.githubusercontent.com/mayracmg/playground-sql-facilito/master/markdowns/EjemploDatos2.png)
:::

::: Aplicar otro filtro
![ER](https://raw.githubusercontent.com/mayracmg/playground-sql-facilito/master/markdowns/EjemploDatos3.png)
:::

## Operadores de Comparación
+ **Típicos** (=, !=, <, <=, >, >=)
+ **AND**: Para unir dos condiciones, ambas deben ser verdaderas.
+ **OR**: Para unir dos condiciones, una condición debe ser verdadera.
+ **IS NULL**: Para obtener las filas donde X columna tiene valor null.
+ **BETWEEN**: para identificar un rango de valores.
+ **NOT**: Para negar una condición.
+ **LIKE**: es posible especificar valores que son solamente similares a los valores almacenados.
    - Signo de porcentaje (%): representa cero o más caracteres desconocidos.
    - Guión bajo (_): representa exactamente un carácter desconocido.
+ **IN**: permite determinar si los valores en la columna especificada de una tabla están contenidos en una lista definida o contenidos dentro de otra tabla.
+ **EXISTS**: Está dedicado únicamente a determinar si la subconsulta arroja alguna fila o no.

```sql
SELECT *
FROM credit_card_fraud_detection
WHERE transaction_amount BETWEEN 20 AND 200
AND (merchant_name LIKE 'A%'
	OR merchant_name LIKE '_A%')
AND country IN ('USA', 'India');
```
:::

::: SubConsultas o SubQueries
Proporcionan una forma de acceder a datos en múltiples tablas con una sola consulta. 
Puede agregarse a una instrucción <span style="color:blue">SELECT</span>, <span style="color:blue">INSERT</span>, <span style="color:blue">UPDATE</span> o <span style="color:blue">DELETE</span> para permitir a esa instrucción utilizar los resultados de la consulta arrojados por la subconsulta. 
La subconsulta es esencialmente una instrucción <span style="color:blue">SELECT</span> incrustada que actúa como una puerta de entrada a los datos en una segunda tabla. 

Se pueden en dos categorías generales:
+ Las que pueden arrojar múltiples filas
+ Las que pueden arrojar solamente un valor

Subconsulta que retorna múltiples filas.
```sql
SELECT *
FROM credit_card_fraud_detection
WHERE transaction_amount BETWEEN 20 AND 200
AND (merchant_name LIKE 'A%'
	OR merchant_name LIKE '_A%')
AND country IN (
	SELECT DISTINCT country
    FROM credit_card_fraud_detection
    WHERE user_age <= 18
);
```

Subconsultas que retornan solamente un valor.
```sql
SELECT *
FROM credit_card_fraud_detection
WHERE transaction_amount BETWEEN 20 AND 200
AND (merchant_name LIKE 'A%'
	OR merchant_name LIKE '_A%')
AND user_age > (
	SELECT AVG(user_age)
    FROM credit_card_fraud_detection
);

SELECT *, (SELECT AVG(user_age) FROM credit_card_fraud_detection) Average_age
FROM credit_card_fraud_detection;
```

El resultado de la subconsulta tambien puede ser utilizado como que fuera una tabla asignandole un alias y seleccionar datos de ese resultado.
```sql
SELECT *
FROM (
	SELECT country, AVG(user_age)
    FROM credit_card_fraud_detection
    GROUP BY country
) SubQuery;
```

Una subconsulta tambien puede tener mas subconsultas, como una cadena. <br>
**<span style="color:red">*</span>** Al utilizar una varias subconsultas es importante ser cuidados ya que puede llegar a afectar significativamente el rendimiento del query.
```sql
SELECT *
FROM (
	SELECT country, AVG(user_age)
    FROM credit_card_fraud_detection
    WHERE user_age > (
		SELECT AVG(user_age)
		FROM credit_card_fraud_detection
	)
    GROUP BY country
) SubQuery;
```
:::

::: Funciones de Agregación
Realizan operaciones sobre un grupo o un set de datos.
Comúnmente son utilizadas con la cláusula <span style="color:blue">GROUP BY</span> para generar grupos y resultados sobre esos grupos.

## Algunas funciones comunes

+ **AVG**: Para promediar valores
+ **COUNT**: Para contar registros
+ **COUNT(DISTINCT)**: Para contar registros unicos.
+ **MAX**: Devuelve el valor máximo.
+ **MIN**: Devuelve el valor mínimo.
+ **SUM**: Para sumar valores.
+ **STD**: Devuelve la desviación estándar.

Ejemplos

1. Seleccionamos el total de registros en la tabla _country_, para obtener otra métrica, solo cambiamos el COUNT por la función que necesitemos.
```sql
SELECT COUNT(*)
FROM country;
```
2. Seleccionamos el total de registros en la tabla _user_, pero en lugar de un total general, es el total agrupado por el campo _gender_id_.
```sql
SELECT gender_id, COUNT(*)
FROM user
GROUP BY gender_id;
```

## GROUP BY
Es posible utilizar más de una función de agregación en un mismo query.
Todos los campos individuales que están junto a la función de agregación en la clausula <span style="color:blue">SELECT</span> debe ir tambien en la clausula <span style="color:blue">GROUP BY</span>.
```sql
SELECT transaction_date, MIN(transaction_amount), MAX(transaction_amount), AVG(Account_balance)
FROM transaction
GROUP BY transaction_date;

SELECT country_id, isFraudulent, location_id, MIN(transaction_amount), MAX(transaction_amount), AVG(Account_balance)
FROM transaction
GROUP BY country_id, isFraudulent, location_id;
```

##  HAVING
A diferencia de la cláusula <span style="color:blue">WHERE</span>, la cláusula <span style="color:blue">HAVING</span> se refiere a grupos, no a filas individuales.
Se aplica a los resultados después de haberse agrupado (en la cláusula <span style="color:blue">GROUP BY</span>).
Tiene la ventaja de permitir el uso de funciones establecidas tales como <span style="color:blue">AVG</span> o <span style="color:blue">SUM</span>, que no se pueden utilizar en la cláusula <span style="color:blue">WHERE</span> a menos que se coloquen dentro de una subconsulta.
```sql
SELECT country_id, isFraudulent, location_id, MIN(transaction_amount), MAX(transaction_amount), AVG(Account_balance)
FROM transaction
WHERE transaction_amount > 300
GROUP BY country_id, isFraudulent, location_id
HAVING AVG(Account_balance) > 18800;
```

# ORDER BY
Toma la salida de la cláusula <span style="color:blue">SELECT</span> y ordena los resultados de la consulta de acuerdo con las especificaciones dentro de la cláusula <span style="color:blue">ORDER BY</span>.
Se especifica una o más columnas y las palabras clave opcionales <span style="color:blue">ASC</span> o <span style="color:blue">DESC</span> (una por columna). Si no se especifica la palabra clave, setoma <span style="color:blue">ASC</span>.
+ **ASC**: Orden ascendente
+ **DESC**: Orden descendente.

```sql
SELECT country_id, isFraudulent, location_id, MIN(transaction_amount), MAX(transaction_amount), AVG(Account_balance)
FROM transaction
WHERE transaction_amount > 300
GROUP BY country_id, isFraudulent, location_id
HAVING AVG(Account_balance) > 18800
ORDER BY country_id DESC, isFraudulent, MIN(transaction_amount)
```


:::

::: Joins
Un componente importante de cualquier base de datos relacional es la correlación que puede existir entre dos tablas cualesquiera. 
En SQL podemos unir las tablas en una instrucción. Una operación join es una operación que hace coincidir las filas en una tabla con las filas de manera tal que las columnas de ambas tablas puedan ser colocadas lado a lado en los resultados de la consulta como si éstos vinieran de una sola tabla.

## Tipos de joins
![Joins](https://ingenieriadesoftware.es/wp-content/uploads/2018/07/sqljoin.jpeg)

+ <span style="color:blue">INNER JOIN</span>: Devuelve registros que tienen valores coincidentes en ambas tablas
+ <span style="color:blue">LEFT JOIN</span>: Devuelve todos los registros de la tabla de la izquierda y los registros coincidentes de la tabla de la derecha.
+ <span style="color:blue">RIGHT JOIN</span>: Devuelve todos los registros de la tabla de la derecha y los registros coincidentes de la tabla de la izquierda.
+ <span style="color:blue">FULL JOIN</span>: No existe como tal en MySQL (si existe en otras bases de datos) pero se puede lograr obtener mediante la union de un <span style="color:blue">LEFT JOIN</span> y un <span style="color:blue">RIGHT JOIN</span>.
+ <span style="color:blue">CROSS JOIN</span>: Combina todas las filas de la tabla A con todas las filas de la tabla B.
+ <span style="color:purple">SELF JOIN</span>: Aplica las reglas de los joins anteriores, solo que se realiza con la misma tabla.

### Ejemplo INNER JOIN
![INNER JOIN](https://raw.githubusercontent.com/mayracmg/playground-3s1cjv5r/refs/heads/master/InnerJoin.png)

**A**: user<br>
**B**: gender
```sql
SELECT G.Name Gender, COUNT(*) users
FROM user U
INNER JOIN gender G ON G.gender_id = U.gender_id
GROUP BY G.Name;
```

Otra forma de escribirlo:
```sql
SELECT G.Name Gender, COUNT(*) users
FROM user U
JOIN gender G ON G.gender_id = U.gender_id
GROUP BY G.Name;

SELECT G.Name Gender, COUNT(*) users
FROM user U, gender G 
WHERE G.gender_id = U.gender_id
GROUP BY G.Name;
```

**A**: credit_card_fraud_detection<br>
**B**: credit_card_fraud_detection
```sql
SELECT *
FROM credit_card_fraud_detection A
INNER JOIN credit_card_fraud_detection B ON A.transaction_date = B.transaction_Date
	AND A.country = 'USA' 
    AND B.country = 'Canada';
```

### Ejemplo LEFT JOIN
![LEFT JOIN](https://raw.githubusercontent.com/mayracmg/playground-3s1cjv5r/refs/heads/master/LeftJoin.jpg)

**A**: credit_card_fraud_detection<br>
**B**: credit_card_fraud_detection
```sql
SELECT *
FROM credit_card_fraud_detection A
LEFT JOIN credit_card_fraud_detection B ON A.transaction_date = B.transaction_Date 
    AND B.country = 'Canada'
	AND A.country = 'USA' 
```

```sql
SELECT *
FROM credit_card_fraud_detection A
LEFT JOIN credit_card_fraud_detection B ON A.transaction_date = B.transaction_Date 
    AND B.country = 'Canada'
	AND A.country = 'USA' 
WHERE B.transaction_id IS NULL;
```

### Ejemplo RIGHT JOIN
![RIGHT JOIN](https://raw.githubusercontent.com/mayracmg/playground-3s1cjv5r/refs/heads/master/RightJoin.jpg)

**A**: credit_card_fraud_detection<br>
**B**: credit_card_fraud_detection
```sql
SELECT *
FROM credit_card_fraud_detection A
RIGHT JOIN credit_card_fraud_detection B ON A.transaction_date = B.transaction_Date 
    AND B.country = 'Canada'
	AND A.country = 'USA';
```

```sql
SELECT *
FROM credit_card_fraud_detection A
RIGHT JOIN credit_card_fraud_detection B ON A.transaction_date = B.transaction_Date 
    AND B.country = 'Canada'
	AND A.country = 'USA' 
WHERE A.transaction_id IS NULL;
```

### Ejemplo FULL JOIN
![CROSS JOIN](https://raw.githubusercontent.com/mayracmg/playground-3s1cjv5r/refs/heads/master/FullOuterJoin.png)

**A**: credit_card_fraud_detection<br>
**B**: credit_card_fraud_detection

```sql
SELECT *
FROM credit_card_fraud_detection A
LEFT JOIN credit_card_fraud_detection B ON A.transaction_date = B.transaction_Date 
    AND B.country = 'Canada'
	AND A.country = 'USA'

UNION

SELECT *
FROM credit_card_fraud_detection A
RIGHT JOIN credit_card_fraud_detection B ON A.transaction_date = B.transaction_Date 
    AND B.country = 'Canada'
	AND A.country = 'USA' 
```

### Ejemplo CROSS JOIN

**A**: country<br>
**B**: location
```sql
SELECT *
FROM country A
CROSS JOIN location B;

SELECT *
FROM country A, location B
```


```sql
SELECT *
FROM credit_card_fraud_detection A
CROSS JOIN credit_card_fraud_detection B 
WHERE A.country != B.country
```

### Ejemplo SELF JOIN

**A**: credit_card_fraud_detection<br>
**B**: credit_card_fraud_detection
```sql
SELECT A.transaction_id, A.merchant_name, A.country, A.card_type, A.transaction_date, B.transaction_date
FROM credit_card_fraud_detection A
LEFT JOIN credit_card_fraud_detection B ON A.merchant_name = B.merchant_name
	AND A.country = B.country
    AND A.card_type = B.card_type
    AND A.transaction_date != B.transaction_date;
```
---
## Joins con varias tablas
Cuando unir dos tablas no es necesario para generar el resultado esperado, puede ser necesario unir mas de 2 tablas.


```sql
SELECT T.*, U.birthdate
FROM transaction T
INNER JOIN USER U ON U.user_id = T.user_id;
```
:::

::: Common Table Expressions
Es un conjunto de resultados con nombre temporal al que puede hacer referencia dentro de una instrucción <span style="color:blue">SELECT</span>, <span style="color:blue">INSERT</span>, <span style="color:blue">UPDATE</span> o <span style="color:blue">DELETE</span>. El CTE también se la puede usar en una vista.

**Sintaxis**:

<span style="color:blue">WITH</span> + alias + <span style="color:blue">AS</span> + (QUERY CTE)
Query que hace referencia al CTE

```sql
WITH A AS (
	SELECT *
	FROM credit_card_fraud_detection
	WHERE country = 'USA' 
),
B AS (
	SELECT *
	FROM credit_card_fraud_detection
	WHERE country = 'Canada' 
)
SELECT *
FROM A
INNER JOIN B ON A.transaction_date = B.transaction_date;


WITH CTE AS (
	SELECT transaction_id, user_id
	FROM credit_card_fraud_detection
)
UPDATE transaction
INNER JOIN cte ON cte.transaction_id = transaction.transaction_id
SET transaction.user_id = cte.user_id;

WITH CTE AS (
	SELECT U.user_id, G.name
	FROM user U
	INNER JOIN gender G ON U.gender_id = G.gender_id
    WHERE G.name = 'Female'
)
UPDATE transaction
INNER JOIN CTE C 
  ON C.user_id = transaction.user_id
SET transaction.transaction_amount = transaction.transaction_amount * 0.9;
```
:::


