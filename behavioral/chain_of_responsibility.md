# Цепочка обязанностей (Chain of responsibility)

Цепочка обязанностей — это поведенческий паттерн, позволяющий передавать запрос
по цепочке потенциальных обработчиков, пока один из них не обработает запрос.

## Признаки применения паттерна

Цепочку обязанностей можно определить по спискам обработчиков или проверок,
через которые пропускаются запросы. Особенно если порядок следования обработчиков важен.


### Пример

HTTP-запрос должен пройти через стек объектов middleware,
прежде чем приложение его обработает.
Каждое middleware может либо отклонить дальнейшую обработку запроса,
либо передать его следующему middleware.
Как только запрос успешно пройдёт все middleware,
основной обработчик приложения сможет окончательно его обработать.

## Note

Можно отметить, что такой подход — своего рода инверсия первоначального замысла паттерна.
Действительно, в стандартной реализации запрос передаётся по цепочке только в том случае,
если текущий обработчик НЕ МОЖЕТ его обработать,
тогда как middleware передаёт запрос дальше по цепочке,
когда считает, что приложение МОЖЕТ обработать запрос.

Тем не менее, поскольку middleware соединены цепочкой,
вся концепция по-прежнему считается примером паттерна CoR.

```php
<?php

abstract class Middleware
{
  private $next;

  public function linkWith(Middleware $next): Middleware
  {
    $this->next = $next;
    return $next;
  }

  public function check($email, $password): bool
  {
    if(!$this->next) {
      return true;
    }

    return $this->next->check($email, $password);
  }
}
```

### Middleware
```php
class UserExistsMiddleware extends Middleware
{
  private $server;

  public function __construct(Server $server)
  {
    $this->server = $server;
  }

  public function check($email, $password): bool
  {
    if(!$this->server->hasEmail($email)) {
      echo "UserExistsMiddleware: This email is not registered!\n";
      return false;
    }

    if(!$this->server->isValidPassword($email, $password)) {
      echo "UserExistsMiddleware: Wrong password!\n";
      return false;
    }

    return parent::check($email, $password);
  }
}

class RoleCheckMiddleware extends Middleware
{
  public function check($email, $password): bool
  {
    if($email === 'admin@example.com') {
      echo "RoleCheckMiddleware: Hello, admin!\n";
      return true;
    }

    echo "RoleCheckMiddleware: Hello, user!\n";

    return parent::check($email, $password);
  }
}

class ThrottlingMiddleware extends Middleware
{
  private $requestPerMinute;
  private $request;
  private $currentTime;

  public function __construct($requestPerMinute)
  {
    $this->requestPerMinute = $requestPerMinute;
    $this->currentTime = time();
  }

  public function check($email, $password): bool
  {
    if (time() > $this->currentTime + 60) {
      $this->request = 0;
      $this->currentTime = time();
    }

    $this->request++;

    if ($this->request > $this->requestPerMinute) {
      echo "ThrottlingMiddleware: Request limit exceeded!\n";
      die();
    }

    return parent::check($email, $password);
  }
}
```

```php
class Server
{
  private $user = [];
  private $middleware;

  public function setMiddleware(Middleware $middleware)
  {
    $this->middleware = $middleware;
  }

  public function logIn($email, $password): bool
  {
    if ($this->middleware->check($email, $password)) {
      echo "Server: Autorization has been successfull!\n";
      return true;
    }
    return false;
  }

  public function register($email, $password)
  {
    $this->users[$email] = $password;
  }

  public function hasEmail($email): bool
  {
    return isset($this->users[$email]);
  }

  public function isValidPassword($email, $password): bool
  {
    return $this->users[$email] === $password;
  }
}
```

### Usage

```php
$server = new Server();
$server->register('admin@example.com', 'admin_pass');
$server->register('user@example.com', 'user_pass');

// Chain
$middleware = new ThrottlingMiddleware(3);
$middleware
  ->linkWith(new UserExistsMiddleware($server))
  ->linkWith(new RoleCheckMiddleware);

$server->setMiddleware($middleware);


// Login
echo $server->logIn('unknown', 'pass');
echo "---\n";
echo $server->logIn('admin@example.com', 'admin_pass1');
echo "---\n";
echo $server->logIn('admin@example.com', 'admin_pass');
echo "---\n";
echo $server->logIn('admin@example.com', 'pass');
```

### Result

```
UserExistsMiddleware: This email is not registered!
---
UserExistsMiddleware: Wrong password!
---
RoleCheckMiddleware: Hello, admin!
Server: Autorization has been successfull!
1---
ThrottlingMiddleware: Request limit exceeded!
```
