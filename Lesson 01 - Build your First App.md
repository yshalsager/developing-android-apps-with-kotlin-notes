# Android with Kotlin Course Notes

## Lesson 01: Build your First App

> Build your first app: "Dice Roller" that covers basic Android components like displaying texts and images as well as a tour of the Android tools you'll be using throughout this course.

##### Android project contains:

- Kotlin files for the core logic of the app

- A resources folder for static content such as images and strings

- An Android Manifest file that defines essential app details so the OS can launch your app

- Gradle scripts, for building and running your app

### [AndroidManifest.xml](https://developer.android.com/guide/topics/manifest/manifest-intro)

- ```xml
  package="com.example.diceroller"
  ```

The name in the manifest's `package` attribute should always match your project's base package name where you keep your activities and other app code.

The `package` attribute also represents your app's universally unique application ID.

- ```xml
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

### [Activity](https://developer.android.com/guide/topics/manifest/activity-element) And [Layout](https://developer.android.com/reference/kotlin/android/text/Layout.html)

- Main activity is an example of an activity.

- An activity is a core Android class that is responsible for drawing an app user interface and receiving input events.

- When your app launches it launches a specific activity this is the activity that was declared in the manifest with the correct intent-filter tag.

- Activities have an Associated layout file which in this case is activity underscore main.

- Layout files are XML files that express what the app actually looks like they do this by defining things like text images and buttons and where these things will appear on the screen. these text images and buttons are called views.

- The activity and the layout are connected by a process known as layout inflation this process is triggered when the activity starts.

##### [setContentView](https://developer.android.com/reference/kotlin/android/app/Activity.html)

Set the activity content from a layout resource. The resource will be inflated, adding all top-level views to the activity.

```kotlin
setContentView(R.layout.activity_main)
```

##### [Linear Layout](https://developer.android.com/reference/kotlin/android/widget/LinearLayout)

`LinearLayout` is a view group that aligns all children in a single direction, vertically or horizontally. You can specify the layout direction with the `android:orientation` attribute.

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >
    ...
</LinearLayout>
```

- ```xml
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

### [OnClickListener](https://developer.android.com/reference/kotlin/android/view/View.OnClickListener.html)

Interface definition for a callback to be invoked when a view is clicked.

- setting a button OnClickListener
  
  ```kotlin
  rollButton.setOnClickListener {
      ....
  }
  ```

### [Toast](https://developer.android.com/reference/kotlin/android/widget/Toast.html)

A toast is a view containing a quick little message for the user.

- Showing a Toast message
  
  ```kotlin
  Toast.makeText(this, "button clicked", Toast.LENGTH_SHORT).show()
  ```

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

### Gradle

Gradle is an advanced build toolkit, to automate and manage the build process while allowing you to define flexible custom build configurations.

It controls:

- What devices run your app

- Compile to executable

- Dependency management

- App signing for Google Play

- Automated Tests

##### build.gradle

- Plugins repositories:
  
  ```groovy
  buildscript {                 
      repositories {
          google()
          jcenter()
      }
  ```

- Android app config:
  
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
