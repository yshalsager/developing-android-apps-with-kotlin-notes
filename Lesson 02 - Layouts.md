## Lesson 02: Layouts

> Designing your app's UI is the first step to a great user experience.
> This lesson covers all the basics of UI layout design. You'll use all the popular view types with a focus on the ContraintLayout to make two apps, "[About Me](https://github.com/udacity/andfun-kotlin-about-me)" and "[Color my Views](https://github.com/udacity/andfun-kotlin-color-my-views)".

### View Groups & View Hierarchy

A [ViewGroup](https://developer.android.com/reference/kotlin/android/view/ViewGroup.html) is a special view that can contain other views (called children.) The view group is the base class for layouts and views containers.

- In Android, all the visual elements that make up a screen are views and they are all children of the view class.

- They share a lot of properties, for example, all views have a width and a height and
  a background.

- **Examples:**
  
  - [TextView](https://developer.android.com/reference/kotlin/android/widget/TextView)
  - [ImageView](https://developer.android.com/reference/kotlin/android/widget/ImageView)
  - [Button](https://developer.android.com/reference/kotlin/android/widget/Button)
  - [EditText](https://developer.android.com/reference/kotlin/android/widget/EditText.html)
  - [CheckBox](https://developer.android.com/reference/kotlin/android/widget/CheckBox.html)
  - Menus
  - Color Pickers

**Density Independent Pixel (DP)**: An abstract unit for expressing location that is based on the physical pixel density of the screen.

Android devices will automatically handle the conversion from
DP to pixel values

Views that make up a layout are organized as a hierarchy of views.

- Views whose primary job it is to contain other views are called view groups.
- Commonly a layout has a top-level view group and any number of views and view groups inside it.
- Linear layout is a view group where you can arrange views horizontally or vertically.
- [ScrollView](https://developer.android.com/reference/kotlin/android/widget/ScrollView.html) is a view group that allows the view hierarchy placed within it to be scrolled.
- ConstraintLayout is a ViewGroup where elements can be arranged freely, are placed by the system based on constraints, and may adapt in size based on screen size and orientation.

***

### [TextView](https://developer.android.com/reference/kotlin/android/widget/TextView)

A user interface element that displays text to the user.

```xml
<TextView
    android:id="@+id/text_view_id"
    android:layout_height="wrap_content"
    android:layout_width="wrap_content"
    android:text="@string/hello" />
```

**Some common TextView XML attributes**:

- `android:textSize`: Size of the text. Recommended dimension type for text is "sp" for scaled-pixels (example: 15sp).
- `android:textAlignment`: Defines the alignment of the text.
- `android:textColor`: Defines the text color.
- `android:fontFamily`: Defines Font family (named by string or as a font resource reference) for the text.
- `android:lineSpacingMultiplier`: Adds some spacing between the lines.
- `android:drawableTop`: Puts an Image before text, instead of having a separate ImageView for it.

#### Controling the visibility of TextView:

`android:visibility`: Controls the initial visibility of the view. Method [setVisibility(int)](https://developer.android.com/reference/kotlin/android/view/View.html#setVisibility(kotlin.Int)) can be also used. 

- **gone**: Completely hidden, as if the view had not been added. **(2)**
- **invisible**: Not displayed, but taken into account during layout (space is left for it). **(1)**
- **visible**: Visible on screen; the default value. **(0)**

```kotlin
val editText = findViewById<EditText>(R.id.nickname_edit)
val nicknameTextView = findViewById<TextView>(R.id.nickname_text)
editText.visibility = View.GONE
nicknameTextView.visibility = View.VISIBLE
```

###### Best Practice

- Store the text size, padding, and margin values in `dimens.xml` file for better and cleaner structure, instead of repeating.
  
```xml
<resources>
    <dimen name="text_size">20sp</dimen>
    <dimen name="small_padding">8dp</dimen>
    <dimen name="layout_margin">16dp</dimen>
    <dimen name="padding">16dp</dimen>
</resources>
```

- Store text styling attributes in `styles.xml` to make the elements of a layout look consistent across the app. Then you can use it for a TextView `style="@style/NameStyle"`
  
```xml
<style name="NameStyle">
    <item name="android:layout_marginTop">@dimen/layout_margin</item>
    <item name="android:fontFamily">@font/roboto</item>
    <item name="android:paddingTop">@dimen/small_padding</item>
    <item name="android:textColor">@android:color/black</item>
    <item name="android:textSize">@dimen/text_size</item>
</style>    
```

#### Padding and Margin

- Padding is a space added inside/within components like `TextView` , `Button`, `EditText`, etc. Eg. space between the Text and Border.
  
```xml
android:padding="10dp"
android:paddingTop="@dimen/small_padding
android:paddingRight="40dp"
```

- [Margin](https://developer.android.com/reference/android/view/ViewGroup.MarginLayoutParams) is the space added outside the boundaries of an element. Eg. space between the left edge of the screen and the border of your component.
  
```xml
android:layout_margin="10dp"
android:layout_marginTop
android:layout_marginStart
```

*Note:* **ImageView contentDescription warning**:

`contentDescription` is a property of `ImageView` used by screen readers to describe an image to the user. It should be added to `ImageView` to fix the warning.

```xml
android:contentDescription="@string/my_image_description"
```

***

### [ScrollView](https://developer.android.com/reference/kotlin/android/widget/ScrollView.html)

- A view group that allows the view hierarchy placed within it to be scrolled.
- Scroll view may have only one direct child placed within it.
- To add multiple views within the scroll view, make the direct child you add a view group, for example, `LinearLayout`, and place additional views within that LinearLayout.
- Scroll view supports vertical scrolling only. For horizontal scrolling, use `HorizontalScrollView` instead.
- Never add a `RecyclerView` or `ListView ` to a scroll view. Doing so results in poor user interface performance and poor user experience.

```xml
<ScrollView
   android:layout_width="match_parent"
   android:layout_height="match_parent">

   <LinearLayout
       android:layout_width="match_parent"
       android:layout_height="wrap_content"
       android:orientation="vertical">

       <ImageView
           android:layout_width="match_parent"
           android:layout_height="wrap_content"
           app:srcCompat="@android:drawable/ic_menu_view" />

       <TextView
           android:layout_width="match_parent"
           android:layout_height="wrap_content"
           android:text="@string/some_long_text"/>

   </LinearLayout>

</ScrollView>
```

***

### [EditText](https://developer.android.com/reference/kotlin/android/widget/EditText.html)

- A user interface element for entering and modifying text.
- When you define an edit text widget, you must specify `inputType` attribute. 

```xml
<EditText
    android:id="@+id/nickname_edit"
    style="@style/NameStyle"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:hint="@string/what_s_your_name"
    android:importantForAutofill="no"
    android:inputType="textPersonName" />
```

##### Hiding the keyboard after input is complete automatically:

```kotlin
// Hide the keyboard.
val imm = getSystemService(Context.INPUT_METHOD_SERVICE) as InputMethodManager
imm.hideSoftInputFromWindow(view.windowToken, 0)
```

***

### Data Binding

- Using find view by ID to get a reference to views is okay, but every time we search for a view in this way after it has been created or recreated Android has to traverse the view hierarchy to find it for us at runtime.
- For a large or deep view hierarchy, this can take enough time that it can slow down the app for the user.
- To fix this there is a technique and pattern called data binding that allows us to connect a layout to an activity or fragment at compile time.
- The compiler generates a helper class called a binding class when the activity is created so is an instance of that binding class and then we can access the view through this generated binding class without any extra overhead.

#### Enabling [Data Binding](https://developer.android.com/topic/libraries/data-binding/start)

1. In `app/build.gradle` add the following to android block and sync the project:

```groovy
buildFeatures {
    dataBinding true
}

// dataBinding {
//        enabled = true
//    }
```

2. Wrap XML layout into `<layout></layout>` tag and move namespaces to it.

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
    </LinearLayout>
</layout> 
```

3. In MainActivity:
   
   - Create a Binding object (Name of the object is derived from the name of the activity or fragment).
     
```kotlin
    import com.example.android.aboutme.databinding.ActivityMainBinding
    private lateinit var binding: ActivityMainBinding
```
   
   - Setting the content view using DataBindingUtil creates an instance of ActivityMainBinding from the supplied activity and the supplied layout. This object contains mappings between the activity and layout, and functionality to interact with them.
     
```kotlin
     import androidx.databinding.DataBindingUtil
     binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
```

#### Accessing Elements using Data Binding

```kotlin
//        findViewById<Button>(R.id.done_button).setOnClickListener {
//            addNickname(it)
//        }
        // Click listener for the Done button.
        binding.doneButton.setOnClickListener {
            addNickname(it)
        }
```

```kotlin
    private fun addNickname(button: View) {
//        val editText = findViewById<EditText>(R.id.nickname_edit)
//        val nicknameTextView = findViewById<TextView>(R.id.nickname_text)
//
//        nicknameTextView.text = editText.text
//        editText.visibility = View.GONE
//        button.visibility = View.GONE
//        nicknameTextView.visibility = View.VISIBLE
        binding.apply {
            // Invalidate all binding expressions and request a new rebind to refresh UI
            invalidateAll()
            nicknameEdit.visibility = View.GONE
            doneButton.visibility = View.GONE
            nicknameText.visibility = View.VISIBLE
        }
    }
```

#### Binding Data

This makes data available to the view directly.

- Define a new data class:
  
```kotlin
  data class MyName(var name: String = "", var nickname: String = "")
```

- In XML layout, create a data block inside `<layout>` at the top, and declare variables with `name` as the same as data class and a `type` which is the full name of data class, inside of that block.
  
```xml
<!-- Use a data block to declare variables. -->
<data>
    <!-- Declare a variable by specifying a name and a data type. -->
    <!-- Use fully qualified name for the type. -->
    <variable
      name="myName"
      type="com.example.android.aboutme.MyName" />
</data>
```

- Now you can reference the new variables from XML directly after creating them.
  
```xml
<TextView
    android:id="@+id/name_text"
    style="@style/NameStyle"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="@={myName.name}"
    android:textAlignment="center" />
    <!--android:text="@string/yshalsager"-->
```

- In MainActivity create an instance of data class and set the name
  
```kotlin
// Instance of MyName data class.
private val myName: MyName = MyName("yshalsager")
```

- Then, in onCreate set the value of myName variable that is declared and used in the layout file.
  
```kotlin
// Set the value of the myName variable that is declared and used in the layout file.
binding.myName = myName
```

- And finally set the nickname like we just did in XML and MainActivity.
  
```xml
<TextView
    android:id="@+id/nickname_text"
    style="@style/NameStyle"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="@={myName.nickname}"
    android:textAlignment="center"
    android:visibility="gone" />
```
  
```kotlin
binding.apply {
  // Set the text for nicknameText to the value in nicknameEdit.
  myName?.nickname = nicknameEdit.text.toString()
```

##### Extra: Localizing bounded-data
- You can do this:
`android:text= "@{String.format(@string/Generic_Text, Profile.name)}"`. If you use string formatting for your `Generic_Text` string. ex. `%s` at the end

- Or even simpler:
`android:text= "@{@string/generic_text(profile.name)}"`so, you string should be like this: `<string name="generic_text">My Name is %s</string>`.
- you can use as many variables as you need:
```xml
android:text= "@{@string/generic_text(profile.firstName, profile.secondName)}"
```
Which equals: `<string name="generic_text">My Name is %1$s %2$s</string>`.

[Source](https://stackoverflow.com/a/38984088) - [Read more](https://developer.android.com/topic/libraries/data-binding/expressions#resources)

***

### Constraint Layout

A ConstraintLayout is a ViewGroup that allows you to position and size widgets in a flexible way.

All the power of `ConstraintLayout` is available directly from the Layout Editor's visual tools because the layout API and the Layout Editor were specially built for each other. So you can build your layout with `ConstraintLayout` entirely by drag-and-dropping instead of editing the XML.

#### Advantages of Constraint Layout:

- You can make it responsive to screens and resolutions
- Usually, flatter view hierarchy
- Optimized for laying out its views
- Free-form place views anywhere and the editor helps add constraints

#### Constraint

A connection or an alignment to another UI element, or to the parent layout, or to an invisible guideline.

When creating constraints, remember the following rules:

- Every view must have at least two constraints: one horizontal and one vertical.
- You can create constraints only between a constraint handle and an anchor point that share the same plane. So a vertical plane (the left and right sides) of a view can be constrained only to another vertical plane, and baselines can constrain only to other baselines.
- Each constraint handle can be used for just one constraint, but you can create multiple constraints (from different views) to the same anchor point.

#### Types of constraints

1. **Fixed constraints**: specified using a hard-coded number, represented by straight lines, and used commonly in margins.
2. **Adaptable constraint**: A constraint that defines a relationship in relative and weighted terms.

#### Positioning types

- Absolute positioning: Positioning is numerical, such as the position in x, y coordinates.
- Relative positioning: Views are positioned by specifying the relationship to other views.

#### Ratio

Ratio constraints are useful when:

- A view needs to have a certain aspect ratio to accommodate an image, no matter what the screen orientation or display size is.
- Or you want to create a layout with squares that is adaptable but forces the squares to remain square.

#### Types of chaining

- **Packed chain**: Elements are packed to use minimum space.
- **Packed chain with bias**: Elements are packed to use minimum space and are moved on their axis depending on bias.
- **Spread chain**: Elements are spread equally in space.
- **Spread inside chain**: Elements are spread to use available space with head and tail attached to the parent.
- **Weighted chain**: Elements are resized to use all available space according to specified weights with head and tail glued to parent.

**Note:** Changing background using `setBackgroundColor()` can be set with:

- Color class colors
  
```kotlin
  view.setBackgroundColor(Color.GRAY)
```

- Android color resources
  
```kotlin
  view.setBackgroundResource(android.R.color.holo_green_light)
```

- Custom defined colors in colors.xml
  
```xml
  <color name="my_green">#12C700</color>
```
  
```kotlin
  box_three_text.setBackgroundResource(R.color.my_red)
```

#### Baseline Constraint

Used to align the text baseline of a view to the text baseline of another view for example.

To create a baseline constraint, right-click the text view you want to constrain and then click **Show Baseline**. Then click on the text baseline and drag the line to another baseline.

***

### Summary

#### LinearLayout using the Layout Editor

* A `ViewGroup` is a view that can contain other views. [`LinearLayout`](https://developer.android.com/reference/android/widget/LinearLayout) and [`ScrollView`](https://developer.android.com/reference/android/widget/ScrollView) are view groups.
* `LinearLayout` is a view group that arranges its child views horizontally or vertically.
* Use a `ScrollView` when you need to display content on the screen, such as long text or a collection of images. A scroll view can contain only one child view. If you want to scroll more than one view, then add a `ViewGroup` such as a `LinearLayout` to the `ScrollView`, and put the views to be scrolled inside that `ViewGroup`.
* The [Layout Editor](https://developer.android.com/studio/write/layout-editor) is a visual design editor inside Android Studio. You can use the Layout Editor to build your app's layout by dragging UI elements into the layout.
* A [style](https://developer.android.com/guide/topics/ui/look-and-feel/themes) is a collection of attributes that specify the appearance for a view. For example, a style can specify font color, font size, background color, padding, and margin.
* You can extract and collect all the formatting of a view into a style. To give your app a consistent look, reuse the style for other views.

#### Add user interactivity

* The [Layout Editor](https://developer.android.com/studio/write/layout-editor) tool in Android Studio is a visual design editor. You can use the Layout Editor to build your app's layout by dragging UI elements into your layout.
* [`EditText`](https://developer.android.com/reference/android/widget/EditText) is a UI element that lets the user enter and modify text.
* A [`Button`](https://developer.android.com/reference/android/widget/Button) is a UI element that the user can tap to perform an action. A button can consist of text, an icon, or both text and an icon.

**Click listeners**

* You can make any `View` respond to being tapped by adding a click listener to it.
* The function that defines the click listener receives the `View` that is clicked.

You can attach a click-listener function to a `View` in either of two ways:

* In the XML layout, add the [`android:onClick`](https://developer.android.com/reference/android/R.attr.html#onClick) attribute to the `<`_`View`_`>` element.
* Programmatically, use the [`setOnClickListener(View.OnClickListener)`](https://developer.android.com/reference/android/view/View.html#setOnClickListener%28android.view.View.OnClickListener%29) function in the corresponding `Activity`.

#### ConstraintLayout using the Layout Editor

* A [`ConstraintLayout`](https://developer.android.com/reference/android/support/constraint/ConstraintLayout.html) is a [`ViewGroup`](http://developer.android.com/reference/android/view/ViewGroup.html) that allows you to position and size the layout's child views in a flexible way.
* In a ConstraintLayout, each view's position is defined using at least one horizontal constraint, and at least one vertical constraint*.*
* A [constraint](https://developer.android.com/training/constraint-layout/#constraints-overview) connects or aligns a view to another UI element, to the parent layout, or to an invisible guideline.

**Advantages of using `ConstraintLayout`:**

* You can make a ConstraintLayout responsive to devices that have different screen sizes and resolutions.
* `ConstraintLayout` usually results in a flatter view hierarchy than `LinearLayout`.
* The design editor and the view inspector in Android Studio help you add and configure constraints.

**Chains:**

* A [chain](https://developer.android.com/training/constraint-layout/#constrain-chain) is a group of views that are linked to each other with bidirectional constraints.
* The views within a chain can be distributed either vertically or horizontally.

**Design-time attributes:**

* Design-time attributes are used and applied only during the layout design, not at runtime. When you run the app, design-time attributes are ignored.
* Design-time attributes are prefixed with the `tools` namespace. For example, the `tools:layout_editor_absoluteY` and `tools:text` attributes are design-time attributes.

**Baseline constraints:**

* A baseline constraint aligns a view's text baseline to the text baseline of another view that has text.
* Baseline constraints are helpful when views have different font sizes.

#### Data binding basics

Steps to use data binding to replace calls to `findViewById()`:

1. Enable data binding in the android section of the `build.gradle` file:

```kotlin
buildFeatures {
    dataBinding true
}
```

2. Use `<layout>` as the root view in your XML layout.
3. Define a binding variable: `private lateinit var binding: ActivityMainBinding`
4. Create a binding object in `MainActivity`, replacing `setContentView`: `binding = DataBindingUtil.setContentView(this, R.layout.activity_main)`
5. Replace calls to `findViewById()` with references to the view in the binding object. For example: `findViewById<Button>(R.id.done_button)` ⇒ `binding.doneButton` (In the example, the name of the view is generated camel case from the view's `id` in the XML.)

**Steps for binding views to data:**

1. Create a data class for your data.
2. Add a `<data>` block inside the `<layout>` tag.
3. Define a `<variable>` with a name, and a type that is the data class.

```xml
<data>
   <variable
       name="myName"
       type="com.example.android.aboutme.MyName" />
</data>
```
4. In `MainActivity`, create a variable with an instance of the data class. For example: `private val myName: MyName = MyName("Aleks Haecky")`
5. In the binding object, set the variable to the variable you just created: `binding.myName = myName`
6. In the XML, set the content of the view to the variable that you defined in the `<data>` block. Use dot notation to access the data inside the data class. `android:text="@={myName.name}"`

