# Classe d'Aide PDO PdoWrapper

## Aperçu

La classe `PdoWrapper` dans Flight est un assistant convivial pour travailler avec les bases de données en utilisant PDO. Elle simplifie les tâches courantes de base de données, ajoute des méthodes pratiques pour récupérer les résultats, et retourne les résultats sous forme de [Collections](/learn/collections) pour un accès facile. Elle supporte également la journalisation des requêtes et la surveillance des performances de l'application (APM) pour des cas d'utilisation avancés.

## Comprendre

Travailler avec les bases de données en PHP peut être un peu verbeux, surtout en utilisant PDO directement. `PdoWrapper` étend PDO et ajoute des méthodes qui rendent les requêtes, la récupération et la gestion des résultats beaucoup plus faciles. Au lieu de jongler avec des instructions préparées et des modes de récupération, vous obtenez des méthodes simples pour les tâches courantes, et chaque ligne est retournée sous forme de Collection, afin que vous puissiez utiliser la notation tableau ou objet.

Vous pouvez enregistrer `PdoWrapper` en tant que service partagé dans Flight, puis l'utiliser n'importe où dans votre application via `Flight::db()`.

## Utilisation de Base

### Enregistrement de l'Aideur PDO

D'abord, enregistrez la classe `PdoWrapper` avec Flight :

```php
Flight::register('db', \flight\database\PdoWrapper::class, [
    'mysql:host=localhost;dbname=cool_db_name', 'user', 'pass', [
        PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES \'utf8mb4\'',
        PDO::ATTR_EMULATE_PREPARES => false,
        PDO::ATTR_STRINGIFY_FETCHES => false,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
    ]
]);
```

Maintenant, vous pouvez utiliser `Flight::db()` n'importe où pour obtenir votre connexion à la base de données.

### Exécution de Requêtes

#### `runQuery()`

`function runQuery(string $sql, array $params = []): PDOStatement`

Utilisez ceci pour les INSERT, UPDATE, ou lorsque vous voulez récupérer les résultats manuellement :

```php
$db = Flight::db();
$statement = $db->runQuery("SELECT * FROM users WHERE status = ?", ['active']);
while ($row = $statement->fetch()) {
    // $row est un tableau
}
```

Vous pouvez également l'utiliser pour les écritures :

```php
$db->runQuery("INSERT INTO users (name) VALUES (?)", ['Alice']);
$db->runQuery("UPDATE users SET name = ? WHERE id = ?", ['Bob', 1]);
```

#### `fetchField()`

`function fetchField(string $sql, array $params = []): mixed`

Obtenez une seule valeur de la base de données :

```php
$count = Flight::db()->fetchField("SELECT COUNT(*) FROM users WHERE status = ?", ['active']);
```

#### `fetchRow()`

`function fetchRow(string $sql, array $params = []): Collection`

Obtenez une seule ligne sous forme de Collection (accès tableau/objet) :

```php
$user = Flight::db()->fetchRow("SELECT * FROM users WHERE id = ?", [123]);
echo $user['name'];
// ou
echo $user->name;
```

#### `fetchAll()`

`function fetchAll(string $sql, array $params = []): array<Collection>`

Obtenez toutes les lignes sous forme de tableau de Collections :

```php
$users = Flight::db()->fetchAll("SELECT * FROM users WHERE status = ?", ['active']);
foreach ($users as $user) {
    echo $user['name'];
    // ou
    echo $user->name;
}
```

### Utilisation des Lieux-Tenants `IN()`

Vous pouvez utiliser un seul `?` dans une clause `IN()` et passer un tableau ou une chaîne séparée par des virgules :

```php
$ids = [1, 2, 3];
$users = Flight::db()->fetchAll("SELECT * FROM users WHERE id IN (?)", [$ids]);
// ou
$users = Flight::db()->fetchAll("SELECT * FROM users WHERE id IN (?)", ['1,2,3']);
```

## Utilisation Avancée

### Journalisation des Requêtes & APM

Si vous voulez suivre les performances des requêtes, activez le suivi APM lors de l'enregistrement :

```php
Flight::register('db', \flight\database\PdoWrapper::class, [
    'mysql:host=localhost;dbname=cool_db_name', 'user', 'pass', [/* options */], true // dernier paramètre active APM
]);
```

Après l'exécution des requêtes, vous pouvez les journaliser manuellement, mais l'APM les journalisera automatiquement si activé :

```php
Flight::db()->logQueries();
```

Cela déclenchera un événement (`flight.db.queries`) avec les métriques de connexion et de requête, que vous pouvez écouter en utilisant le système d'événements de Flight.

### Exemple Complet

```php
Flight::route('/users', function () {
    // Obtenir tous les utilisateurs
    $users = Flight::db()->fetchAll('SELECT * FROM users');

    // Diffuser tous les utilisateurs
    $statement = Flight::db()->runQuery('SELECT * FROM users');
    while ($user = $statement->fetch()) {
        echo $user['name'];
    }

    // Obtenir un seul utilisateur
    $user = Flight::db()->fetchRow('SELECT * FROM users WHERE id = ?', [123]);

    // Obtenir une seule valeur
    $count = Flight::db()->fetchField('SELECT COUNT(*) FROM users');

    // Syntaxe spéciale IN()
    $users = Flight::db()->fetchAll('SELECT * FROM users WHERE id IN (?)', [[1,2,3,4,5]]);
    $users = Flight::db()->fetchAll('SELECT * FROM users WHERE id IN (?)', ['1,2,3,4,5']);

    // Insérer un nouvel utilisateur
    Flight::db()->runQuery("INSERT INTO users (name, email) VALUES (?, ?)", ['Bob', 'bob@example.com']);
    $insert_id = Flight::db()->lastInsertId();

    // Mettre à jour un utilisateur
    Flight::db()->runQuery("UPDATE users SET name = ? WHERE id = ?", ['Bob', 123]);

    // Supprimer un utilisateur
    Flight::db()->runQuery("DELETE FROM users WHERE id = ?", [123]);

    // Obtenir le nombre de lignes affectées
    $statement = Flight::db()->runQuery("UPDATE users SET name = ? WHERE name = ?", ['Bob', 'Sally']);
    $affected_rows = $statement->rowCount();
});
```

## Voir Aussi

- [Collections](/learn/collections) - Apprenez comment utiliser la classe Collection pour un accès facile aux données.

## Dépannage

- Si vous obtenez une erreur concernant la connexion à la base de données, vérifiez votre DSN, nom d'utilisateur, mot de passe et options.
- Toutes les lignes sont retournées sous forme de Collections—si vous avez besoin d'un tableau simple, utilisez `$collection->getData()`.
- Pour les requêtes `IN (?)`, assurez-vous de passer un tableau ou une chaîne séparée par des virgules.

## Journal des Modifications

- v3.2.0 - Publication initiale de PdoWrapper avec les méthodes de requête et de récupération de base.