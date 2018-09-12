[<< Инъектор зависимостей](08-dependency-injector.md) | [Динамические страницы >>](10-dynamic-pages.md)

### Шаблонизация

PHP не нуждается в сторонних механизмах шаблонизации, так как сам язык может позаботиться об этом. Но такой механизм может упростить нам жизнь. С помощью него мы могли бы провести четкую линию между логикой приложения и файлми шаблонов, которые отвечают только за вывод переменных в окно бразузера.

Можно прочесть набольшую статейку о шаблонизации [здесь](http://blog.ircmaxell.com/2012/12/on-templating.html). Также есть и [противоположное мнение](http://chadminick.com/articles/simple-php-template-engine.html). Лично у меня нету единого устоявшегося мнения касательно этой темы, по-этому я не буду советовать следовать какому-то одному подходу. Выбирайте то, что счтаете лучшим для вас.

В данном руководстве мы будем использовать PHP реализацию [Mustache](https://github.com/bobthecow/mustache.php). Давайте установим пакет: (`composer require mustache/mustache`).

Еще могу выделить [Twig](http://twig.sensiolabs.org/), как хорошую альтернативу.

Давайте посмотрим на исходный код [движка](https://github.com/bobthecow/mustache.php/blob/master/src/Mustache/Engine.php). Как вы видите, данный класс не имплементирует никакой интерфейс.

Мы конечно можем при внедрении шаблонизатора привзятся к конкретному классу, но минус данного подхода в том, что теперь мы будем тесно связаны с реализацией, а не с абстракцией.

Другими словами, весь ваш код который использует шаблонизатор будет зависеть от пакета `mustache`. Теперь если вы захотите изменить реализацию, то просто так это сделать не выйдет. Возможно вы захотите переключится на `Twig`, или возможно захотите  использовать свой собственный шаблонизатор. В следствии тесной связи с реализацией нужно будет исправить много кода.

Конечно хотелось бы не зависеть от реализации и в нашем случае это поправимо. Но вместо редактирования кода стороннего пакета предлагаю использовать [адаптер](http://en.wikipedia.org/wiki/Adapter_pattern). Звучит это намного сложнее чем есть на самом деле.

Для начала определим нужный нам интерфейс. Все мы [помним принцип разделения интерфейсов](https://ru.wikipedia.org/wiki/%D0%9F%D1%80%D0%B8%D0%BD%D1%86%D0%B8%D0%BF_%D1%80%D0%B0%D0%B7%D0%B4%D0%B5%D0%BB%D0%B5%D0%BD%D0%B8%D1%8F_%D0%B8%D0%BD%D1%82%D0%B5%D1%80%D1%84%D0%B5%D0%B9%D1%81%D0%B0), который гласит что слишком «толстые» интерфейсы необходимо разделять на более маленькие.

Итак, что должен делать наш шаблонизатор? Сейчас нам нужен только метод рендеринга. Давайте создадим папку `Template` в корневой директории проекта `src/`, где мы будем хранить все файлы связанные с шаблонизацией.

В папке `Template` создадим `Renderer.php` файл, который должен выглядеть следующим образом:

```php
<?php declare(strict_types = 1);

namespace Example\Template;

interface Renderer
{
    public function render($template, $data = []) : string;
}
```

Интерфейс готов, теперь давайте создадим реализацию для `mustache`. В этой же папке создаем файл `MustacheRenderer.php`:

```php
<?php declare(strict_types = 1);

namespace Example\Template;

use Mustache_Engine;

class MustacheRenderer implements Renderer
{
    private $engine;

    public function __construct(Mustache_Engine $engine)
    {
        $this->engine = $engine;
    }

    public function render($template, $data = []) : string
    {
        return $this->engine->render($template, $data);
    }
}
```

Как вы видите, адаптер получился очень простой.

Теперь нужно добавить алиас в `Dependencies.php`, так как инъектор пока ничего на знает о созданном ранее интерфейсе. Добавим строку:

`$injector->alias('Example\Template\Renderer', 'Example\Template\MustacheRenderer');`

Теперь можем добавить еще один аргумент к контроллеру `Homepage`:

```php
<?php declare(strict_types = 1);

namespace Example\Controllers;

use Http\Request;
use Http\Response;
use Example\Template\Renderer;

class Homepage
{
    private $request;
    private $response;
    private $renderer;

    public function __construct(
        Request $request, 
        Response $response,
        Renderer $renderer
    ) {
        $this->request = $request;
        $this->response = $response;
        $this->renderer = $renderer;
    }

...
```

Теперь перепишем метод `show`. Обратите внимание что сейчас шаблонизатор принимает массив. Это сделано для простоты примера. Также `Mustache` предоставляет возможность передавать объект контекста представления. Но к более сложным вещам мы перейдем немного позже:

```php
    public function show()
    {
        $data = [
            'name' => $this->request->getParameter('name', 'stranger'),
        ];
        $html = $this->renderer->render('Hello {{name}}', $data);
        $this->response->setContent($html);
    }
```

По умолчанию `Mustache` использует простой парсер строк. Но нам хотелось бы использовать файлы шаблонов. Все что нам нужно, это передать конструктору `Mustache_Engine` нужные параметры. Даввайте перейдем в `Dependencies.php` и напишем следующий код:

```php
$injector->define('Mustache_Engine', [
    ':options' => [
        'loader' => new Mustache_Loader_FilesystemLoader(dirname(__DIR__) . '/templates', [
            'extension' => '.html',
        ]),
    ],
]);
```

Мы передаем массив с параметрами, так как хотели бы использовать расширение `.html` вместо `.mustache`, которое установлено по умолчанию. Почему? Другие шаблонизаторы используют подобный синтаксис и если мы все же захотим изменить шаблонизатор, нам не придется переименовыввать все файлы шаблонов.

В корневой директории проекта создадим папку `templates`. Создадим файл `Homepage.html` и вставим в него код:

```
<h1>Hello World</h1>
Hello {{ name }}
```

Теперь мы можем вернутся к `Homepage` контроллеру и изменить строку где вызывается метод рендер на `$html = $this->renderer->render('Homepage', $data);`. Откроем главную страницу сайта в браузере и убедимся что все работает корректно.

[<< Инъектор зависимостей](08-dependency-injector.md)