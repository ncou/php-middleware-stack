# PHP Middleware Stack
[![Build Status](https://travis-ci.com/idealo/php-middleware-stack.svg?token=dB3owzyXmEKz9x3RX1AW&branch=master)](https://travis-ci.com/idealo/php-middleware-stack)

This is an implementation of [PSR-15 Draft](https://github.com/php-fig/fig-standards/blob/master/proposed/http-middleware/middleware.md) using the proposed Interface package [http-interop/http-middleware](https://github.com/http-interop/http-middleware) for PHP7+ runtime environment.

It enables a sequential execution of middlewares that use a PSR-7 conform Response/Request implementation.

## Install

```bash 

$ composer require idealo/php-middleware-stack

```

## How to
```php

<?php

use Idealo\Middleware\Stack;

$stack = new Stack(
    $defaultResponse,
    $middleware1,
    $middleware2,
    $middleware3
);

$stackResponse = $stack->process($request);


```

## Usage
**idealo/php-middleware-stack** provides the ```Idealo\Middleware\Stack``` class. All it has to know in order to be instantiable is:
* an instance of ```Psr\Http\Message\ResponseInterface``` as the default response
* and middlewares, that implement the ```Psr\Http\Middleware\ServerMiddlewareInterface```

To perform a sequential processing of injected middlewares you have to call stack's ```process``` method with:
* an instance of ```Psr\Http\Message\RequestInterface```.

By default stack's ```process``` method returns the injected response object. If any middleware decides to answer on it's own, than the response object of this certain middleware is returned.

Stack implements ```Interop\Http\ServerMiddleware\DelegateInterface```.

## For example

```php

<?php

// you decide what middleware you want to put in a stack.
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Interop\Http\ServerMiddleware\DelegateInterface;
use Interop\Http\ServerMiddleware\MiddlewareInterface;

class TrickyMiddleware implements MiddlewareInterface
{
    public function process(ServerRequestInterface $request, DelegateInterface $frame) : ResponseInterface
    {
        $requestBody = $request->getBody();
        try {
            // implement your middleware logic here  
        } catch (\Exception $exception){
            return new CustomExceptionResponse($exception);
        }
    
        return $frame->process($request);
    }
}

class VeryTrickyMiddleware implements MiddlewareInterface
{
    ...
}

class LessTrickyMiddleware implements MiddlewareInterface
{
    ...
}

// you define your PSR7 conform response instance
$defaultResponse = new DefaultResponse();

// you put your request into a PSR7 conform way
$request = new ServerRequest();

// and here we are
$stack = new \Idealo\Middleware\Stack(
    $defaultResponse,
    new TrickyMiddleware(),
    new VeryTrickyMiddleware(),
    new LessTrickyMiddleware()
);

$stackResponse = $stack->process($request);

// if everything goes well then
var_dump($stackResponse === $defaultResponse); // gives: true

```
