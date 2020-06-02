# Делегирование (Delegation)

Основной шаблон проектирования

[https://ru.wikipedia.org/wiki/Шаблон_делегирования](https://ru.wikipedia.org/wiki/Шаблон_делегирования)

Делегирование (англ. Delegation) — основной шаблон проектирования,
в котором объект внешне выражает некоторое поведение,
но в реальности передаёт ответственность
за выполнение этого поведения связанному объекту.

Шаблон делегирования является фундаментальной абстракцией,
на основе которой реализованы другие шаблоны -
композиция (также называемая агрегацией),
примеси (mixins) и аспекты (aspects).

```php
<?php

interface MessagerInterface
{
  public function setSender($value): MessagerInterface;
  public function setRecipient($value): MessagerInterface;
  public function setMessage($value): MessagerInterface;
  public function send(): bool;
}
```

```php
abstract class AbstractMessager implements MessagerInterface
{
  protected $sender;
  protected $recipient;
  protected $message;

  public function setSender($value): MessagerInterface
  {
    $this->sender = $value;
    return $this;
  }

  public function setRecipient($value): MessagerInterface
  {
    $this->recipient = $value;
    return $this;
  }

  public function setMessage($value): MessagerInterface
  {
    $this->message = $value;
    return $this;
  }

  public function send(): bool
  {
    return true;
  }
}
```

```php
class EmailMessager extends AbstractMessager
{
  public function send(): bool
  {
    echo "Send by ".__METHOD__."\n";
    return parent::send();
  }
}

class SmsMessager extends AbstractMessager
{
  public function send(): bool
  {
    echo "Send by ".__METHOD__."\n";
    return parent::send();
  }
}
```

```php
class Messanger implements MessagerInterface
{
  private $messanger;

  public function __construct()
  {
    $this->toEmail();
  }

  // Config
  public function toEmail()
  {
    $this->messanger = new EmailMessager();
    return $this;
  }

  public function toSms()
  {
    $this->messanger = new SmsMessager();
    return $this;
  }

  // Delegate
  public function setSender($value): MessagerInterface
  {
    $this->messanger->setSender($value);
    return $this->messanger;
  }

  public function setRecipient($value): MessagerInterface
  {
    $this->messanger->setRecipient($value);
    return $this->messanger;
  }

  public function setMessage($value): MessagerInterface
  {
    $this->messanger->setMessage($value);
    return $this->messanger;
  }

  public function send(): bool
  {
    return $this->messanger->send();
  }
}
```

## Usage

```
$messanger = new Messanger();
$messanger->setSender('11111')
          ->setRecipient('22222')
          ->setMessage('message1')
          ->send();
var_dump($messanger);

$messanger->toSms()
          ->setSender('33333')
          ->setRecipient('44444')
          ->setMessage('message2')
          ->send();
var_dump($messanger);
```

## Output

```
Send by EmailMessager::send
object(Messanger)#1 (1) {
  ["messanger":"Messanger":private]=>
  object(EmailMessager)#2 (3) {
    ["sender":protected]=>
    string(5) "11111"
    ["recipient":protected]=>
    string(5) "22222"
    ["message":protected]=>
    string(8) "message1"
  }
}
Send by SmsMessager::send
object(Messanger)#1 (1) {
  ["messanger":"Messanger":private]=>
  object(SmsMessager)#3 (3) {
    ["sender":protected]=>
    string(5) "33333"
    ["recipient":protected]=>
    string(5) "44444"
    ["message":protected]=>
    string(8) "message2"
  }
}
```

