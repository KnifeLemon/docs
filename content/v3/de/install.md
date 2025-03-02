# Installation

## Lade die Dateien herunter.

Wenn Sie [Composer](https://getcomposer.org) verwenden, können Sie den folgenden Befehl ausführen:

```bash
composer require flightphp/core
```

ODER Sie können die [Dateien herunterladen](https://github.com/flightphp/core/archive/master.zip)
 direkt und sie in Ihr Webverzeichnis extrahieren.

## Konfigurieren Sie Ihren Webserver.

### Apache
Für Apache bearbeiten Sie Ihre `.htaccess`-Datei wie folgt:

```apacheconf
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php [QSA,L]
```

> **Hinweis**: Wenn Sie Flight in einem Unterverzeichnis verwenden müssen, fügen Sie die Zeile hinzu
> `RewriteBase /subdir/` direkt nach `RewriteEngine On` hinzu.

> **Hinweis**: Wenn Sie alle Serverdateien schützen möchten, wie z.B. eine db- oder env-Datei.
> Fügen Sie dies Ihrer `.htaccess`-Datei hinzu:

```apacheconf
RewriteEngine On
RewriteRule ^(.*)$ index.php
```

### Nginx

Für Nginx fügen Sie Folgendes zu Ihrer Serverdeklaration hinzu:

```nginx
server {
  location / {
    try_files $uri $uri/ /index.php;
  }
}
```

## Erstellen Sie Ihre `index.php`-Datei.

```php
<?php

// Wenn Sie Composer verwenden, fordern Sie den Autoloader an.
require 'vendor/autoload.php';
// wenn Sie Composer nicht verwenden, laden Sie das Framework direkt
// require 'flight/Flight.php';

// Definieren Sie dann eine Route und weisen Sie eine Funktion zu, um die Anfrage zu bearbeiten.
Flight::route('/', function () {
  echo 'Hallo Welt!';
});

// Starten Sie schließlich das Framework.
Flight::start();
```