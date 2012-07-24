Pupcake --- a micro framework for PHP 5.3+
=======================================

##About Pupcake Framework
Pupcake is a minimal but extensible microframework for PHP 5.3+. It has a powerful plugin and event handling system, which makes it "simple at the beginning, powerful at the end".
Pupcake mimic some of the api syntax in Express Node.js Framework, so developers in Express will feel at home. 

New Feature: Pupcake now supports asynchronous internal request.

For more detail usages, please see https://github.com/superjimpupcake/Pupcake/wiki/_pages

##Installation:

####install package "Pupcake/Pupcake" using composer (http://getcomposer.org/)

###.htaccess File for Apache
    RewriteEngine On
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^(.*)$ index.php/$1 [L]

###Simple requests
```php
<?php
//Assuming this is public/index.php and the composer vendor directory is ../vendor

require_once __DIR__.'/../vendor/autoload.php';

$app = new Pupcake\Pupcake();

$app->get("date/:year/:month/:day", function($req, $res){
    $output = $req->params('year').'-'.$req->params('month').'-'.$req->params('day');
    $res->send($output);
});

$app->get("/hello/:name", function($req, $res){
  $res->send("hello ".$req->params('name')." in get");
});

$app->post("/hello/:name", function($req, $res){
  $res->send("hello ".$req->params('name')." in post");
});

$app->put("/hello/:name", function($req, $res){
  $res->send("hello ".$req->params('name')." in put");
});

$app->delete("/hello/:name", function($req, $res){
  $res->send("hello ".$req->params('name')." in delete");
});

/**
 * Multiple request methods for one route
 */
$app->map("/api/hello/:action", function($req, $res){
  $res->send("hello ".$req->params('action')." in get and post");
})->via('GET','POST');


$app->run();
```

###Handling request with [action_class]#[action_method] string
Pupcake also support handling route with [action_class]#[action_method] string. 
In the example below, we use Page class's home action to handle the get request to the homepage.
```php
<?php
//Assuming this is public/index.php and the composer vendor directory is ../vendor

require_once __DIR__.'/../vendor/autoload.php';

class Page
{
  public function home($req, $res)
  {
    $res->send("this is the homepage"); 
  }
}

$app = new Pupcake\Pupcake();

$app->get("/", "Page#home");

$app->run();

```

###Using constraint in route and using $next function to find the next route like Express framework
```php
<?php
//Assuming this is public/index.php and the composer vendor directory is ../vendor

require_once __DIR__.'/../vendor/autoload.php';

$app = new Pupcake\Pupcake();

$app->usePlugin("Pupcake\Plugin\RouteConstraint"); 

$app->any("api/12", function($req, $res, $next){
    $next();
});

$app->any("api/:number", function($req, $res, $next){
    $next();
})->constraint(array(
    'number' => function($value){
        $result = true;
        if($value < 15){
            $result = false;
        }
        return $result;
    }
));

$app->get("api/12", function($req, $res, $next){
    $next();
});

$app->get("api/:number", function($req, $res, $next){
    $res->send("this is finally number ".$req->params('number'));
});


$app->run();
```

###Event Helper and Event Handler
In Pupcake framework, each event can have only one event handler, it is swappable. Each event can register one or many event helpers to join the process of the event handling.
```php
<?php
//Assuming this is public/index.php and the composer vendor directory is ../vendor

require_once __DIR__.'/../vendor/autoload.php';

$app = new Pupcake\Pupcake();

/**
 * we override the system.request.found event's handler
 */
$app->on("system.request.found", function($event){
    /**
     * We register 3 event helper callbacks for the sytem.request.found event
     */
    $results = $event->register(
        function(){
            return "output 1";
        },
        function(){
            return "output 2";
        },
        function(){
            return "output 3"; 
        }
    )->start();

    $output = "";
    if(count($results) > 0){
        foreach($results as $result){
            $output .= $result;
        } 
    }

    return $output;
});

$app->any("*path", function($req, $res){
});

$app->run();
```

### Custom Event Triggering and Handling
```php
<?php
//Assuming this is public/index.php and the composer vendor directory is ../vendor

require_once __DIR__.'/../vendor/autoload.php';

$app = new Pupcake\Pupcake();

/**
 * We define a custom event handler for the node.view event
 */
$app->on("node.view", function($event){
    return "viewing node id ".$event->props('id');
});

$app->any("node/:id", function($req, $res) use ($app) {
    /**
     * We trigger the node.view event when we have node/1, node/2... in the request path
     * The second parameter in the trigger method is empty string since we don't have 
     * a default event handler callback for the node.view event
     */
    $output = $app->trigger("node.view", "", array('id' => $req->params('id')));
    $res->send($output);
});

$app->run();
```

### URL Alias effect using request forwarding, custom event trigging and handling
```php
<?php
//Assuming this is public/index.php and the composer vendor directory is ../vendor

require_once __DIR__.'/../vendor/autoload.php';

$app = new Pupcake\Pupcake();

$app->on("node.view", function($event){
    return "viewing node id ".$event->props('id');
});

$app->get("article/*path", function($req, $res){
    $map = array(
        "stock-exchange" => 1,
        "world/market/preview" => 2,
        "wine/list" => 3
    );
    $path = $req->params("path");
    if(isset($map[$path])){
        $res->send($res->forward("get", "node/".$map[$path])); //we forward the request to node
    }
});

$app->get("node/:id", function($req, $res) use ($app) {
    /**
     * We trigger the node.view event when we have node/1, node/2... in the request path
     * The second parameter in the trigger method is empty string since we don't have 
     * a default event handler callback for the node.view event
     */
    $output = $app->trigger("node.view", "", array('id' => $req->params('id')));
    $res->send($output);
});

$app->run();
```

### Sending asynchronous internal request
```php
<?php
//Assuming this is public/index.php and the composer vendor directory is ../vendor
require_once __DIR__.'/../vendor/autoload.php';

$app = new Pupcake\Pupcake();

/**
 * Pupcake now support asynchronous internal request, the request is non-blocking
 */
$app->get("file_create_call", function($req, $res) use ($app) {
  session_start();
  $_SESSION['session_jim'] = 1;
  $_SESSION['session_tom'] = 2;
  $_SESSION['session_huang'] = 5;
  $_SESSION['more_data'] = 300;
  $result = $app->sendAsyncRequest("post", "file_create");   
});

$app->post("file_create", function($req, $res){
  session_start();
  file_put_contents("tmp", print_r($_SESSION,true));
});

$app->run();
```
