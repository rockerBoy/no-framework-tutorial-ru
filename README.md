## Руководство по созданию PHP приложения без фреймворков

### Введение

Данное руководство предназначено для тех, кто хорошо знаком с PHP, а также с азами объектно-ориентированного программированния.

Наверняка вы слышали о принципах [SOLID](https://ru.wikipedia.org/wiki/SOLID_(%D0%BE%D0%B1%D1%8A%D0%B5%D0%BA%D1%82%D0%BD%D0%BE-%D0%BE%D1%80%D0%B8%D0%B5%D0%BD%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%BD%D0%BE%D0%B5_%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5)). Если я ошибаюсь, то сейчас самое время ознакомиться с ними, прежде чем начать двигатся дальше по мануалу.

Я видел как многие задавали вопросы в чат-руме на Stack Overflow, на сколько хорош фреймворк X и стоит ли использовать его в своих проектах. В большинстве случаев ответ заключался в том, что они должны просто использовать PHP, а не фреймворк. Для многих такой ответ является крайне неприятным, потому что они не знают с чего начать.

Моя цель заключается в создании ресурса который бы послужил отправной точкой для разработчиков, которые задаются вопросом о построении PHP приложений без фреймворков. В некоторых случаях использование фреймворка бессмысленно, и писать приложение с нуля, с помощью сторонних пакетов намного проще, чем вы думаете.

**Это руководство было написано для версии PHP 7.0 или более поздних версий.** Если вы используете старую версию, обновите ее перед началом. Я рекомендую использовать [текущую стабильную версию](http://php.net/downloads.php).

Итак, [поехали](01-front-controller.md).

### Разделы

1. [Точка входа](01-front-controller.md)
2. [Composer](02-composer.md)
3. [Обработчик ошибок](03-error-handler.md)
4. [HTTP](04-http.md)
5. [Маршрутизатор](05-router.md)
6. [Контроллер](06-controller.md)
7. [Инверсия управления](07-inversion-of-control.md)
8. [Внедрение зависимостей](08-dependency-injector.md)
9. [Шаблонизатор](09-templating.md)
10. [Динамические страницы](10-dynamic-pages.md)
11. [Навигация](11-page-menu.md)
12. [Frontend](12-frontend.md)

---
