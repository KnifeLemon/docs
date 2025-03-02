# Latte

[Latte](https://latte.nette.org/en/guide) ist ein voll ausgestatteter Template-Engine, der sehr einfach zu bedienen ist und sich näher an einer PHP-Syntax anfühlt als Twig oder Smarty. Es ist auch sehr einfach zu erweitern und eigene Filter und Funktionen hinzuzufügen.

## Installation

Installieren Sie mit Composer.

```bash
composer require latte/latte
```

## Grundkonfiguration

Es gibt einige grundlegende Konfigurationsoptionen, um loszulegen. Weitere Informationen dazu finden Sie in der [Latte-Dokumentation](https://latte.nette.org/en/guide).

```php

use Latte\Engine as LatteEngine;

require 'vendor/autoload.php';

$app = Flight::app();

$app->register('latte', LatteEngine::class, [], function(LatteEngine $latte) use ($app) {

	// Hier speichert Latte Ihre Templates, um die Geschwindigkeit zu erhöhen
	// Eine interessante Sache an Latte ist, dass es automatisch Ihren Cache aktualisiert,
	// wenn Sie Änderungen an Ihren Templates vornehmen!
	$latte->setTempDirectory(__DIR__ . '/../cache/');

	// Geben Sie Latte an, wo sich das Stammverzeichnis für Ihre Ansichten befinden wird.
	// $app->get('flight.views.path') ist in der config.php-Datei festgelegt
	//   Sie könnten auch einfach etwas wie `__DIR__ . '/../views/'` tun
	$latte->setLoader(new \Latte\Loaders\FileLoader($app->get('flight.views.path')));
});
```

## Einfaches Layoutbeispiel

Hier ist ein einfaches Beispiel für eine Layoutdatei. Diese Datei wird verwendet, um alle Ihre anderen Ansichten zu umschließen.

```html
<!-- app/views/layout.latte -->
<!doctype html>
<html lang="de">
	<head>
		<title>{$title ? $title . ' - '}Meine App</title>
		<link rel="stylesheet" href="style.css">
	</head>
	<body>
		<header>
			<nav>
				<!-- Ihre Navigations-Elemente hier -->
			</nav>
		</header>
		<div id="content">
			<!-- Hier liegt die Magie -->
			{block content}{/block}
		</div>
		<div id="footer">
			&copy; Urheberrecht
		</div>
	</body>
</html>
```

Und jetzt haben wir Ihre Datei, die in diesem Inhaltsblock gerendert werden soll:

```html
<!-- app/views/home.latte -->
<!-- Dies sagt Latte, dass diese Datei "innerhalb" der layout.latte-Datei ist -->
{extends layout.latte}

<!-- Dies ist der Inhalt, der innerhalb des Layouts im Inhaltsblock gerendert wird -->
{block content}
	<h1>Startseite</h1>
	<p>Willkommen bei meiner App!</p>
{/block}
```

Dann, wenn Sie dies in Ihrer Funktion oder Ihrem Controller rendern möchten, würden Sie etwas Ähnliches tun:

```php
// einfache Route
Flight::route('/', function () {
	Flight::latte()->render('home.latte', [
		'title' => 'Startseite'
	]);
});

// oder wenn Sie einen Controller verwenden
Flight::route('/', [HomeController::class, 'index']);

// HomeController.php
class HomeController
{
	public function index()
	{
		Flight::latte()->render('home.latte', [
			'title' => 'Startseite'
		]);
	}
}
```

Schauen Sie in der [Latte-Dokumentation](https://latte.nette.org/en/guide) nach weiteren Informationen darüber, wie Sie Latte optimal nutzen können!