# Команда (Command)

Также: Действие,  Транзакция,  Action,  Command

Команда — это поведенческий паттерн проектирования,
который превращает запросы в объекты,
позволяя передавать их как аргументы при вызове методов,
ставить запросы в очередь, логировать их,
а также поддерживать отмену операций.

![command](./command.png)

```php
<?php

interface Command
{
  public function execute();
}
```

## Commands

```php
class SimpleCommand implements Command
{
  private $payload;

  public function __construct($payload)
  {
    $this->payload = $payload;
  }

  public function execute()
  {
    echo "SimpleCommand: (" . $this->payload . ")\n";
  }
}

class ComplexCommand implements Command
{
  private $receiver;
  private $a;
  private $b;

  public function __construct(Receiver $receiver, $a, $b)
  {
    $this->receiver = $receiver;
    $this->a = $a;
    $this->b = $b;
  }

  public function execute()
  {
    echo "ComplexCommand.\n";
    $this->receiver->doSomething($this->a);
    $this->receiver->doSomethingElse($this->b);
  }
}
```

## Receiver

```php
class Receiver
{
  public function doSomething($a)
  {
    echo "Receiver: Working on (" . $a . ".)\n";
  }

  public function doSomethingElse(string $b)
  {
    echo "Receiver: Also working on (" . $b . ".)\n";
  }
}
```
## Invoker

```php
class Invoker
{
  private $onStart;
  private $onFinish;

  public function setOnStart(Command $command)
  {
    $this->onStart = $command;
  }

  public function setOnFinish(Command $command): void
  {
    $this->onFinish = $command;
  }

  public function doSomethingImportant()
  {
    if ($this->onStart instanceof Command) {
      $this->onStart->execute();
    }

    if ($this->onFinish instanceof Command) {
        $this->onFinish->execute();
    }
  }
}
```

## Usage

```php
$invoker = new Invoker;
$invoker->setOnStart(new SimpleCommand("Say Hi!"));
$receiver = new Receiver;
$invoker->setOnFinish(new ComplexCommand($receiver, "Send email", "Save report"));
$invoker->doSomethingImportant();

```

## Output

```
SimpleCommand: (Say Hi!)
ComplexCommand.
Receiver: Working on (Send email.)
Receiver: Also working on (Save report.)
```