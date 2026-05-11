URI - унифицированный индикатор ресурса.
Deep link - это URI произвольной схемы, переносящий пользователей в наше приложение.
URI принято разделять на URL (uniform resource locator) и URN (uniform resource name)
![[URI classif.png]]
Виды идентификаторов ресурсов, их описание и примеры.

|   |   |   |
|---|---|---|
|**Имя**|**Описание**|**Пример**|
|[XRI](http://docs.oasis-open.org/xri/2.0/specs/xri-syntax-V2.0.html)|Обобщенный URI для унификации многообразия идентификаторов|xri://broadview.library.example.com/(urn:isbn:0-395-36341-1)|
|[IRI](https://ru.wikipedia.org/wiki/Internationalized_Resource_Identifier)|URI с локализацией для удобства использования в не англоговорящих странах|[http://ru.wikipedia.org/wiki/](http://ru.wikipedia.org/wiki/)Кириллица|
|[URI](https://datatracker.ietf.org/doc/html/rfc3986#section-1.1.3)|Единый идентификатор ресурса|mailto:John.Doe@example.com|
|[URL](https://en.wikipedia.org/wiki/URL)|Единый локатор ресурса. Указывает на место, где ресурс расположен|file://C:\UserName.HostName\Projects\Wikipedia_Articles\URI.xml|
|[URN](https://en.wikipedia.org/wiki/Uniform_Resource_Name)|Единое имя ресурса. Независимо от его расположения|urn:isbn:0451450523|
|[URC](http://sti15.com/bib/formats/urc.html)|Единая характеристика ресурса. Содержит метаинформацию о ресурсе / описание ресурса|URN:IANA:626:oit:cs:ftp-and-telnet; URL:[http://www.gatech.edu/oit/cs/ftp-and-telnet.html](http://www.gatech.edu/oit/cs/ftp-and-telnet.html); <br><br>Author: Arrowood, Adam;|
|[PURL](https://ru.wikipedia.org/wiki/PURL)|Постоянный URL. Защищает от перемещения ресурса (404)|[http://purl.russian-books.com/WarAndPeace/chapter12.html](http://purl.russian-books.com/WarAndPeace/chapter12.html)|
|[PRURL](https://www.paulirish.com/2010/the-protocol-relative-url/)|URL, способный определить схему по контексту использования|//domain.com/img/logo.png|
|[CURIE](https://en.wikipedia.org/wiki/CURIE)|Compact URI. Сокращенное представление URI|[isbn:0393315703]|

![[URI table.png]]
![[URI scheme.png]]
URI состоит из обязательных параметров:
- scheme — определяет контекст / смысл идентификатора
- path — иерархические данные для идентификации ресурса.

И опциональных:
- userinfo — информация о том, кто взаимодействует с ресурсом.
- host — адрес расположения ресурса.
- port — точка взаимодействия с host / конкретный процесс или сервис.
- query — неиерархические данные для идентификации ресурса.
- fragment — идентификация вторичного ресурса внутри исходного ресурса.

Подробнее о параметрах можно почитать [тут](https://datatracker.ietf.org/doc/html/rfc3986#section-3) и [тут](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier#Syntax). 

Если схема URI зарегистрирована в [IANA](https://en.wikipedia.org/wiki/Internet_Assigned_Numbers_Authority), то она считается официальной. Но есть неофициальные схемы, которые [сыскали популярность в сообществе](https://en.wikipedia.org/wiki/List_of_URI_schemes#Unofficial_but_common_URI_schemes)
### Мобильные deep links
Это подвид неофициальных схем, используемых для входа в мобильные приложения.

При нажатии на ссылку система показывает *(не всегда)* пользователю **disambiguation dialog.** В нем отображается список приложений-кандидатов для обработки перехода по ссылке. Список формируется в зависимости от ссылки и intent-filter установленных приложений. Кроме списка приложений пользователю доступны две опции:
- **Just once (Только сейчас)** открывает выбранное приложение и попросит заново выбрать из списка кандидатов при повторном нажатии. При этом система может запомнить предыдущий выбор и показать его в самом вверху диалога (возможно, для удобства пользователя).
- **Always (Всегда)** запомнит выбор и больше не будет просить его сделать. В настройках приложения можно будет найти ссылки, которые всегда обрабатываются приложением и изменить поведение при необходимости.
#### Web link
Подвид deep link, который может иметь только http и https схемы.

Очень важный момент: на Android 11 и ниже клик по web link может быть обработан вашим приложением, **но, начиная с 12 версии, web link открываются только в браузере**. Это может быть связано с тем, что Google настаивает на использовании Android App Links.
#### Android App Link
В Android 6.0 появилась возможность выполнять переход по deep link без disambiguation dialog. Эта фича дает пользователю возможность не совершать дополнительных действий и сделать прямой переход в приложение, улучшая UX. Именно с версии 6.0 и появились Android App Link.

**Q:** Почему app link это web link? 
**A:** Потому что app link связывается с веб-доменом по протоколу http ([ссылка](https://developer.android.com/training/app-links/verify-site-associations)).
**https://**domain.name/.well-known/assetlinks.json
**Q:** Зачем связываться с веб-доменом? 
**A:** Для того, чтобы выполнить верификацию приложения, то есть указать системе, что именно это приложение ответственно за выбранный домен ([“... to associate an app with a web domain they own."](https://chris.orr.me.uk/android-app-linking-how-it-works/)). Получается, после успешной верификации система знает, кому адресовать переход по ссылке, и может не открывать disambiguation dialog.
https:/**/domain.name/**.well-known/assetlinks.json
**Q:** Как происходит связывание между приложением и веб-доменом? 
**A:** Автоматически (“...contain the autoVerify attribute”). Посмотрим на схему. Во время установки приложения система по autoVerify атрибуту в intent-filter понимает, что необходимо выполнить верификацию. Так как приложение обычно скачивается из сети (Google Play Store, например), это самый удобный момент для выполнения http запроса на специальный ресурс (`/.well-known/assetlinks.json`, который описан в спецификации [Digital Asset Link](https://developers.google.com/digital-asset-links/v1/getting-started)). В случае успеха система выполняет ассоциацию. Подробнее [здесь](https://chris.orr.me.uk/android-app-linking-how-it-works/).
![[linking app and web domain.png]]
Схема связывания приложения и веб-домена в момент установки. Связывание выполняется один раз - во время установки приложения.

Ранее было сказано, что web link не откроется на Android 12. **Но web link можно превратить в app link и** [**обойти это ограничение**](https://developer.android.com/training/app-links#web-links)**!**

Также доступен поиск по контенту, на который есть deep link. На примере видно, что при вводе “Kenny” в поисковую строку у нас есть возможность перейти в другое приложение, для которого настроен deep link для обработки этой информации. Подробнее [тут](https://developer.android.com/training/app-indexing) и [тут](https://moz.com/blog/how-to-get-your-app-content-indexed-by-google)

Также deep link используются в дополнении к [instant apps](https://developer.android.com/topic/google-play-instant), делая вход в ваше приложении более нативным и прозрачным ([ссылка](https://developer.android.com/training/app-links/instant-app-links)). 

В силу ограничений Android мы не можем использовать параметры: userinfo, query, fragment. Поэтому URI формат урезается до нижней схемы. Обратите внимание, что в ней scheme и host стали обязательными (в отличие от scheme и path в основной спецификации).
![[android uri scheme.png]]
Получается, что из-за ограничений формата URI Android не может обрабатывать все их виды. Из рассмотренных ранее примеров могут быть обработанными только: URL, PURL, Clean URL, а значит:
> “Deep links are URIs of any scheme that take users to your app”

заменяется на:
>“Deep links are ~~URIs~~ **URLs (but not all)** of any scheme that take users to your app”

и классификация идентификаторов принимает вид:
![[new identifiers classif.png]]
Deep Link — технология, пусть и самая старая, но базовая. Deep Link используется в уведомлениях, при поиске контента, в instant apps, передаче данных внутри приложения или между ними и т.д. Основная задача — перенести пользователя в мобильное приложение. Web Link нацелен на привлечение трафика из web в mobile. Поскольку http ссылки сильно распространены, именно такие ссылки дают возможность контенту шире распространяться в интернете. Android App Link — это современный и более продуманные с технической точки зрения Web Link. Нацелен на улучшение пользовательского опыта и безопасность.
![[app web deep link.png]]
Android накладывает ограничения на формат URI. Доступно всего четыре параметра для настройки:
- Два обязательных: scheme, host
- Два опциональных: port, path

Чтобы не допустить ошибки в ваших deep link и сделать их список более наглядным, рекомендуем обратить внимание на Deep Link Tree. Для его построения начните с определения scheme, далее соедините их с host, потом с path. Deep Link Tree может быть представлен графически, простым списком URL, XML-разметкой / программным кодом в intent-filter.

Ссылки:
https://habr.com/ru/articles/686990/
https://developer.android.com/training/app-links


