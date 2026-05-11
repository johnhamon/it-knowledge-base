### PUT
**Метод запроса HTTP PUT** создаёт новый ресурс или заменяет представление целевого ресурса данными, представленными в теле запроса.

Разница между `PUT` и [`POST`](https://developer.mozilla.org/ru/docs/Web/HTTP/Methods/POST) в том, что `PUT` является идемпотентным, т.е. единичный и множественные вызовы этого метода, с идентичным набором данных, будут иметь тот же результат выполнения (без сторонних эффектов), в случае с `POST`, множественный вызов с идентичным набором данных может повлечь за собой сторонние эффекты.

|   |   |
|---|---|
|Запрос имеет тело|Да|
|Успешный ответ имеет тело|Нет|
|[Безопасный](https://developer.mozilla.org/ru/docs/Glossary/Safe)|Нет|
|[Идемпотентный](https://developer.mozilla.org/ru/docs/Glossary/Idempotent)|Да|
|[Кешируемый](https://developer.mozilla.org/ru/docs/Glossary/Cacheable)|Нет|
|Допускается в [HTML-формах](https://developer.mozilla.org/ru/docs/Learn/Forms)|Нет|
### PATCH
**Метод запроса HTTP `PATCH`** частично изменяет ресурс.

В какой-то степени `PATCH` можно назвать аналогом действия «обновить» из [CRUD (en-US)](https://developer.mozilla.org/en-US/docs/Glossary/CRUD "Currently only available in English (US)") (однако не следует путать HTTP и [CRUD (en-US)](https://developer.mozilla.org/en-US/docs/Glossary/CRUD "Currently only available in English (US)") — это две разные вещи). Запрос `PATCH` является набором инструкций о том, как изменить ресурс. В отличие от [`PUT`](https://developer.mozilla.org/ru/docs/Web/HTTP/Methods/PUT), который полностью заменяет ресурс.

`PATCH` может как быть идемпотентным, так и не быть, в отличие от [`PUT`](https://developer.mozilla.org/ru/docs/Web/HTTP/Methods/PUT), который всегда идемпотентен. Операция считается идемпотентной, если её многократное выполнение приводит к тому же результату, что и однократное. Например, если автоинкрементное поле является важной частью ресурса, то [`PUT`](https://developer.mozilla.org/ru/docs/Web/HTTP/Methods/PUT) перезапишет его (т.к. он перезаписывает всё), но `PATCH` может и не перезаписать.

`PATCH` (как и [`POST`](https://developer.mozilla.org/ru/docs/Web/HTTP/Methods/POST)) _может_ иметь побочные эффекты. Чтобы обозначить, что сервер поддерживает `PATCH`, можно добавить этот метод в список заголовков ответа [`Allow` (en-US)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Allow "Currently only available in English (US)") или [`Access-Control-Allow-Methods`](https://developer.mozilla.org/ru/docs/Web/HTTP/Headers/Access-Control-Allow-Methods) (для [CORS](https://developer.mozilla.org/ru/docs/Web/HTTP/CORS)).

Другим (неявным) индикатором, что метод `PATCH` разрешён, является наличие заголовка [`Accept-Patch`](https://developer.mozilla.org/ru/docs/Web/HTTP/Headers/Accept-Patch), который описывает, какой формат изменения документа принимает сервер.

|   |   |
|---|---|
|Запрос имеет тело|Да|
|Успешный ответ имеет тело|Может|
|[Безопасный](https://developer.mozilla.org/ru/docs/Glossary/Safe)|Нет|
|[Идемпотентный](https://developer.mozilla.org/ru/docs/Glossary/Idempotent)|Нет|
|[Кешируемый](https://developer.mozilla.org/ru/docs/Glossary/Cacheable)|Только если включена информация о дате последнего изменения|


https://ru.wikipedia.org/wiki/Multipart/form-data