# 升级到 v3

在很大程度上保持了向后兼容性，但在从 v2 迁移到 v3 时有一些更改是您应该注意的。

## 输出缓冲行为 (3.5.0)

[输出缓冲](https://stackoverflow.com/questions/2832010/what-is-output-buffering-in-php) 是指 PHP 脚本生成的输出存储在一个缓冲区（PHP 内部）中，然后再发送给客户端。这允许您在发送给客户端之前修改输出。

在 MVC 应用程序中，控制器是“管理者”，它管理视图的操作。在控制器之外生成输出（或在 Flight 的情况下有时是匿名函数）会破坏 MVC 模式。这种更改是为了更符合 MVC 模式，并使框架更加可预测和易于使用。

在 v2 中，输出缓冲处理方式是不一致关闭自己的输出缓冲区，这使得[单元测试](https://github.com/flightphp/core/pull/545/files#diff-eb93da0a3473574fba94c3c4160ce68e20028e30b267875ab0792ade0b0539a0R42) 
和[流处理](https://github.com/flightphp/core/issues/413)更加困难。对大多数用户来说，这种更改实际上可能不会影响您。但是，如果您在可调用函数和控制器之外输出内容（例如在钩子中），您可能会遇到问题。在钩子中输出内容，并在框架实际执行之前，以前可能有效，但在向前移动时将无效。

### 您可能会遇到问题的地方
```php
// index.php
require 'vendor/autoload.php';

// just an example
define('START_TIME', microtime(true));

function hello() {
	echo 'Hello World';
}

Flight::map('hello', 'hello');
Flight::after('hello', function(){
	// this will actually be fine
	echo '<p>This Hello World phrase was brought to you by the letter "H"</p>';
});

Flight::before('start', function(){
	// things like this will cause an error
	echo '<html><head><title>My Page</title></head><body>';
});

Flight::route('/', function(){
	// this is actually just fine
	echo 'Hello World';

	// This should be just fine as well
	Flight::hello();
});

Flight::after('start', function(){
	// this will cause an error
	echo '<div>Your page loaded in '.(microtime(true) - START_TIME).' seconds</div></body></html>';
});
```

### 打开 v2 渲染行为

您是否仍然可以保持旧代码的编写方式而不必重写以使其与 v3 兼容？是的，您可以！您可以通过将 `flight.v2.output_buffering` 配置选项设置为 `true` 来打开 v2 渲染行为。这将允许您继续使用旧的渲染行为，但建议您在向前移动时修复它。在框架的 v4 中，这将被移除。

```php
// index.php
require 'vendor/autoload.php';

Flight::set('flight.v2.output_buffering', true);

Flight::before('start', function(){
	// Now this will be just fine
	echo '<html><head><title>My Page</title></head><body>';
});

// more code 
```

## 调度器更改 (3.7.0)

如果您直接调用 `Dispatcher` 的静态方法，例如 `Dispatcher::invokeMethod()`、`Dispatcher::execute()` 等，您需要更新代码以避免直接调用这些方法。`Dispatcher` 已经转换为更符合面向对象的方式，因此可以更轻松地使用依赖注入容器。如果需要调用类似于 Dispatcher 的方法，您可以手动使用类似于 `$result = $class->$method(...$params);` 或 `call_user_func_array()` 的方式。

## `halt()` `stop()` `redirect()` 和 `error()` 更改 (3.10.0)

在 3.10.0 之前的默认行为是清除标头和响应主体。这已更改为仅清除响应主体。如果您需要清除标头，您可以使用 `Flight::response()->clear()`。