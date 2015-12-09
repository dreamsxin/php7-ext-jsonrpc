php7-ext-jsonrpc
================

[![Build Status](https://travis-ci.org/rryqszq4/php7-ext-jsonrpc.svg?branch=master)](https://travis-ci.org/rryqszq4/php7-ext-jsonrpc)

Json-RPC 2.0 client/server for php-7 extension.

[php5.3~php5.6](https://github.com/rryqszq4/JsonRPC)

Features
--------
* JSON-RPC 2.0 protocol
* Base on curl and epoll of the multi client
* Persistent epoll in php-fpm
* Persistent curl_multi queue in php-fpm
* Support message and notifi notification
* Linux only(need to epoll)

Requirement
-----------
- PHP 7.0

Install
-------
```
$/path/to/phpize
$./configure --with-php-config=/path/to/php-config
$make && make install
```

Server
-----------
**Interface**
- Jsonrpc_Server::__construct
- Jsonrpc_Server::register
- Jsonrpc_Server::bind
- Jsonrpc_Server::jsonformat
- Jsonrpc_Server::rpcformat
- Jsonrpc_Server::executeprocedure
- Jsonrpc_Server::executecallback
- Jsonrpc_Server::executemethod
- Jsonrpc_Server::getresponse
- Jsonrpc_Server::execute

**Register Function**
```php
<?php

$server = new Jsonrpc_Server();

// style one function variable
$add1 = function($a, $b){
  return $a + $b;
};
$server->register('addition1', $add1);

// style two function string
function add2($a, $b){
  return $a + $b;
}
$server->register('addition2', 'add2');

// style three function closure
$server->register('addition3', function ($a, $b) {
    return $a + $b;
});

//style four class method string
class Api 
{
  static public function add($a, $b)
  {
    return $a + $b;
  }
}
$server->register('addition4', 'Api::add');

echo $server->execute();

//output >>>
//{"jsonrpc":"2.0","id":null,"error":{"code":-32700,"message":"Parse error"}}

?>
```

**Bind Method**
```php
<?php

$server = new Jsonrpc_Server();

class Api
{
  static public function add($a, $b)
  {
    return $a + $b;
  }

  public function newadd($a,$b){
    return $a + $b;
  }
}

$server->bind('addition5', 'Api', 'add');

$server->bind('addition6', $a=new Api, 'newadd');

echo $server->execute();

//output >>>
//{"jsonrpc":"2.0","id":null,"error":{"code":-32700,"message":"Parse error"}}

?>

```

Client
------------
**Interface**
- Jsonrpc_Client::__construct
- Jsonrpc_Client::call
- Jsonrpc_Client::execute

**Persistent**
> Jsonrpc_client(1) 
> When two resource epoll and curl_multi queue persist, the parame is 1. The default use of non-persistent.

**Multi Call**
```php
<?php

$client = new Jsonrpc_Client(1);
$client->call('http://localhost/server.php', 'addition1', array(3,5));
$client->call('http://localhost/server.php', 'addition2', array(10,20));

/* ... */
$result = $client->execute();

var_dump($result);

//output >>>
/*
array(2) {
  [0]=>
  array(3) {
    ["jsonrpc"]=>
    string(3) "2.0"
    ["id"]=>
    int(110507766)
    ["result"]=>
    int(8)
  }
  [1]=>
  array(3) {
    ["jsonrpc"]=>
    string(3) "2.0"
    ["id"]=>
    int(1559316299)
    ["result"]=>
    int(30)
  }
  ...
}
*/
?>
```
**Custom ID**
```php
<?php

$client = new Jsonrpc_client(1);
$client->call('http://localhost/server.php', 'addition', array(3,5),"custom_id_001");
$result = $client->execute();
var_dump($result);

//output >>>
/*
array(1) {
  [0]=>
  array(3) {
    ["jsonrpc"]=>
    string(3) "2.0"
    ["id"]=>
    string(13) "custom_id_001"
    ["result"]=>
    int(8)
  }
}
*/
?>
```

Error Info
--------------
**jsonrpc 2.0 Error**
```javascript
// Parse error
{"jsonrpc":"2.0","id":null,"error":{"code":-32700,"message":"Parse error"}}

// Invalid Request
{"jsonrpc":"2.0","id":null,"error":{"code":-32600,"message":"Invalid Request"}}

// Method not found
{"jsonrpc":"2.0","id":null,"error":{"code":-32601,"message":"Method not found"}}

// Invalid params
{"jsonrpc":"2.0","id":null,"error":{"code":-32602,"message":"Invalid params"}}

//
```

**HTTP Error**
```javascript
// 400
{"jsonrpc":"2.0","id":null,"error":{"code":-32400,"message":"Bad Request"}}
// 401
{"jsonrpc":"2.0","id":null,"error":{"code":-32401,"message":"Unauthorized"}}
// 403
{"jsonrpc":"2.0","id":null,"error":{"code":-32403,"message":"Forbidden"}}
// 404
{"jsonrpc":"2.0","id":null,"error":{"code":-32404,"message":"Not Found"}}

// 500
{"jsonrpc":"2.0","id":null,"error":{"code":-32500,"message":"Internal Server Error"}}
// 502
{"jsonrpc":"2.0","id":null,"error":{"code":-32502,"message":"Bad Gateway"}}
...

// unknow
{"jsonrpc":"2.0","id":null,"error":{"code":-32599,"message":"HTTP Unknow"}}
```

**Curl Error**
```javascript
// 1 CURLE_UNSUPPORTED_PROTOCOL
{"jsonrpc":"2.0","id":null,"error":{"code":-32001,"message":"Curl Unsupported Protocol"}}

// 2 CURLE_FAILED_INIT
{"jsonrpc":"2.0","id":null,"error":{"code":-32002,"message":"Curl Failed Init"}}

// 3 CURLE_URL_MALFORMAT
{"jsonrpc":"2.0","id":null,"error":{"code":-32003,"message":"Curl Url Malformat"}}

// 4
{"jsonrpc":"2.0","id":null,"error":{"code":-32004,"message":"Curl Not Built In"}}

// 5 CURLE_COULDNT_RESOLVE_PROXY
{"jsonrpc":"2.0","id":null,"error":{"code":-32005,"message":"Curl Couldnt Resolve Proxy"}}

// 6 CURLE_COULDNT_RESOLVE_HOST
{"jsonrpc":"2.0","id":null,"error":{"code":-32006,"message":"Curl Couldnt Resolve Host"}}

// 7 CURLE_COULDNT_CONNECT
{"jsonrpc":"2.0","id":null,"error":{"code":-32007,"message":"Curl Couldnt Connect"}}
...

// CURL ERROR UNKNOW
{"jsonrpc":"2.0","id":null,"error":{"code":-32099,"message":"Curl Error Unknow"}}
```

