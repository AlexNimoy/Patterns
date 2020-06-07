# Декоратор (Decorator)

[wiki](https://ru.wikipedia.org/wiki/Декоратор_(шаблон_проектирования))

Так-же: Обёртка (Wrapper)

Декоратор — это структурный паттерн, который позволяет добавлять
объектам новые поведения на лету, помещая их в объекты-обёртки.

Декоратор позволяет оборачивать объекты
бесчисленное количество раз благодаря тому,
что и обёртки, и реальные оборачиваемые объекты имеют общий интерфейс.

```php
<?php

interface Wrapper
{
  public function wrap(): string;
}
```

## Base Component

```php
class Component implements Wrapper
{
  public function wrap(): string
  {
    return "<button>Text</button>";
  }
}

```

## Base Decorator

```php
class Decorator implements Wrapper
{
  protected $component;

  public function __construct(Wrapper $component)
  {
    $this->component = $component;
  }

  public function wrap(): string
  {
    return $this->component->wrap();
  }
}

```

## Decorators

```php
class ParagraphDecorator extends Decorator
{
  public function wrap(): string
  {
    return "<p>" . parent::wrap() . "</p>";
  }
}

class SectionDecorator extends Decorator
{
  public function wrap(): string
  {
    return "<section>" . parent::wrap() . "</section>";
  }
}

```

## Client

```php
function clientCode(Wrapper $component)
{
  echo $component->wrap();
  echo "\n";
}

```


## Usage

```php

$simpleButton = new Component;

clientCode($simpleButton);

$decorated = new ParagraphDecorator($simpleButton);

clientCode($decorated);

$rowDecorated = new SectionDecorator($decorated);

clientCode($rowDecorated);
```

## Result

```
<button>Text</button>
<p><button>Text</button></p>
<section><p><button>Text</button></p></section>
```

