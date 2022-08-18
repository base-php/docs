# Base de datos: Como empezar

- [Introducción](#introduccion)
    - [Configuración](#configuracion)
- [Ejecutando SQL](#ejecutando-sql)
    - [Usando multiples conexiones a base de datos](#usando-multiples-conexiones)
- [Database Transactions](#database-transactions)
- [Connecting To The Database CLI](#connecting-to-the-database-cli)

<a name="introduccion"></a>
## Introducción

Casi todas las aplicaciones web modernas interactúan con una base de datos. Base PHP hace que la interacción con las bases de datos sea extremadamente simple en una variedad de bases de datos compatibles utilizando SQL sin procesar, un [generador de consultas fuildo](/docs/queries) y el [ORM Eloquent](/docs/eloquent). Actualmente, Base PHP proporciona soporte propio para cinco bases de datos:

<div class="content-list" markdown="1">

- MariaDB 10.3+ ([Version Policy](https://mariadb.org/about/#maintenance-policy))
- MySQL 5.7+ ([Version Policy](https://en.wikipedia.org/wiki/MySQL#Release_history))
- PostgreSQL 10.0+ ([Version Policy](https://www.postgresql.org/support/versioning/))
- SQLite 3.8.8+
- SQL Server 2017+ ([Version Policy](https://docs.microsoft.com/en-us/lifecycle/products/?products=sql-server))

</div>

<a name="configuracion"></a>
### Configuración

La configuración de los servicios de base de datos de BasePHP se encuentra en el archivo de configuración de su aplicación `app/config.php`. En este archivo, puede definir todas las conexiones de su base de datos, así como especificar que conexión debe usarse de manera predeterminada.

<a name="configuracion-sqlite"></a>
#### Configuración SQLite

Las bases de datos SQLite están contenidas en un solo archivo en su sistema de archivos. Puede crear una nueva base de datos SQLite usando el comando `touch` en su terminal: `touch database/database.sqlite`. Una vez creada la base de datos, puede configurar fácilmente sus variables de entorno para que apunten a esta base de datos colocando la ruta absoluta a la base de datos en su variable `database`:


```php
'database'          => [
    [
        'driver'    => 'sqlite',
        'database'  => 'base',
    ]
]
```

<a name="configuracion-mssql"></a>
#### Microsoft SQL Server Configuration

Para usar una base de datos de Microsoft SQL Server, debe asegurarse de tener instaladas las extensiones de PHP `sqlsrv` y `pdo_sqlsrv`, así como cualquier dependencia que puedan requerir, como el controlador ODBC de Microsoft SQL.

<a name="configuracion-usando-urls"></a>
#### Configuración usando URLs

Por lo general, las conexiones de la base de datos se configuran utilizando valores de configuración, como `host`, `database`, `username`, `password`, etc. Cada uno de estos valores de configuración tiene su propia variable de entorno correspondiente. Esto significa que al configurar la información de conexión de su base de datos en un servidor de producción, debe administrar varias variables de entorno.

Algunos proveedores de bases de datos administradas, como AWS y Heroku, proporciona solo una "URL" de base de datos que contiene toda la información de conexión en una sola cadena. Una URL de ejemplo puede parecerse a lo siguiente

```html
mysql://root:password@127.0.0.1/forge?charset=UTF-8
```

Estas URL suelen seguir una convención de esquema estándar:

```html
driver://username:password@host:port/database?options
```

Para mayor comodidad, Base PHP admite estas URL como alternativa a la configuración de su base de datos con múltiples opciones de configuración. Si la opción de configuración `url` (o la variable de entorno `DATABASE_URL` correspondiente) está presente, se usará para extraer la conexión de la base de datos y la información de crendenciales.

<a name="ejecutando-sql"></a>
## Ejecutando SQL

Una vez que haya configuración su conexión a la base de datos, puede ejcutar consultas utilizando la clase `DB`. La clase `DB` proporciona métodos para cada tipo de consulta: `select`, `update`, `insert`, `delete` y `statement`.

<a name="ejecutando-select"></a>
#### Ejecutando un select

Para ejecutar una consulta SELECT básica, puede usar el método `select` en la fachada `DB`:

    <?php

    namespace App\Controllers;

    use DB;
    use View;

    class UserController extends Controller
    {
        /**
         * Show a list of all of the application's users.
         *
         * @return View
         */
        public function index()
        {
            $users = DB::select('select * from users where active = ?', [1]);

            return view('user.index', ['users' => $users]);
        }
    }

El primer argumento que se pasa al método `select` es la consulta SQL, mientras que el segundo argumento son los enlaces de parámetros que deben vincularse a la consulta. Por lo general, estos son los valores de las restricciones de la cláusula `where`. El enlace de parámetros proporciona protección contra la inyección de SQL.

El método `select` siempre devolverá una `matriz` de resultados. Cada resultado dentro de la matriz será un objeto PHP `stdClass` que representa un registro de la base de datos:

    use DB;

    $users = DB::select('select * from users');

    foreach ($users as $user) {
        echo $user->name;
    }

<a name="usando-enlaces-nombre"></a>
#### Uso de enlaces con nombre

En lugar de usar `?` para representar sus enlaces de parámetros, puede ejecutar una consulta usando enlaces con nombre:

    $results = DB::select('select * from users where id = :id', ['id' => 1]);

<a name="ejecutando-sentencia-insert"></a>
#### Ejecutando una sentencia insert

Para ejecutar un `insert`, puede usar el método `insert` de la clase `DB`. Al igual que `select`, este método acepta la consulta SQL como primer argumento y los enlaces como segundo argumento:

    use DB;

    DB::insert('insert into users (id, name) values (?, ?)', [1, 'Marc']);

<a name="ejecutando-sentencia-update"></a>
#### Ejecutando una sentencia update

El método `update` debe utilizarse para actualizar los registros existentes de la base de datos. El método devuelve el numero de filas afectadas por la sentencia:

    use DB;

    $affected = DB::update(
        'update users set votes = 100 where name = ?',
        ['Anita']
    );

<a name="ejecutando-sentencia-delete"></a>
#### Ejecutando una sentencia delete

El método `delete` debe usarse para eliminar registros de la base datos. Al igual que `update`, el método devolverá el número de filas afectadas:

    use DB;

    $deleted = DB::delete('delete from users');

<a name="ejecutando-sentencia-general"></a>
#### Ejecutando una sentencia general

Algunas sentencias no devuelven ningún valor. Para este tipo de operaciones, puede usar el método `statement` de la clase `DB`:

    DB::statement('drop table users');

<a name="ejecutando-sentencia-no-preparada"></a>
#### Ejecutando sentencia no preparada

A veces, es posible que desee ejecutar una instrucción SQL sin vincular ningún valor. Puede usar el método `unprepared` de la calse `DB`:

    DB::unprepared('update users set votes = 100 where name = "Dries"');

> **Advertencia**  
> Dado que las sentencias no preparadas no vinculan parámetros, pueden ser vulnerables a la inyección SQL. Nunca debe permitir valores controladors por el usuario dentro de una sentencia no preparada.

<a name="uso-varias-conexiones"></a>
### Uso de varias conexiones

Si su aplicación define múltiples conexiones en su archivo `app/config.php`, puede acceder a cada conexión a través del método `connection` de la clase `DB`. El nombre de la conexión pasado al método `connection` debe corresponder a uno de las conexiones definidas en su archivo `app/config.php`.

    use DB;

    $users = DB::connection('sqlite')->select(/* ... */);

Puede acceder a la instancia PDO subyacente sin procesar de una conexión utilizando el método `getPdo`:

    $pdo = DB::connection()->getPdo();

<a name="transacciones-base-datos"></a>
## Transacciones de base de datos

Puede utilizar el método `transation` proporcionado por la clase `DB` para ejecutar un conjunto de operaciones dentro de una transacción de base de datos. Si se lanza una excepción antes del cierre de la transacción, la transacción se revertirá automáticamente y se lanzará la excepción. Si la transacción se ejecuta con éxito, se confirma automáticamente. No necesita preocuparse por revertir o confirmar manualmente mientras usa el método `transaction`:

    use DB;

    DB::transaction(function () {
        DB::update('update users set votes = 1');

        DB::delete('delete from posts');
    });

<a name="manejo-errores"></a>
#### Manejo de errores

El método `transaction` acepta un segundo argumento opcional que define el número de veces que se debe volver a intentar una transacción cuando se produce un error. Una vez que se hayan agotado estos intentos, se lanzará una excepción:

    use DB;

    DB::transaction(function () {
        DB::update('update users set votes = 1');

        DB::delete('delete from posts');
    }, 5);

<a name="uso-manual-transacciones"></a>
#### Uso manual de transacciones

Si desea comenzar una transacción manualmente y tener un control completo sobre las reversiones y confirmaciones, puede usar el método `beginTransaction` de la clase `DB`:

    use DB;

    DB::beginTransaction();

Puede revertir la transacción con el método `rollBack`:

    DB::rollBack();

Por último, puede confirmar una transacción con el método `commit`:

    DB::commit();

> **Nota**  
> Los métodos de la clase `DB` controlan las transacciones tanto para el [construtor de consultas](/docs/queries) como para el [ORM Eloquent](/docs/eloquent).

<a name="conectando-cli-base-datos"></a>
## Conectando a la CLI de la base de datos

Si desea conectarse a la CLI de su base de datos, puede usar el comando `db`:

```shell
php base db
```

Si es necesario, puede especificar un nombre de conexión para conectarse a la base de datos que no sea la predeterminada:

```shell
php base db mysql
```
