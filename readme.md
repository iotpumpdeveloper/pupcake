Pubcake, a micro framework for PHP 5.3+
=============================

Pubcake is a micro framework for PHP 5.3+

Usage:

1. Simple get request

<?php
require "pupcake.php";

$app = new \Pupcake\Pupcake();

$app->get("/hello/:name", function($name){
  return "hello ".$name;
});

$app->run();


