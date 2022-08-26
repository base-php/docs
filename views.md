# Views

- [Introducción](#introduction)
- [Crear y renderizar vistas](#creating-and-rendering-views)
    - [Directorios de vistas anidadas](#nested-view-directories)
- [Pasar datos a vistas](#passing-data-to-views)

<a name="introduction"></a>
## Introducción

Por supuesto, no es práctico devolver cadenas completas de documentos HTML directamente desde sus rutas y controladores. Afortunadamente, las vistas brindan una forma conveniente de colocar todo nuestro HTML en archivos separados. Las vistas separan la lógica de su controaldor de su lógica de presentación y se almacenan en el directorio `resources/views`. Una vista simple podría verse así:

```blade
<!-- View stored in resources/views/greeting.blade.php -->

<html>
    <body>
        <h1>Hello, {{ $name }}</h1>
    </body>
</html>
```

Dado que esta vista se almacena en `resources/views/greeting.blade.php`, podemos devolverla usando el helper global `view` de la siguiente manera:

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

> **Nota**  
> ¿Busca más información sobre cómo escribir plantillas Blade? Consulta la [documentación de Blade](/docs/blade) para comenzar.

<a name="creating-and-rendering-views"></a>
## Crear y renderizar vistas

Puede crear una vista colocando un archivo con la extensión `.blade.php` en el directorio `resources/views` de su aplicación. La extensión `.blade.php` le informa al framework que ese archivo contiene una [plantilla Blade](/docs/{{version}}/blade). Las plantillas Blade contienen HTML, así como directivas Blade que le permiten reproducir fácilmente valores, crear sentencias "si", iterar datos y más.

Una vez que haya creado una vista, puede devolverla desde una de las rutas o controladores de su aplicación usando el helper global `view`:

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

Como puede ver, el primer argumento pasado al helper `view` corresponde al nombre del archivo de vista en el directorio `resources/views`. El segundo argumento es una arreglo de datos que debe estar disponible para la vista. En este caso, estamos pasando la variable `name`, que se muestra en la vista usando [sintaxis de Blade](/docs/{{version}}/blade).

<a name="nested-view-directories"></a>
### Directorios de vistas anidadas

Las vistas también se pueden anidar dentro de los subdirectorios del directorio `resources/views`. La notación de "puntos" se puede utilizar para hacer referencia a vistas anidadas. Por ejemplo, si su vista está almacenada en `resources/views/admin/profile.blade.php`, puede devolverla desde una de las rutas/controladores de su aplicación así:

    return view('admin.profile', $data);

> **Advertencia**  
> Los nombres de los directorios no deben contener el carácter `.`.

<a name="passing-data-to-views"></a>
## Passing Data To Views

Como vio en los ejemplos anteriores, puede pasar una matriz de datos a las vistas para que esos datos estén disponibles para la vista:

    return view('greetings', ['name' => 'Victoria']);

Al pasar información de esta manera, los datos deben ser una matriz con pares clave/valor. Después de proporcionar datos a una vista, puede acceder a cada valor dentro de su vista utilizando las claves de los datos, como `<?php echo $name; ?>`.
