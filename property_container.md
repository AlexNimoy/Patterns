# Контейнер свойств (Property container)

[wikipedia](https://ru.wikipedia.org/wiki/Контейнер_свойств_(шаблон_проектирования))

Позволяет добавлять дополнительные свойства для класса
в контейнер (внутри класса), вместо расширения класса новыми свойствами.

```php
<?php

class PropertyContainer implements PropertyContainerInterface
{
  private $propertyContainer = [];

  public function addProperty($propertyName, $value)
  {
    $this->propertyContainer[$propertyName] = $value;
  }

  public function deleteProperty($propertyName)
  {
    unset($this->propertyContainer[$propertyName]);
  }

  public function getProperty($propertyName)
  {
    return $this->propertyContainer[$propertyName] ?? null;
  }

  public function updateProperty($propertyName, $value)
  {
    if(!isset($this->propertyContainer[$propertyName]))
    {
      throw new Exception("Property [{$propertyName}] is not set.");
    }

    $this->propertyContainer[$propertyName] = $value;
  }
}
```

```php
interface PropertyContainerInterface
{
  function addProperty($propertyName, $value);
  function deleteProperty($propertyName);
  function getProperty($propertyName);
  function updateProperty($propertyName, $value);
}
```

```php
class Post extends PropertyContainer
{
  private $title;

  public function getTitle()
  {
    return $this->title;
  }

  public function setTitle($title)
  {
    $this->title = $title;
  }
}
```

```php
$post = new Post();
$post->setTitle("Title#1");

$post->addProperty("Prop1", "PropText");
$post->addProperty("Prop2", "PropText2");
$post->addProperty("Prop3", "PropText3");

$post->deleteProperty("Prop2");
$post->updateProperty("Prop3", "PropText3Updated");

var_dump($post);

echo $post->getProperty("Prop1");
```

```
object(Post)#1 (2) {
  ["title":"Post":private]=>
  string(7) "Title#1"
  ["propertyContainer":"PropertyContainer":private]=>
  array(2) {
    ["Prop1"]=>
    string(8) "PropText"
    ["Prop3"]=>
    string(16) "PropText3Updated"
  }
}
PropText
```