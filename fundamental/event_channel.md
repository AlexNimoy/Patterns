# Канал событий (event channel)

[Канал_событий](https://ru.wikipedia.org/wiki/Канал_событий_(шаблон_проектирования))

* Расширяет шаблон Publish/Subscribe, создавая централизованный канал для событий.
* Использует объект-представитель для подписки и объект-представитель для публикации события в канале.
* Представитель существует отдельно от реального издателя или подписчика.
* Подписчик может получать опубликованные события от более чем одного объекта, даже если он зарегистрирован только на одном канале.

```php
<?php

interface SubscriberInterface
{
  public function notify($data);
}

interface PublisherInterface
{
  public function publish($data);
}

interface EventChannelInterface
{
  public function publish($topic, $data);
  public function subscribe($topic, SubscriberInterface $subscriber);
}
```

```php
class EventChannel implements EventChannelInterface
{
  private $topics = [];

  public function subscribe($topic, SubscriberInterface $subscriber)
  {
    $this->topics[$topic][] = $subscriber;

    echo "[{$subscriber->getName()}] subscribed [{$topic}]\n";
  }

  public function publish($topic, $data)
  {
    if (empty($this->topics[$topic])) {
      return;
    }

    foreach($this->topics[$topic] as $subscriber) {
      $subscriber->notify($data);
    }
  }
}
```

```php
class Publisher implements PublisherInterface
{
  private $topic;
  private $eventChannel;

  public function __construct($topic, EventChannelInterface  $eventChannel)
  {
    $this->topics = $topic;
    $this->eventChannel = $eventChannel;
  }

  public function publish($data)
  {
    $this->eventChannel->publish($this->topics, $data);
  }
}
```

```php
class Subscriber implements SubscriberInterface
{
  private $name;

  public function __construct($name)
  {
    $this->name = $name;
  }

  public function notify($data)
  {
    echo "[{$this->getName()}] notified with [{$data}]\n";
  }

  public function getName()
  {
    return $this->name;
  }
}
```

## Usage

```php
$newsChanel = new EventChannel();

// Publishers
$rubyGroup = new Publisher('ruby-news', $newsChanel);
$railsGroup = new Publisher('ruby-news', $newsChanel);
$phpGroup = new Publisher('php-news', $newsChanel);

// Subscribers
$dmitriy = new Subscriber('Dimon');
$artem = new Subscriber('Artem');
$alex = new Subscriber('Alex');

// Subscribe
$newsChanel->subscribe('ruby-news', $artem);
$newsChanel->subscribe('php-news', $dmitriy);
$newsChanel->subscribe('ruby-news', $alex);
$newsChanel->subscribe('php-news', $alex);

$rubyGroup->publish("Matz published a new version of ruby");
$railsGroup->publish("DHH won the race");
$phpGroup->publish("Zend has announced a new version");
```

## Result

```
[Artem] subscribed [ruby-news]
[Dimon] subscribed [php-news]
[Alex] subscribed [ruby-news]
[Alex] subscribed [php-news]
[Artem] notified with [Matz published a new version of ruby]
[Alex] notified with [Matz published a new version of ruby]
[Artem] notified with [DHH won the race]
[Alex] notified with [DHH won the race]
[Dimon] notified with [Zend has announced a new version]
[Alex] notified with [Zend has announced a new version]
```
