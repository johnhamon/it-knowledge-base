Вопреки расхожему мнению, Android с самых первых версий использовал песочницы для изоляции приложений. И реализованы они были весьма интересным образом. Каждое приложение запускалось от имени отдельного пользователя Linux и, таким образом, имело доступ только к своему каталогу внутри `/data/data`.

Друг с другом и операционной системой приложения могли общаться только через IРС-механизм Binder, который требовал авторизациию на выполнение того или иного действия. В Android Binder использовался и продолжает использоваться буквально для всего: от запуска приложений до вызова функций операционной сис­темы.

Android спроектирован так, что пользователи и даже·программисты не догадыва­ются о существовании механизма Binder. Если, например, программист хочет скопировать текст в буфер обмена, он просто получает ссылку на объект-сервис и вызывает один из его методов. Под капотом фреймворк преобразует этот вызов в со­общение Binder и отправляет его в ядро через файл-устройство `/dev/binder`. Это сообщение перехватывает Service manager, который находит в своем каталоге сер­вис буфера обмена, проверяет полномочия приложения на отправку ему сообщений и, если оно имеет все необходимые права, передает сообщение ему. После получе­ния и обработки сообщения сервис буфера обмена отправляет ответ, используя все тот же Binder.

Кроме стандартных проверок полномочий Service Manager также проверяет, имеет ли приложение право на создание сервиса и может ли оно использовать ряд «опасных сервисов». И первое, и второе осуществляется с помощью проверки значения UID (идентификатор пользователя) вызывающего процесса. Если UID больше 1 0000 - приложение не имеет права регистрировать новый сервис (все сторонние приложения в Android получают UID больше 1 0000), а если UID находится в районе 99000-99999, приложение получает ограничение на использование опасных сер­ висов. Именно в эту группу попадают вкладки браузера Chrome. 

Этот же механизм применяется и для несколько других целей: с его помощью система оповещает приложения о системных событиях, таких как входящий вызов, пришедшее SMS, подключение зарядки и т. д. Приложения получают сообщения и могут на них реагировать (рис. 1 .4)
![[binder.webp]]
Система оповещения базируется на интентах (intent), специальном механизме, реализованном поверх Binder. Интенты предназначены для обмена информацией между приложениями (или ОС и приложениями), а также запуска компонентов приложений. С помощью интентов можно оповещать приложения о событиях, попросить систему открыть приложение для обработки определенных типов данных (напри­ мер, чтобы открыть определенную страницу в бразуере, достаточно послать широковещательный интент со ссылкой на страницу, и на него откликнутся все приложения, способные отображать веб-страницы, либо только дефолтовый браузер) или просто запустить компонент того или иного приложения. В частности, запуск приложений в Android осуществляется не напрямую, а с помощью интентов.

К сожалению, как и службы, интенты стали проблемой для Google и пользователей Android. Дело в том, что широковещательные интенты, используемые для уведомления приложений о событиях, приходят сразу ко всем приложениям, которые заявили, что способны на них реагировать. А чтобы приложение смогло среагировать на интент, его надо запустить. Картина получается такая: на смартфоне есть 20 приложений, которые могут реагировать на интент `android.net.conn.CONNECTIVITY_CНANGE`, и при каждом подключении/отключении от сети система запускает эти приложения, чтобы они смогли среагировать на интент.

Google исправила это недоразумение в Android 8.0. Теперь приложения могут регистрировать обработчики широковещательных интентов только во время своей работы (за небольшими исключениями).

In the Linux OS, there are [several](http://www.tldp.org/LDP/lpg/node7.html) [techniques](http://en.wikipedia.org/wiki/Inter-process_communication) to achieve IPC (Inter-process communication) like files, sockets, signals, pipes, message queues, semaphores, shared memory, etc. However, Android’s modified Linux kernel comes with a binder framework which enables an RPC (remote procedure call) mechanism between the client and server processes, where the client process can execute remote methods in the server process as if they were executed locally. So data can be passed to the remote method calls and results can be returned to the client calling thread. It appears as if the thread from the client process jumps into another (remote) process and starts executing in there (known as _Thread Migration_).
# **IPC**
- Inter-process communication (IPC) is a framework for the exchange of signals and data across multiple processes.
- Used for message passing, synchronization, shared memory, and remote procedure calls (RPC).
- Enables information sharing, computational speedup, modularity, convenience, privilege separation, data isolation, stability( Each process has its own (sandboxed) address space, typically running under a unique system ID)
- Many IPC options:-

> Files (including memory mapped)
> 
> Signals Sockets (UNIX domain, TCP/IP)
> 
> Pipes (including named pipes)
> 
> Semaphores
> 
> Shared memory
> 
> Message passing (including queues, message bus)
> 
> Intents, ContentProviders, Messenger
> 
> Binder!

# **Binder Terminology**

The Binder framework uses its own terminology to name facilities and components.

**Binder** This term is used ambiguously. The Binder refers to the overall Binder architecture, whereas a Binder refers to a particular implementation of a Binder interface.

**Binder Object** is an instance of a class that implements the Binder interface. A Binder object can implement multiple Binders.

**Binder Protocol** The Binder middleware uses a very low-level protocol to communicate with the driver.

**IBinder Interface** A Binder interface is a well-defined set of methods, properties, and events that a Binder can implement. It is usually described by the AIDL1 language.

**Binder Token** A numeric value that uniquely identifies a Binder.

# Why Binder?

- Android apps and system services run in separate processes for security, stability, and memory management reasons, but they need to communicate and share data! Security: each process is sandboxed and run under a distinct system identity, Stability: if a process misbehaves (e.g. crashes), it does not affect any other processes Memory management: “unneeded” processes are removed to free resources (mainly memory) for new ones, In fact, a single Android app can have its components run in separate processes.
- IPC to the rescue But we need to avoid the overhead of traditional IPC and avoid denial of service issues.
- Android’s libc (a.k.a. bionic) does not support System V IPCs, No SysV semaphores, shared memory segments, message queues, etc. System V IPC is prone to kernel resource leakage when a process “forgets” to release shared IPC resources upon termination Buggy, malicious code, or a well-behaved app that is low-memory.
- Binder to the rescue! Its built-in reference-counting of “object” references plus death-notification mechanism make it suitable for “hostile” environments (where low memory killer roams) When a binder service is no longer referenced by any clients, its owner is automatically notified that it can dispose of it.
- Many other features: “Thread migration” — like programming model: Automatic management of thread-pools Methods on remote objects can be invoked as if they were local — the thread appears to “jump” to the other process Synchronous and asynchronous (oneway) invocation model Identifying senders to receivers (via UID/PID) — important for security reasons Unique object-mapping across process boundaries A reference to a remote object can be passed to yet another process and can be used as an identifying token Ability to send file descriptors across process boundaries Simple Android Interface Definition Language (AIDL) Built-in support for marshaling many common data-types Simplified transaction invocation model via auto-generated proxies and stubs (Java-only) Recursion across processes — i.e. behaves the same as recursion semantics when calling methods on local objects Local execution mode (no IPC/data marshaling) if the client and the service happen to be in the same process.
- But: No support for RPC (local-only) Client-service message-based communication — not well-suited for streaming Not defined by POSIX or any other standard.
- Most apps and core system services depend on Binder Most app component life-cycle call-backs (e.g. onResume(), onDestory(), etc.) are invoked by ActivityManagerService via binder Turn off binder, and the entire system grinds to a halt (no display, no audio, no input, no sensors, …) Unix domain sockets used in some cases (e.g. RILD).

### Binder Communication Model
- As far as the client is concerned, it just wants to use the service:
![[binder2.webp]]
- While processes cannot directly invoke operations (or read/write data) on other processes, the kernel can, so they make use of the Binder driver:
![[binder communication.webp]]

Since the service may get concurrent requests from multiple clients, it needs to protect (synchronize access to) its mutable state.

The Binder framework communication is a client-server model. Each client initiates communication and waits for a response from the server. Each client would have a proxy object for the client-side communication. The server side constitutes a pool of worker threads. The server shall spawn a new thread for each new Binder request from the client. The bridge between the client and the server process is the binder driver. The **Binder driver** is a character device that is part of kernel space. This module ensures the client reaches the appropriate destination across process boundaries.

![[binder driver.webp]]


