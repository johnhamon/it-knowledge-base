![[user tap.png]]
Перед тем как запустить новый процесс приложения, `system_server` создает стартовое окно используя метод `PhoneWindowManager**.addSplashScreen()`.

Стартовое окно - то, что пользователь будет видеть, пока запускается само приложение. Окно будет отображаться до тех пор пока не будет запущена `Activity` и не будет отрисован первый кадр. То есть пока не будет завершен **«холодный» запуск**

Содержимое стартового окна берется из drawable-ресурсов `windowSplashscreenContent` и `windowBackground` запускаемого `Activity`
![[launcher process.png]]
Если пользователь восстанавливает `Activity` из режима **последнего экрана (Recent screen)**, при этом на нажимая на иконку приложения, то `system_server` вызывает метод `TaskSnapshotSurface.create()`, чтобы создать стартовое окно из уже сделанного скриншота.  
  
Как только стартовое окно показано пользователю, `system_server` готов запустить процесс приложения и вызывает метод `ZygoteProcess.startViaZygote()`

Когда система загружается, процесс [[Zygote]] стартует и выполняет метод `ZygoteInit.main()`. `ZygoteInit.main()` подгружает все необходимые системные библиотеки и ресурсы Android-фреймворка. Подобная предзагрузка не только экономит память, но и время запуска приложений. Далее он запускает метод `ZygoteServer.runSelectLoop()`, который в свою очередь запускает сокет, и начинает слушать вызовы данного сокета.

Когда же на сокет приходит команда на форкинг процесса, метод `ZygoteConnection.processOneCommand()` обрабатывает аргументы, и запускает метод `Zygote.forkAndSpecialize()`:
![[process forking.png]]
**На заметку:** Начиная с Android 10 есть оптимизационная фича под названием **Unspecialized App Process**, которая имеет пул не специализированных Zygote-процессов, для еще более быстрого запуска приложений.

После форка дочерний процесс запускает метод `RuntimeInit.commonInit()`, который устанавливает дефолтный `UncaughtExceptionHandler`. Далее, процесс запускает метод `ActivityThread.main()`.  Метод создает новый поток и вызывает метод `Looper.loop()`, в котором будет запущен новый инстанс **Looper**-а. Он будет привязан к новому потоку (который становится **MainThread**-ом aka UiThread) и будет работать (теоретически) бесконечно. `Looper` привязавшись, будет ожидать сообщений для того чтобы поместить их к своему `MessageQueue`. Далее метод `ActivityThread.attach()` делает IPC-запрос к методу `ActivityManagerService.attachApplication()` `system_server`-а, тем самым давая понять, что **MainThread** нашего приложения запущен и готов к работе.
![[attach app.png]]
В процессе `system_server` метод `ActivityManagerService.attachApplication()` вызывает метод `ActivityManagerService.attachApplicationLocked()`, который завершает настройку запускаемого приложения.

Парочка ключевых выводов:  
- Процесс `system_server` делает IPC-запрос к методу `ActivityThread.bindApplication()` в процессе нашего приложения, который направляет запрос к методу `ActivityThread.handleBindApplication()` в `MainThread`-е приложения.
- Сразу после этого, `system_server` планирует запуск Pending Activity, Service и BroadcastReciever нашего приложения.
- Метод `ActivityThread.handleBindApplication()` загружает APK-файл и компоненты приложения.
- Разработчики имеют возможность немного повлиять на процессы перед запуском метода `ActivityThread.handleBindApplication()`, так что именно здесь должен начаться мониторинг холодного запуска приложения.
![[bindapplication.png]]
Давайте немного подробно разберем 3-ий пункт и узнаем что и как происходит при загрузке компонентов и ресурсов приложения. Порядок шагов такой:  
- Загрузка и создание инстанса класса `AppComponentFactory`.
- Вызов метода `AppComponentFactory.instantiateClassLoader()`.
- Вызов метода `AppComponentFactory.instantiateApplication()` для загрузки и создания инстанса класса `Application`.
- Для каждого объявленного ContentProvider, в порядке приоритета, вызов метода `AppComponentFactory.instantiateProvider()` для загрузки его класса и создания инстанса, после вызов метода `ContentProvider.onCreate()`.
- И наконец, вызов метода `Application.onCreate()`.
![[app cold start.png]]

Ссылки:
https://nurlandroid.com/?p=321
https://habr.com/ru/articles/521522/
