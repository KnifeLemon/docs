# Instalación

## Descargar los archivos.

Si estás utilizando [Composer](https://getcomposer.org), puedes ejecutar el siguiente comando:

```bash
composer require flightphp/core
```

O puedes [descargar los archivos](https://github.com/flightphp/core/archive/master.zip)
 directamente y extraerlos en tu directorio web.

## Configurar tu Servidor Web.

### Apache
Para Apache, edita tu archivo `.htaccess` con lo siguiente:

```apacheconf
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php [QSA,L]
```

> **Nota**: Si necesitas usar flight en un subdirectorio, añade la línea
> `RewriteBase /subdir/` justo después de `RewriteEngine On`.

> **Nota**: Si deseas proteger todos los archivos del servidor, como un archivo de base de datos o env.
> Agrega esto en tu archivo `.htaccess`:

```apacheconf
RewriteEngine On
RewriteRule ^(.*)$ index.php
```

### Nginx

Para Nginx, agrega lo siguiente a la declaración de tu servidor:

```nginx
server {
  location / {
    try_files $uri $uri/ /index.php;
  }
}
```

## Crea tu archivo `index.php`. 

```php
<?php

// Si estás utilizando Composer, requiere el cargador automático.
require 'vendor/autoload.php';
// si no estás utilizando Composer, carga el framework directamente
// require 'flight/Flight.php';

// Luego define una ruta y asigna una función para manejar la solicitud.
Flight::route('/', function () {
  echo '¡Hola Mundo!';
});

// Finalmente, inicia el framework.
Flight::start();
```