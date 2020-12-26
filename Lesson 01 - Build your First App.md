# Android with Kotlin Course Notes

## Lesson 01: Build your First App

> Build your first app: "[Dice Roller](https://github.com/udacity/andfun-kotlin-dice-roller)" that covers basic Android components like displaying texts and images as well as a tour of the Android tools you'll be using throughout this course.

##### Android project contains:

- Kotlin files for the core logic of the app
- A resources folder for static content such as images and strings
- An Android Manifest file that defines essential app details so the OS can launch your app
- Gradle scripts, for building and running your app

***

### [AndroidManifest.xml](https://developer.android.com/guide/topics/manifest/manifest-intro)

```xml
  package="com.example.diceroller"
```

The name in the manifest's `package` attribute should always match your project's base package name where you keep your activities and other app code.

The `package` attribute also represents your app's universally unique application ID.

```xml
<manifest package="com.example.myapp" ... >
  <application ... >
      <activity android:name=".MainActivity" ... >
          <intent-filter>
              <action android:name="android.intent.action.MAIN" />
              <category android:name="android.intent.category.LAUNCHER" />
          </intent-filter>
      </activity>
  </application>
</manifest>
```

This intent-filter tells the OS where to start the app when the user clicks on the app icon. 

***

### [Activity](https://developer.android.com/guide/topics/manifest/activity-element) And [Layout](https://developer.android.com/reference/kotlin/android/text/Layout.html)

- Main activity is an example of an activity.
- An activity is a core Android class that is responsible for drawing an app user interface and receiving input events.
- When your app launches it launches a specific activity this is the activity that was declared in the manifest with the correct intent-filter tag.
- Activities have an Associated layout file which in this case is activity underscore main.
- Layout files are XML files that express what the app actually looks like they do this by defining things like text images and buttons and where these things will appear on the screen. these text images and buttons are called views.
- The activity and the layout are connected by a process known as layout inflation this process is triggered when the activity starts.

#### [setContentView](https://developer.android.com/reference/kotlin/android/app/Activity.html)

Set the activity content from a layout resource. The resource will be inflated, adding all top-level views to the activity.

```kotlin
setContentView(R.layout.activity_main)
```

#### [Linear Layout](https://developer.android.com/reference/kotlin/android/widget/LinearLayout)

`LinearLayout` is a view group that aligns all children in a single direction, vertically or horizontally. You can specify the layout direction with the `android:orientation` attribute.

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >
    ...
</LinearLayout>
```

```xml
android:layout_gravity
```

[Gravity](https://developer.android.com/reference/kotlin/android/widget/LinearLayout.LayoutParams.html) specifies how a component should be placed in its group of cells.

```xml
<Button
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="center_horizontal"
    android:text="@string/roll" />
```

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_gravity="center_vertical"
    android:orientation="vertical"
    tools:context=".MainActivity">
```

###### Best Practice:

Always extract hard-coded text to strings.xml file. This will make changing strings easier and allows you to localize your app.

***

### Calling elements

By using `findViewById`to get a reference to the element.

- in xml layout file:
  
```xml
android:id="@+id/roll_button"
```

- in Kotlin file, set a [button](https://developer.android.com/reference/kotlin/android/widget/Button.html) with:
  
```kotlin
val rollButton: Button = findViewById(R.id.roll_button)
```

- To change a property of the element you can call methods directly:
  
```kotlin
rollButton.text = "Let's Roll"
```

***

### [OnClickListener](https://developer.android.com/reference/kotlin/android/view/View.OnClickListener.html)

Interface definition for a callback to be invoked when a view is clicked.

- setting a button OnClickListener
  
```kotlin
rollButton.setOnClickListener {
  ....
}
```

***

### [Toast](https://developer.android.com/reference/kotlin/android/widget/Toast.html)

A toast is a view containing a quick little message for the user.

- Showing a Toast message
  
```kotlin
Toast.makeText(this, "button clicked", Toast.LENGTH_SHORT).show()
```

***

### [ImageView](https://developer.android.com/reference/kotlin/android/widget/ImageView.html)

Displays image resources, also commonly used to apply tints to an image and handle image scaling.

```xml
<ImageView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:src="@drawable/my_image"
    android:contentDescription="@string/my_image_description"
/>
```

```kotlin
val diceImage: ImageView = findViewById(R.id.dice_image)
diceImage.setImageResource(drawableResource)
```

###### Best Practice:

Always use lateinit to access certain element variable efficiently. Declare it at the beginning of the class.

```kotlin
lateinit var diceImage: ImageView
diceImage = findViewById(R.id.dice_image)
```

***

### XML namespace

- XML namespaces are used for providing uniquely named elements and attributes in an XML document.

- Android Studio supports a variety of XML attributes in the `tools` namespace that enable design-time features (such as which layout to show in a fragment) or compile-time behaviors (such as which shrinking mode to apply to your XML resources). When you build your app, the build tools remove these attributes so there is no effect on your APK size or runtime behavior.

```xml
<RootTag xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools" >
```

**Example:**

```xml
tools:src="@drawable/dice_1"
```

***

### Gradle

Gradle is an advanced build toolkit, to automate and manage the build process while allowing you to define flexible custom build configurations.

It controls:

- What devices run your app
- Compile to executable
- Dependency management
- App signing for Google Play
- Automated Tests

#### build.gradle

- Plugins repositories:
  
```groovy
buildscript {                 
  repositories {
      google()
      jcenter()
  }
```

- Android app configuration:
  
```groovy
android {
  compileSdkVersion 26
  defaultConfig {
      applicationId "org.gradle.helloworldgradle"
      minSdkVersion 19
      targetSdkVersion 26
      versionCode 1
      versionName "1.0"
      testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
  }
  buildTypes {
      release {
          minifyEnabled false
          proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
      }
  }
}
```

- External Dependencies:
  
```groovy
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'com.android.support:appcompat-v7:26.1.0'
    androidTestImplementation 'com.android.support.test:runner:1.0.1'
}
```

***

### Vector Drawables

A [VectorDrawable](https://developer.android.com/reference/android/graphics/drawable/VectorDrawable.html) is a vector graphic defined in an XML file as a set of points, lines, and curves along with its associated color information. The major advantage of using a vector drawable is image scalability. It can be scaled without loss of display quality, which means the same file is resized for different screen densities without loss of image quality.

**Vector drawables backward compatibility solution**:

To support vector drawable and animated vector drawable on devices running platform versions lower than Android 5.0 (API level 21) Use the following code snippet to configure the `vectorDrawables` element:

```groovy
android {
    defaultConfig {
        vectorDrawables.useSupportLibrary = true
    }
}
```

Then you can use the new `app:srcCompat` attribute to reference vector drawables as well as any other drawable available to `android:src`:

```xml
<ImageView
  android:layout_width="wrap_content"
  android:layout_height="wrap_content"
  app:srcCompat="@drawable/ic_add" />
```

You’ll also need to add the namespace to the root of the layout

```xml
xmlns:app="http://schemas.android.com/apk/res-auto"
```

***

### Summary

* To install Android Studio, go to [Android Studio](https://developer.android.com/sdk/index.html) and follow the instructions to download and install it.
* To see an app's Android hierarchy in the Project pane, click the **Project** tab in the vertical tab column. Then select **Android** in the drop-down menu at the top.
* When you need to add new dependencies to your project or change dependency versions, edit the `build.gradle(Module:app)` file.
* All code and resources for an app are located within the `app` and `res` folders. The `java` folder includes activities, tests, and other components in Kotlin or Java source code (or both). The `res` folder holds resources, such as layouts, strings, and images.
* To add features, components, and permissions to your Android app, edit the `AndroidManifest.xml` file. All app components, such as additional activities, must be declared in this XML file.
* To create an Android virtual device (an emulator) to run your app, use the [AVD Manager](https://developer.android.com/tools/devices/managing-avds.html).
* To run your app on a physical Android device using Android Studio, enable USB debugging on the device. To do this, open **Settings > About phone** and tap **Build number** seven times. Then open **Settings** \> **Developer options** and select **USB debugging**.

**Activities**

* `MainActivity` is a subclass of `AppCompatActivity`, which in turn is a subclass of `Activity`. An `Activity` is a core Android class that is responsible for drawing an Android app UI and receiving input events.
* All activities have an associated layout file, which is an XML file in the app's resources. The layout file is named for the activity, for example `activity_main.xml`.
* The `setContentView()` method in `MainActivity` associates the layout with the activity, and inflates that layout when the activity is created.
* Layout inflation is a process where the views defined in the XML layout files are turned into (or "inflated" into) Kotlin view objects in memory. Once layout inflation happens, the `Activity` can draw these objects to the screen and dynamically modify them.

**Views**

* All UI elements in the app layout are subclasses of the [`View`](http://developer.android.com/reference/android/view/View.html) class and are called _views_. `TextView` and `Button` are examples of views.
* `View` elements can be grouped inside a [`ViewGroup`](https://developer.android.com/reference/android/view/ViewGroup.html). A view group acts as a container for the views, or other view groups, within it. `LinearLayout` is an example of a view group that arranges its views linearly.

**View attributes**

* The `android:layout_width` and `android:layout_height` attributes indicate the width and height of a view. The `match_parent` value stretches the view to its parent's width or height. The `wrap_content` value shrinks the view to fit the view's contents.
* The `android:text` attribute indicates the text that a view should display (if that view displays text.) For buttons, `android:text` is the button label.
* The `android:orientation` attribute in a `LinearLayout` view group arranges the view elements it contains. A value of `horizontal` arranges views left to right. A value of `vertical` arranges the views top to bottom.
* The `android:layout_gravity` attribute determines the placement of a view and all that view's children.
* The `android:textSize` attribute defines the size of the text in a text view. Text sizes are specified in sp units (_scalable pixels_). By using sp units, you can size text independently of the device's display quality.

**Strings**

* Instead of hardcoding strings in the layout, it's a best practice to use string resources.
* String resources are contained in the `res/values/string.xml` file.
* To extract strings, use `Alt+Enter` (`Option+Enter` on a Mac). Select **Extract string resources** from the popup menu.

**Using views**

* To connect your Kotlin code to a view that you defined in the layout, you need to get a reference to the view object after the view has been inflated. Assign an ID (`android:id`) to the view in the layout, then use the [`findViewById()`](https://developer.android.com/reference/android/view/View#findViewById%28int%29) method to get the associated view object.
* When you create an ID for a view in the XML layout file, Android Studio creates an integer constant with that ID's name in the generated `R` class. You can then use that `R.id` reference in the `findViewById()` method.
* You can set the attributes of a view object in your Kotlin code directly by property name. For example, the text in a text view is defined by the `android:text` attribute in the XML, and it is defined by the `text` property in Kotlin.
* A _click handler_ is a method that is invoked when the user clicks or taps on a UI element. To attach a click-handler method to a view such as a button, use the `setOnClickListener()` method.

**Using toasts**

A [toast](https://developer.android.com/reference/android/widget/Toast.html) is a view that shows the user a simple message in a small popup window.

To create a toast, call the [`makeText()`](https://developer.android.com/reference/android/widget/Toast.html#makeText%28android.content.Context,%20int,%20int%29) factory method on the [`Toast`](https://developer.android.com/reference/android/widget/Toast.html) class with three arguments:

* The [context](https://developer.android.com/reference/android/content/Context.html) of the app `Activity`
* The message to display, for example a string resource
* A duration, for example [`Toast.LENGTH_SHORT`](https://developer.android.com/reference/android/widget/Toast.html#LENGTH_SHORT)

To display the toast, call `show()`.

**App resources:**

* Your app's resources can include images and icons, standard colors used in the app, strings, and XML layouts. All of those resources are stored in the `res` folder.
* The `drawable` resources folder is where you should put all the image resources for your app.

**Using vector drawables in image views:**

* Vector drawables are images described in XML format. Vector drawables are more flexible than bitmap images (such as PNG files) because they can be scaled to any size or resolution.
* To add a drawable to your app's layout, use an `<ImageView>` element. The source of the image is in the `android:src` attribute. To refer to the drawable resource folder, use `@drawable`, for example `"@drawable/image_name"`.
* Use the `ImageView` view in your `MainActivity` code for the image. You can use `setImageResource()` to change the view's image to a different resource. Use `R.drawable` to refer to specific drawables, for example `setImageResource(R.drawable.image_name)`.

**The `lateinit` keyword:**

* Minimize the calls to `findViewById()` in your code by declaring fields to hold those views, and initializing the fields in `onCreate()`. Use the `lateinit` keyword for the field to avoid needing to declare it nullable.

**The `tools` namespace for design-time attributes:**

* Use the `tools:src` attribute in the `<ImageView>` element in your layout to display an image in only Android Studio's preview or design editor. You can then use an empty image for `android:src` for the final app.
* Use the `tools` namespace in the Android layout file to create placeholder content or hints for layout in Android Studio. Data declared by `tools` attributes is not used in the final app.

**API levels:**

* Each Android OS has an official version number and name (for example Android 9.0, "Pie") and an API level (API 28). Use the API levels in your app's Gradle files to indicate the versions of Android your app supports.
* The `compileSdkVersion` parameter in the `build.gradle` file specifies the Android API level that Gradle should use to compile your app.
* The `targetSdkVersion` parameter specifies the most recent API level that you have tested your app against. In many cases this parameter has the same value as `compileSdkVersion`.
* The `minSdkVersion` parameter specifies the oldest API level your app can run on.

**Android Jetpack:**

* Android Jetpack is a collection of libraries, developed by Google, that offers backward-compatible classes and helpful functions for supporting older versions of Android. Jetpack replaces and expands on the set of libraries formerly known as the Android Support Library.
* Classes imported from the `androidx` package refer to the Jetpack libraries. Dependencies to Jetpack in your `build.gradle` file also start with `androidx`.

**Backward compatibility for vector drawables:**

* Vector drawables are only natively supported in versions of Android higher than API 21. In older versions, Gradle generates PNG images for those drawables when your app is built.
* You can specify that the Android Support Library should be used for vector drawables in older API versions with the `vectorDrawables.useSupportLibrary = true` configuration parameter in the `build.gradle` file.
* Once you've enabled the support library for vector drawables, use the `app:srcCompat` attribute in the `<ImageView>` element (instead of `android:src`) to specify the vector drawable source for that image.

**The `app` namespace:**

* The `app` namespace in your XML layout file is for attributes that come from either your custom code or from libraries, not from the core Android framework.

#### Learn to help yourself

* Official Android developer documentation is at [developer.android.com](http://developer.android.com/index.html).
* _Material Design_ is a conceptual design philosophy that outlines how apps should look and function on mobile devices. Material Design isn't just for Android apps. The Material Design guidelines are at [material.io](https://material.io/).
* Android Studio provides templates for common and recommended app and activity designs. These templates offer working code for common use cases.
* When you create a project, you can choose a template for your first activity.
* While you are developing your app, you can create activities and other app components from built-in templates.
* [Google Samples](https://github.com/googlesamples) contains code samples that you can study, copy, and incorporate into your projects.