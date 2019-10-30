## Lesson 04: Activity & Fragment Lifecycle

> Understanding the concept of Lifecycles for both activities and fragments is what makes a great Android developer! Have a treat with this delicious "[Dessert Pusher](https://github.com/udacity/andfun-kotlin-dessert-pusher)" app.

- Every activity and every fragment have what's known as a life cycle.

- The activity life cycle is made up of the different states that the activity goes through from when it's first initialized to what it's finally destroyed.

- They track things like whether the activity is on-screen or whether the user has navigated away from the activity entirely.

##### What is the use of the activity life cycle?

- Keeping your activities and fragments from using excessive amounts of the operating systems limited resources (being a good Android citizen).

- The Android OS is also watching the state of your activities to improve battery life and app performance the operating system will occasionally stop and restart apps behind the scenes, therefore you'll both want to program **proactively** and **defensively**. **Proactively** to clean up unused resources so activities on-screen run smoothly and **defensively** in case the OS does something like restart your app.

### Introduction to Logging

[Log](https://developer.android.com/reference/kotlin/android/util/Log.html) is an API for sending log output. Generally, you should use the `Log.v()`, `Log.d()`,` Log.i()`, `Log.w()`, and `Log.e()` methods to write logs. You can then view the logs in [logcat](https://developer.android.com/studio/debug/am-logcat).

Logs have different levels that are used in different situations. The levels and their usages are listed below:

- **Verbose**: Show all log messages (the default).

- **Debug**: Show debug log messages that are useful during development only, as well as the message levels lower in this list.

- **Info**: Show expected log messages for regular usage, as well as the message levels lower in this list.

- **Warn**: Show possible issues that are not yet errors, as well as the message levels lower in this list.

- **Error**: Show issues that have caused errors, as well as the message level lower in this list.

- **Assert**: Show issues that the developer expects should never happen.

The order in terms of verbosity, from least to most is ERROR, WARN, INFO, DEBUG, VERBOSE. Verbose should never be compiled into an application except during development. Debug logs are compiled in but stripped at runtime. Error, warning and info logs are always kept.

#### Add a log message

```kotlin
Log.i("MainActivity", "onCreate called")
```

#### [Timber](https://github.com/JakeWharton/timber)

Timber is a logger library with a small, extensible API that provides utility on top of Android's normal `Log` class. The benefits of using it are:

- Generate Tags. (so that you don't have to do it manually)

- Avoid showing logs in the release version of your android app.

- Allows for easy integration with crash reporting libraries.

Timber needs to use an application class because the whole app will be using this logging library. Also, you want to initialize timber before anything else is initialized that includes all activities and fragments so in cases like this. What you're actually going to do is you're going to make a subclass of application and override the defaults with your own custom implementation.

##### Setting it up

- Add the timber library in `app/build.gradle`.
  
  ```groovy
  implementation 'com.jakewharton.timber:timber:4.7.1'
  ```

- Make application class.
  
  An application class is a base class that contains global application state for your entire app.
  
  ```kotlin
  class PusherApplication : Application() {
     override fun onCreate() {
         super.onCreate()
     }
  }
  ```

- Add application class to `<application>` tag in the manifest.
  
  ```xml
  android:name=".PusherApplication"
  ```

- Initialize Timber in the application class.
  
  The installation of logging trees should be done as early as possible. The `onCreate` of your application is the most logical choice. Go ahead, locate the `onCreate` callback in your application class and plant Timber tree there.
  
  ```kotlin
  Timber.plant(Timber.DebugTree())
  ```

### Active Lifecycle Diagram

- Open your activity and then navigate out of the activity by pressing the back button. Which callbacks are called and in what order?
  
  `onCreate` -> `onStart` -> `onResume` -> `onPause` -> `onStop` -> `onDestroy`

- Open the app, then press the home button and go to the home screen. Then navigate back to the activity. Starting from when you navigate to the home screen, Which callbacks are called and in what order?
  
  `onPause` -> `onStop` -> `onRestart` -> `onStart` -> `onResume`

- Open up the share dialog, then click outside of the share dialog to close it. Which callbacks are called in the activity and in what order?
  
  `onPause` -> `onResume`

### General Definitions

- **Visible Lifecycle:** The part of the Lifecycle between `onStart` and `onStop` when the Activity is visible.

- **Focus:** An Activity is said to have focus when it's the activity the user can interact with.

- **Foreground:** When the activity is on screen.

- **Background:** When the activity is fully off-screen, it is considered in the background.

### Lifecycle States

These are the same for both the Fragment Lifecycle and the Activity Lifecycle.

- **Initialized:** This is the starting state whenever you make a new activity. This is a transient state -- it immediately goes to Created.

- **Created:** Activity has just been created, but it’s not visible and it doesn’t have focus (you’re not able to interact with it).

- **Started:** Activity is visible but doesn’t have focus.

- **Resumed:** The state of the activity when it is running. It’s visible and has focus.

- **Destroyed:** Activity is destroyed. It can be ejected from memory at any point and should not be referenced or interacted with.

### Activity Lifecycle Callbacks

**onCreate:** This is called the first time the activity starts and is therefore only called once during the lifecycle of the activity. It represents when the activity is created and initialized. The activity is not yet visible and you can't interact with it. You must implement onCreate. In onCreate you should:

- Inflate the activity's UI, whether that's using `findViewById` or databinding.
- Initialize variables.
- Do any other initialization that only happens once during the activity lifetime.

**onStart:** This is triggered when the activity is about to become visible. It can be called multiple times as the user navigates away from the activity and then back. Examples of the user "navigating away" are when they go to the home screen, or to a new activity in the app. At this point, the activity is not interactive. In `onStart` you should:

- Start any sensors, animations or other procedures that need to start when the activity is visible.

**onResume:** This is triggered when the activity has focus and the user can interact with it. Here you should:

- Start any sensors, animations or other procedures that need to start when the activity has focus (the activity the user is currently interacting with).

**onPause:** The mirror method to `onResume`. This method is called as soon as the activity loses focus and the user can't interact with it. An activity can lose focus without fully disappearing from the screen (for example, when a dialog appears that partially obscures the activity). Here you should:

- Stop any sensors, animations or other procedures that should not run when the activity doesn't have focus and is partially obscured.
- Keep execution fast. The next activity is not shown until this completes.

**onStop:** This is the mirror method to `onStart`. It is called when you can no longer see the activity. Here you should:

- Stop any sensor, animations or other procedures that should not run when the activity is not on screen.
- You can use this to persist (permanently save) data, which you’ll be learning more about in Lesson 6
- Stop logic that updates the UI. This should not be running when the activity is off-screen; it's a waste of resources.
- There are also restrictions as soon as the app goes into the background, which is when all activities in your app are in the background. We'll talk more about this in Lesson 9.

**onDestroy:** This is the mirror method to `onCreate`. It is called once when the activity is fully destroyed. This happens when you navigate back out of the activity (as in press the back button), or manually call `finish()`. It is your last chance to clean up resources associated with the activity. Here you should:

- Tear down or release any resources that are related to the activity and are not automatically released for you. Forgetting to do this could cause a memory leak! The logic that refers to the activity or attempts to update the UI after the activity has been destroyed could crash the app!

### Summary of the Fragment Lifecycle

Fragments also have lifecycle states that they go-between. The lifecycle states are the same as the activity states. You’ll notice that in your Android Trivia app, you’re using the `onCreateView` callback - while the fragment lifecycle states are the same, **the callbacks are different**.

A deep dive into the fragment lifecycle could be a lesson in itself. Here, we’ll just cover the basics with the summary below:

#### Important Fragment Callbacks to Implement

**onCreate:** Similar to the Activity’s `onCreate` callback. This is when the fragment is created. This will only get called once. Here you should:

- Initialize anything essential for your fragment.
- **DO NOT inflate XML**, do that in `onCreateView`, when the system is first drawing the fragment.
  NOT reference the activity, it is still being created. Do this in `onActivityCreated`.

**onCreateView:** This is called between `onCreate` and `onActivityCreated`. when the system will draw the fragment for the first time when the fragment becomes visible again. You must return a view in this callback if your fragment has a UI. Here you should:

- Create your views by inflating your XML.

**onStop:** Very similar to Activity’s `onStop`. This callback is called when the user leaves your fragment. Here you should:

- Save any permanent fragment state (this will be discussed in lesson 6)

#### Other callbacks

**onAttach:** When the fragment is first attached to the activity. This is only called once during the lifecycle of the fragment.

**onActivityCreated:** Called when the activity `onCreate` method has returned and the activity has been initialized. If the fragment is added to an activity that's already created, this still gets called. Its purpose is to contain any code the requires the activity exists and it is called multiple times during the lifecycle of the fragment. Here you should:

- Execute any code that requires an activity instance

**onStart:** Called right before the fragment is visible to the user.

**onResume:** When the activity resumes the fragment. This means the fragment is visible, has focus and is running.

**onStop:** Called when the Activity’s `onStop` is called. The fragment no longer has focus.

**onDestroyView:** Unlike activities, **fragment views are destroyed every time they go off-screen**. This is called after the view is no longer visible.

- Do not refer to views in this callback, since they are destroyed

**onDestroy:** Called when the Activity’s `onDestroy` is called.

**onDetach:** Called when the association between the fragment and the activity is destroyed.

#### Lifecycle [Cheat sheets](https://github.com/JoseAlcerreca/android-lifecycles)

What you’ve seen up to this point are the Activity Lifecycle and the Fragment Lifecycle for a single Activity or Fragment. For more complicated apps, it becomes important to understand the interactions between Activity and Fragment life cycles and multiple activities. This is outside of the scope of this lesson, but there are a series of excellent blog posts and cheat sheets posted by Googler which are helpful references for this:

- [**The Android Lifecycle cheat sheet — part I: Single Activity**](https://medium.com/androiddevelopers/the-android-lifecycle-cheat-sheet-part-i-single-activities-e49fd3d202ab) - This is a visual recap of much of the material here.
- [**The Android Lifecycle cheat sheet — part II: Multiple Activities**](https://medium.com/androiddevelopers/the-android-lifecycle-cheat-sheet-part-ii-multiple-activities-a411fd139f24) - This shows the order of lifecycle calls when two activities interact.
- [**The Android Lifecycle cheat sheet — part III: Fragments**](https://medium.com/androiddevelopers/the-android-lifecycle-cheat-sheet-part-iii-fragments-afc87d4f37fd) - This shows the order of lifecycle calls when activity and fragment interact.

### Introduction to the [Lifecycle](https://developer.android.com/topic/libraries/architecture/lifecycle) Library

- When you set up a lot of things in `onStart` or `onCreate` and then tear down all those things in `onStop` or `onDestroy`. For example, maybe you have animations and music and sensors and timers that you need to set up and tear down. If you forgot a one that leads to bugs and headaches.

- It would be nice if those classes that manage timers, music, sensors, and animation could be in charge of themselves. For example, if our dessert timer could be in charge of stopping and starting itself at the exact correct times well hold that thought.

- In 2017 Google announced the lifecycle library to simplify working with the activity and fragment lifecycle. Before the lifecycle library, the only way to interact with a fragment or activity lifecycle was through the callback methods.

- The lifecycle library introduced the Android lifecycle as an actual object that can be queried and checked for the state, it also introduced the lifecycle owner interface and the lifecycle observer interface.

- **LifecycleOwner** are anything that has a lifecycle, represented by that lifecycle object. Ex. activities and fragments.

- **LifecycleObserver** observes the life cycle of a lifecycle owner such as an activity.

So, Lifecycle-aware components perform actions in response to a change in the lifecycle status of another component, such as activities and fragments. These components help you produce better-organized, and often lighter-weight code, that is easier to maintain.

#### [Lifecycle Observation](https://developer.android.com/topic/libraries/architecture/lifecycle)

- Observer pattern is a software design pattern that involves an object called a subject which keeps track of a list of other objects called observers.

- The observers are said to be watching or observing the subject. When the state of the subject changes it notifies its list of all observers.

- So if we think about our code, the activity is currently in charge of telling the timer to start and stop but if we were using the observer pattern we'd actually flip that responsibility of the relationship. Instead, the timer would become an observer of the activity as the activity state changes the timer would be notified it would then be the timer's responsibility to decide what to actually do with that information.
  In this case, it could call its own start and stop methods on itself.
1. Make DessertTimer a LifecycleObserver:
   
   DessertTimer should implement a LifecycleObserver, take in a Lifecycle as a parameter.
   
   ```kotlin
   class DessertTimer(lifecycle: Lifecycle) : LifecycleObserver {
   ```
   
   Then establish observer relationship in init block.
   
   ```kotlin
    init {
        lifecycle.addObserver(this)
    }
   ```

2. Pass in 'this' MainActivity's lifecycle so that it is observed:
   
   ```kotlin
   dessertTimer = DessertTimer(this.lifecycle)
   ```

3. Annotate startTimer and stopTimer with @OnLifecycleEvent and the correct event:

### Process Shutdown

- When the OS restarts your app for you Android tries it's best to reset your app to the state that I had before. The Android OS takes the state of some of your views and saves them to a bundle whatever you navigate away from the activity.

- An example of some of the data that's automatically saved for you is the text in your edit text as long as that edit text has its ID set it'll save it for you it also saves things like the back stack of your activity.

- When the OS restarts the app for you it's going to use this data bundle to try to get your activity back to the same state as when you left it, except sometimes the OS won't know all about your data.

- For example, in the desert pusher app, you have custom variables like revenue there's no way that the Android OS actually knows about this data or that it's important to you or your activity. So you're gonna need to manually add this data to the bundle yourself.

- The way you do that is using another callback called `onSaveInstanceState`.

#### Terminating a process

1. Make sure you are using a device or emulator running at least API level 28. This is easier to set up for an emulator.
2. Make sure adb is installed.
3. Make sure your app is in the background - this only happens when the app is in the background. You can do this by hitting the **home** button.
4. Run the command:

```bash
adb shell am kill com.example.android.dessertpusher
```

This will stop the process as-if it had been stopped by the Android operating system.

#### Adding ADB to your path

[ADB (Android Debug Bridge)](https://developer.android.com/studio/command-line/adb) is a command-line tool, and if you want to use it from the command line, it needs to be part of your path. First, you’ll find where the adb executable lives, then you’ll add that to your path.

1. Find the platform-tools folder which contains adb:

Adb is part of the Android SDK, which is downloaded as part of Android Studio. You can find the location of this SDK by going to **Tools -> SDK Manager**.

ADB is located in this location, followed by platform-tools/ so in the example above, you could find adb in:

/Users/lmf/Library/Android/sdk/platform-tools/

2. Add adb to your path:

Adding a variable to your path varies by platform, follow the instructions below to add the platform-tools location you located above.

**Windows**

1. Go to **Advanced system settings:**
   - Windows 8 and 10: **Search -> System (Control Panel) -> Advanced system settings**
   - Windows 7: Right-click **Computer -> Properties -> Advanced system settings**
   - Windows Vista: Right click **My Computer -> Properties -> Advanced system settings**
   - Windows XP: **Start -> Control Panel -> System -> Advanced tab**
2. Click **Environment Variables**
3. Find the **System Variables** section and then look to see if you have a **PATH** environment variable:
   - If you find one, click **Edit**
   - If you do not find one, click **New** to add one
4. Add `;<Path to platform-tools>` to the end of the **Variable** value box
5. Click **OK** on all windows to save
6. Ensure you can run adb by typing:
   - `adb`

**Mac/Linux**

Adding a path variable is done using the terminal on Mac/Linux.

1. Open a **Terminal**
2. Create a **.bash_profile** file if you don’t have one already. This is a configuration file for [bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)) - it’s executed when you start bash:
   - `touch ~/.bash_profile`
3. Open up the ~/.bash_profile file in your preferred text editor:
   - `open ~/.bash_profile`
4. Add the following to your .bash_profile file and save:
   - `export PATH=<Path to platform-tools>:$PATH`
5. Either restart your terminal, or enter:
   - `source ~/.bash_profile`
6. Ensure you can run adb by typing:
   - `adb`

#### [onSaveInstanceState](https://developer.android.com/reference/kotlin/android/app/Activity.html#onsaveinstancestate_1)

This method is called before an activity may be killed so that when it comes back some time in the future it can restore its state. For example, if activity B is launched in front of activity A, and at some point activity A is killed to reclaim resources, activity A will have a chance to save the current state of its user interface via this method so that when the user returns to activity A, the state of the user interface can be restored via `onCreate` or `onRestoreInstanceState`. 

###### Best Practice:

Store only data smaller than 100KB.

1. Override `onSaveInstanceState` and `onRestoreInstanceState`:
   
   You can use the keyboard shortcut Ctrl + O to do this. For `onSaveInstanceState`, you can use the [version that has one parameter](https://developer.android.com/reference/android/app/Activity.html#onSaveInstanceState(android.os.Bundle)), the Bundle (`onSaveInstanceState(outState: Bundle, outPersistentState: PersistableBundle): Unit`).
   
   ```kotlin
       override fun onSaveInstanceState(outState: Bundle) {
           Timber.i("onSaveInstanceState Called")
           super.onSaveInstanceState(outState)
       }
   ```

2. Save the data in `onSaveInstanceState`:
   
   You should save the `revenue`, `dessertsSold` and `dessertTimer.secondsCount` in the state Bundle.
   
   ```kotlin
   outState.putInt("key_revenue", revenue)
   ```
   
   It's preferred to store keys in constants.
   
   ```kotlin
   /** onSaveInstanceState Bundle Keys **/
   const val KEY_REVENUE = "revenue_key"
   const val KEY_DESSERT_SOLD = "dessert_sold_key"
   const val KEY_TIMER_SECONDS = "timer_seconds_key"
   ```
   
   And thus, the will be like:
   
   ```kotlin
           outState.putInt(KEY_REVENUE, revenue)
           outState.putInt(KEY_DESSERT_SOLD, dessertsSold)
           outState.putInt(KEY_TIMER_SECONDS, dessertTimer.secondsCount)
   ```

3. Retrieve and use the data in `onCreate` if you're restarting to Activity:
   
   Check that the Bundle is not null in `onCreate`, and then retrieve and use the data. For example:
   
   ```kotlin
   if (savedInstanceState != null) {
           revenue = savedInstanceState.getInt(KEY_REVENUE, 0)
           dessertsSold = savedInstanceState.getInt(KEY_DESSERT_SOLD, 0)
           dessertTimer.secondsCount = savedInstanceState.getInt(KEY_TIMER_SECONDS, 0)
        }
   ```

### Configuration Changes

- Rotate your phone and record the callbacks triggered:
  
  `onPause` -> `onStop` -> `onDestroy` -> `onCreate` -> `onStart` -> `onResume`

- A configuration change is where the state of the device changes so radically that the system decides the easiest way to resolve this is to rebuild the activity. It uses the same `onSaveInstanceState` bundle to restore the activity which is why implementing `onSaveInstanceState` fixed that rotation problem.

- Examples of configuration changes include:
  
  - User changed device language.
  
  - User plugs in or removes a hardware keyboard.
  
  - User rotates device.

- Android is flexible enough that when you choose to rotate your phone you could show the user a completely different UI with a completely different view hierarchy thus if you start off in portrait orientation and then rotate your phone the device actually tears down any activities in portrait orientation and recreates them in landscape orientation.

- What happens to an activity if you override `onSaveInstanceState` and save some of your data, but don't call the super method? When the activity restarts after a configuration change, it could be missing some data, such as the edit text data.

- `onSaveInstanceState` automatically saves some data for you, such as EditTexts (as long as they have an id). But if you remove the call to the super method, the superclass won't be able to automatically do this for you. Always remember to call `super.onSaveInstanceState(outState)` when overriding `onSaveInstanceState`!
