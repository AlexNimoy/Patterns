# Наблюдатель (Observer)

Также известен как: Издатель-Подписчик(pub/sub), Слушатель

Определяет зависимость типа «один ко многим» между объектами таким образом,
что при изменении состояния одного объекта все зависящие от него оповещаются об этом событии.

```php
<?php

interface SubjectInterface
{
  public function attachObserver(ObserverInterface $observer);
  public function detachObserver(ObserverInterface $observer);
  public function notify();
}

interface ObserverInterface
{
  public function update(SubjectInterface $subject);
}
```

```php
class Team implements SubjectInterface
{
  private $observers = [];

  public function attachObserver(ObserverInterface $observer)
  {
    $this->observers[] = $observer;
  }

  public function detachObserver(ObserverInterface $observer)
  {
    foreach ($this->observers as $key => $obs) {
      if ($obs === $observer) {
        unset($this->observers[$key]);
        return;
      }
    }
  }

  public function notify()
  {
    foreach ($this->observers as $observer) {
      $observer->update($this);
    }
  }
}
```

```php
class Fan implements ObserverInterface
{
  private $name;

  function __construct($name) {
    $this->name = $name;
  }

  public function getName()
  {
    return $this->name;
  }

  public function update(SubjectInterface $team)
  {
    echo "Event reaction [{$this->name}]\n";
  }
}
```

## Usage

```php
$team = new Team();

$fan1 = new Fan('Fan #1');
$fan2 = new Fan('Fan #2');
$fan3 = new Fan('Fan #3');

$team->attachObserver($fan1);
$team->attachObserver($fan2);
$team->attachObserver($fan3);

$team->detachObserver($fan2);

$team->notify();
```

## Result

```
Event reaction [Fan #1]
Event reaction [Fan #3]
```
