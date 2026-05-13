`android:taskAffinity`

The task that the activity has an affinity for. Activities with the same affinity conceptually belong to the same task, to the same "application" from the user's perspective. The affinity of a task is determined by the affinity of its root activity.

The affinity determines two things: the task that the activity is re-parented to (see the `[allowTaskReparenting](https://developer.android.com/guide/topics/manifest/activity-element#reparent)` attribute) and the task that houses the activity when it launches with the `[FLAG_ACTIVITY_NEW_TASK](https://developer.android.com/reference/android/content/Intent#FLAG_ACTIVITY_NEW_TASK)` flag.

By default, all activities in an application have the same affinity. You can set this attribute to group them differently, and even place activities defined in different applications within the same task. To specify that the activity doesn't have an affinity for any task, set it to an empty string.

If this attribute isn't set, the activity inherits the affinity set for the application. See the `[<application>](https://developer.android.com/guide/topics/manifest/application-element)` element's `[taskAffinity](https://developer.android.com/guide/topics/manifest/application-element#aff)` attribute. The name of the default affinity for an application is the [namespace](https://developer.android.com/studio/build/configure-app-module#set-namespace) set in the `build.gradle` file.