Here is the code extract from **ComponentActivity** that listen its destruction and call `ViewModelStore.clear()` :
```Kotlin
getLifecycle().addObserver(new LifecycleEventObserver() {
	@Override
	public void onStateChanged(@NonNull LifecycleOwner source,
		@NonNull Lifecycle.Event event) {
		if (event == Lifecycle.Event.ON_DESTROY) {
...
			// And clear the ViewModelStore
			if (!isChangingConfigurations()) {
				getViewModelStore().clear();
			}
		}
	}
});

```
A **ViewModelStore** keeps a reference on a **ViewModel** by maintaining a `HashMap<String, ViewModel>`.

The String parameter, named `key`, is by default `ViewModelProvider.DEFAULT_KEY:canonicalName`.

We can specify a `key` when we call `ViewModelProvider.get()`.

As described above, we pass a **ViewModelStoreOwner** when we create a **ViewModelProvider**. The **ViewModelStore** is retrieved from its owner.

When the **ViewModelProvider** creates a new **ViewModel**, it adds the **ViewModel** to the map in **ViewModelStore**.
## How does it work in a ComponentActivity
Let’s choose the **ComponentActivity** class to dig inside and see how this works.

**ComponentActivity** implements **ViewModelStoreOwner**, so it overrides `getViewModelStore()`.
```Kotlin
@NonNull
@Override
public ViewModelStore getViewModelStore() {
    if (getApplication() == null) {
        throw new IllegalStateException("Your activity is not yet attached to the "
                + "Application instance. You can't request ViewModel before onCreate call.");
    }
    ensureViewModelStore();
    return mViewModelStore;
}
```
`mViewModelStore` can’t be null. To avoid that, the method `ensureViewModelStore()` is called. If there was a previous **ViewModelStore**, it’s retrieved, otherwise, a new **ViewModelStore** is instantiated.
```Java
void ensureViewModelStore() {
    if (mViewModelStore == null) {
        NonConfigurationInstances nc =
                (NonConfigurationInstances) getLastNonConfigurationInstance();
        if (nc != null) {
            // Restore the ViewModelStore from NonConfigurationInstances
            mViewModelStore = nc.viewModelStore;
        }
        if (mViewModelStore == null) {
            mViewModelStore = new ViewModelStore();
        }
    }
}
```
The interesting part is the call to `getLastNonConfigurationInstance()`. It’s a method of the **Activity** class that returns a `NonConfigurationInstances`. The class is declared in the **ComponentActivity**:
```Java
static final class NonConfigurationInstances {
    Object custom;
    ViewModelStore viewModelStore;
}
```
The `custom` variable was used for a previous behavior and will now be `null` every time.

The second parameter is of type **ViewModelStore** and is used to keep the reference of our **ViewModelStore** during a configuration change.

The documentation of `getLastNonConfigurationInstance()` says that it returns the result of the previous call to `onRetainNonConfigurationInstance()`. 

Here is the documentation of this method:

> Called by the system, as part of destroying an activity due to a configuration change, when it is known that a new instance will immediately be created for the new configuration. You can return any object you like here, including the activity instance itself, which can later be retrieved by calling `getLastNonConfigurationInstance()` in the new activity instance. If you are targeting `HONEYCOMB` or later, consider instead using a `Fragment` with `[Fragment.setRetainInstance(boolean)`

And here is the code of the method:
```Java
public final Object onRetainNonConfigurationInstance() {
    // Maintain backward compatibility.
    Object custom = onRetainCustomNonConfigurationInstance();

    ViewModelStore viewModelStore = mViewModelStore;
    if (viewModelStore == null) {
        // No one called getViewModelStore(), so see if there was an existing
        // ViewModelStore from our last NonConfigurationInstance
        NonConfigurationInstances nc =
                (NonConfigurationInstances) getLastNonConfigurationInstance();
        if (nc != null) {
            viewModelStore = nc.viewModelStore;
        }
    }

    if (viewModelStore == null && custom == null) {
        return null;
    }

    NonConfigurationInstances nci = new NonConfigurationInstances();
    nci.custom = custom;
    nci.viewModelStore = viewModelStore;
    return nci;
}
```

We see that if there is an existing **ViewModelStore**, the method returns a new instance of `NonConfigurationInstances` containing the retrieved instance of the **ViewModelStore**.

We continue our way to find out who calls `onRetainNonConfigurationInstance()`. The caller is `Activity.retainNonConfigurationInstances()`. Here is the method body without the unnecessary code:
```Java    
NonConfigurationInstances retainNonConfigurationInstances() {
    Object activity = onRetainNonConfigurationInstance();
    ...
    NonConfigurationInstances nci = new NonConfigurationInstances();
    nci.activity = activity;
    nci.children = children;
    nci.fragments = fragments;
    nci.loaders = loaders;
    if (mVoiceInteractor != null) {
        mVoiceInteractor.retainInstance();
        nci.voiceInteractor = mVoiceInteractor;
    }
	    return nci;
    }
```
We see a class `NonConfigurationInstances`, but it’s not the same as the one of the `ComponentActivity`. This one is an inner class of `Activity`:
```Java
static final class NonConfigurationInstances {
    Object activity;
    HashMap<String, Object> children;
    FragmentManagerNonConfig fragments;
    ArrayMap<String, LoaderManager> loaders;
    VoiceInteractor voiceInteractor;
}
```
Small summary of the situation:

- our **ViewModel** is stored in a `ComponentActivity#NonConfigurationInstances`
- which is stored in an `Activity#NonConfigurationInstance`. This is done in the `retainNonConfigurationInstances()` of the **Activity** class.
## Let’s dig inside the AOSP
We can find that the caller of retainNonConfigurationInstances() is inside ActivityThread:
```Java
void performDestroyActivity(
	ActivityClientRecord r, 					
	boolean finishing,
    int configChanges, 
    boolean getNonConfigInstance, 
    String reason
) {
    ...
    if (getNonConfigInstance) {
        try {
            r.lastNonConfigurationInstances = r.activity.retainNonConfigurationInstances();
		} catch (Exception e) {
            if (!mInstrumentation.onException(r.activity, e)) {
                throw new RuntimeException("Unable to retain activity "
                        + r.intent.getComponent().toShortString() + ": " + e.toString(), e);
            }
        }
    }
    ...
}
```
The result of `retainNonConfigurationInstances()` is set in an `ActivityClientRecord`. The documentation of this class says:
>Activity client record, used for bookkeeping for the real {@link [Activity](https://cs.android.com/android/platform/superproject/+/master:frameworks/base/core/java/android/app/Activity.java;drc=79783873d85a62c8193ccc1c5ec0fc18221c751c;bpv=1;bpt=1;l=746)} instance.

 It’s a POJO containing the Activity itself, its state, and many other information. Now, we have to find where this `ActivityClientRecord` is kept. We will follow a long road across several method calls:

- `ActivityThread.handleDestroyActivity()`
- `ActivityThread.handleRelaunchActivityInner()`
- `ActivityThread.handleRelaunchActivity()`
- `ActivityRelaunchItem.execute()`
- `ActivityTransactionItem.execute()`

This last method internally calls `getActivityClientRecord()`, which calls `ClientTransactionHandler.getActivityClient()`. `ClientTransactionHandler` is an abstract class, and one of the implementations is a class we already saw: `ActivityThread`. This `ActivityThread` keeps a map of `ActivityClientRecord`. 
```Java
final ArrayMap<IBinder, ActivityClientRecord> mActivities = new ArrayMap<>();
```
So, we finally found that our **ViewModel** is stored in the `ActivityThread` that is a singleton so the **ViewModel** is not destroyed during a configuration changed. Here is the documentation of `ActivityThread`:

> This manages the execution of the main thread in an application process, scheduling and executing activities, broadcasts, and other operations on it as the activity manager requests.
## Restore the ViewModelStore
Following what we discovered just before, here is the situation:
- our **ViewModel** is stored in a `ComponentActivity#NonConfigurationInstances`
- which is stored in an `Activity#NonConfigurationInstance`.
- which is stored in an `ActivityClientRecord`.
- which is stored in `ActivityThread`.
When an **Activity** is recreated, its method `attach()` is called. One of the parameter is an `Activity#NonConfigurationInstances`. It’s retrieved from the `ActivityClientRecord` associated to the Activity. 



https://proandroiddev.com/how-viewmodel-works-under-the-hood-52a4f1ff64cf