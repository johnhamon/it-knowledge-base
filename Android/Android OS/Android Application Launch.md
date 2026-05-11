# **When does Android process start?**
_An Android process is started whenever it is required._

Any time a user or some other system component requests a component (could be a service, an activity or an intent receiver) that belongs to your application be executed, the Android system spins off a new process for your app if it’s not already running. Generally processes keep running until killed by the system. Application processes are created on demand and a lot of things happen before you see your application’s launch activity up and running.

**Every app runs in its own process:** By default, every Android app runs in its own Android process which is nothing but a Linux process which gets one execution thread to start with. For example, when you click on a hyper-link in your e-mail, a web page opens in a browser window. Your mail client and the browser are two separate apps and they run in their two separate individual processes. The click event causes Android platform to launch a new process so that it can instantiate the browser activity in the context of its own process. The same holds good for any other component in an application.
# **Zygote : Spawning new life, new process**

Let’s step back for a moment and have a quick look on system start-up process. Like the most Linux based systems, at startup, the boot loader loads the kernel and starts the **_init process_**. The init then spawns the low level Linux processes called “daemons” e.g. android debug daemon, USB daemon etc. These daemons typically handle the low level hardware interfaces including radio interface.

Init process then starts a very interesting process called Zygote.

As the name implies it’s the very beginning for the rest of the Android application. This is the process which initializes a very first instance of Dalvik virtual machine. It also pre-loads all common classes used by Android application framework and various apps installed on a system. It thus prepares itself to be replicated. It stats listening on a socket interface for future requests to spawn off new virtual machines (VM)for managing new application processes. _On receiving a new request, it forks itself to create a new process which gets a pre-initialized VM instance_.

After zygote, init starts the **_runtime process_**.

The zygote then forks to start a well managed process called **_system server_**. _System server starts all core platform services e.g activity manager service and hardware services in its own context._

At this point the full stack is ready to launch the first app process — Home app which displays the home screen also known as Launcher application.

# **When a user clicks an app icon in Launcher …**

The click event gets translated into _startActivity(intent)_ and it is routed to _ActivityManagerService_ via Binder IPC. The ActvityManagerService performs multiple steps

1. The first step is to collect information about the target of the intent object. This is done by using _resolveIntent_() method on _PackageManager_ object. _PackageManager.MATCH_DEFAULT_ONLY_ and _PackageManager.GET_SHARED_LIBRARY_FILES_ flags are used by default.
2. The target information is saved back into the intent object to avoid re-doing this step.
3. Next important step is to check if user has enough privileges to invoke the target component of the intent. This is done by calling _grantUriPermissionLocked()_ method.
4. If user has enough permissions, _ActivityManagerService_ checks if the target activity requires to be launched in a new task. The task creation depends on _Intent_ flags such as _FLAG_ACTIVITY_NEW_TASK_ and other flags such as _FLAG_ACTIVITY_CLEAR_TOP_.
5. Now, it’s the time to check if the _ProcessRecord_ already exists for the process.If the _ProcessRecord_ is null, the _ActivityManager_ has to create a new process to instantiate the target component.

As you saw above many things happen behind the scene when a user clicks on an icon and a new application gets launched. Here is the full picture :
![[application_launch.webp]]
## There are three distinct phases of process launch :
1. Process Creation
2. Binding Application
3. Launching Activity / Starting Service / Invoking intent receiver …

**Process Creation:**

_ActivityManagerService_ creates a new process by invoking _startProcessLocked()_ method which sends arguments to Zygote process over the socket connection. Zygote forks itself and calls _ZygoteInit.main()_ which then instantiates _ActivityThread_ object and returns a process id of a newly created process.

Every process gets one thread by default. The main thread has a _Looper_ instance to handle messages from a message queue and it calls _Looper.loop()_ in its every iteration of _run()_ method. It’s the job of a _Looper_ to pop off the messages from message queue and invoke the corresponding methods to handle them. ActivityThread then starts the message loop by calling _Looper.prepareLoop()_ and _Looper.loop()_ subsequently.

The following sequence captures the call sequence in detail -
![[process creation sequence.webp]]
### Application binding

The next step is to attach this newly created process to a specific application. This is done by calling _bindApplication()_ on the thread object. This method sends **_BIND_APPLICATION_** message to the message queue. This message is retrieved by the _Handler_ object which then invokes _handleMessage()_ method to trigger the message specific action — _handleBindApplication()_. This method invokes _makeApplication()_ method which loads app specific classes into memory.

This call sequence is depicted in following figure.
![[applicaion binding.webp]]
### Launching an Activity

After the previous step, the system contains the process responsible for the application with application classes loaded in process’s private memory. The call sequence to launch an activity is common between a newly created process and an existing process.

The actual process of launching starts in _realStartActivity()_ method which calls _sheduleLaunchActivity()_ on the application thread object. This method sends **_LAUNCH_ACTIVITY_** message to the message queue. The message is handled by _handleLaunchActivity()_ method as shown below. Assuming that user clicks on Video Browser application. The call sequence to launch the activity is as shown in the figure.
![[activity lauching.webp]]

The Activity starts its managed lifecycle with _onCreate()_ method call. The activity comes to foreground with _onRestart()_ call and starts interacting with the user with _onStart()_ call.