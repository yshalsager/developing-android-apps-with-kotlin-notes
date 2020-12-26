## Lesson 10: Designing for Everyone

> Coming up with a good design for an app is always hard, but creating a great design is more than just colors and layouts. Let's build an app that is designed for everyone, everywhere!

### Styling on Android

Android provides a rich styling system that lets you control the appearance of all the views in your app. You can use themes, styles, and view attributes to affect styling. The diagram shown below summarizes the precedence of each method of styling. The pyramid diagram shows the order in which styling methods are applied by the system, from the bottom up. For example, if you set the text size in the theme, and then set the text size differently in the view attributes, the view attributes will override the theme styling.

![](https://developer.android.com/codelabs/kotlin-android-training-styles-and-themes/img/e7851427054b568d.png)

##### View attributes

* Use view attributes to set attributes explicitly for each view. (View attributes are not reusable, as styles are.)
* You can use every property that can be set via styles or themes.
* Use for custom or one-off designs such as margins, paddings, or constraints.

##### Styles

* Use a style to create a collection of reusable styling information, such as font size or colors.
* Good for declaring small sets of common designs used throughout your app.
* Apply a style to several views, overriding the default style. For example, use a style to make consistently styled headers or a set of buttons.

##### Default style

* This is the default styling provided by the Android system.

##### Themes

* Use a theme to define colors for your whole app.
* Use a theme to set the default font for the whole app.
* Apply to all views, such as text views or radio buttons.
* Use to configure properties that you can apply consistently for the whole app.

##### TextAppearance

* For styling with text attributes only, such as `fontFamily`.

When Android styles a view, it applies a combination of themes, styles, and attributes, which you can customize. Attributes always overwrite anything specified in a style or theme. And styles always overwrite anything specified in a theme.

The screenshots below show the [GDG-finder app](https://github.com/udacity/andfun-kotlin-gdg-finder) with light theme (left) and a dark theme (right), as well as with a custom font and header sizes. This can be implemented in several ways, and you learn some of them in this lesson.

 ![](https://developer.android.com/codelabs/kotlin-android-training-styles-and-themes/img/eab9826ee090267a.png)  ![](https://developer.android.com/codelabs/kotlin-android-training-styles-and-themes/img/80cc74df29081180.png)
 
 ***

### Design Through the Looking Glass

The focus in this lesson is on one of the most important aspects of Android development, app design. Obvious aspects of app design are views, text, and buttons, and where they are on the screen, as well as the colors and fonts they use. Hints to the user about what to do next are also an essential aspect of design. Users need to be able to tell at a glance what they are looking at, what is important, and what they can do.

Compare the two screens below. Notice that by moving elements around and drawing focus to what's important, you can help the user understand what's going on. For simple screens, good design often means showing less. For screens with lots of important information, good design makes dense information understandable at a glance. As you work on Android Apps you may hear this concept called [information architecture](https://en.wikipedia.org/wiki/Information_architecture) (IA).

 ![](https://developer.android.com/codelabs/kotlin-android-training-styles-and-themes/img/50ba139460366f18.png)   ![](https://developer.android.com/codelabs/kotlin-android-training-styles-and-themes/img/f2bb268d07be0e05.png) 

Another level of design is building coherent user flows, or _use cases_, that let users accomplish tasks. This form of design is called [user experience design](https://en.wikipedia.org/wiki/User_experience_design) (UXD), and some designers specialize in it.

If you don't have access to a designer, here are a few tips for getting started:

* Define use cases. Write up what users should accomplish with your app, and how.
* Implement a design. Don't be attached to your first draft, and only make it "good enough," because you'll change it after you see how real users interact with it.
* Get feedback. Find anyone who you can talk into testing your app—your family, friends, or even people you just met at a Google Developer Group. Ask them to use your app to perform a use case while you watch and take detailed notes.
* Refine! With all that information, make the app better, and then test it again.

Here are some other questions to ask yourself when designing a great app experience. You learned techniques for tackling these issues in preceding tasks:

* Does the app lose its state when the user rotates their device?
* What happens when the user opens the app? Does the user see a loading spinner, or is the data ready in an offline cache?
* Is the app coded in a way that is efficient and ensures that the app is always responsive to user touch?
* Does the app interact with backend systems in a way that never leads to odd, incorrect, or stale data being presented to the user?

As you work on apps for larger audiences, it's essential to make your apps accessible to as many types of users as possible. For example:

* Many users interact with computer systems in different ways. Many users are color blind, and colors that contrast for one user may not work for another. Many users have vision impairments, from needing reading glasses all the way through blindness.
* Some users can't use touch screens, and they interact via other input devices such as switches.

Good design is the most important way to get users using your app.

***

### Adding [TextView Attributes](https://developer.android.com/reference/android/widget/TextView#xml-attributes_1)

1. Review the original layout. Open up `home_fragment.xml` and have a look at the layout. It’s just a `ConstraintLayout` with a few images and some unstyled `TextViews`.  
  
2. Change the size of the title and subtitle.

In `home_fragment.xml`, add `textSize` attributes to the `TextViews`. The subtitle should be smaller than the title.
```xml
     <TextView
           android:id="@+id/title"
           android:textSize="24sp" />
    
     <TextView
           android:id="@+id/subtitle"
           android:textSize="18sp"/>
```

3. Add a `textColor` attribute to change the text color to dark gray. Here’s how the new attributes should look in your `TextView`s:

```xml
     <TextView
           android:id="@+id/title"
           android:textColor="#555555"
           android:textSize="24sp" />
     <TextView
           android:id="@+id/subtitle"
           android:textColor="#555555"
           android:textSize="18sp"/>
```

Finally, run your code and make sure you see that the text is larger and the dark gray.

* As a reminder, `sp` stands for _scale-independent pixels_, which are scaled by both the pixel density and the font-size preference that the user sets in their device settings. Android figures out how large the text should be on the screen when it draws the text. Always use `sp` for text sizes.

**Tip:** An aRGB value expresses a color's alpha transparency, red value, green value, and blue value. An aRGB value uses a hexadecimal number ranging from 00 to FF for each color component.

`#(alpha)(red)(green)(blue)`

`#(00-FF)(00-FF)(00-FF)(00-FF)`

Examples:

* `#FFFF0000` for opaque red
* `#5500FF00` for semi-transparent green
* `#FF0000FF` for opaque blue

***

### Themes and [Fonts](https://developer.android.com/guide/topics/ui/look-and-feel/fonts-in-xml)

When using fonts with your app, you could ship the necessary font files as part of your APK. While simple, this solution is usually not recommended, because it makes your app take longer to download and install.

Android lets apps download fonts at runtime using the [Downloadable Fonts](https://developer.android.com/guide/topics/ui/look-and-feel/downloadable-fonts) API. If your app uses the same font as another app on the device, Android only downloads the font once, saving the device's storage space.

### Applying a Theme

#### Step 1: Apply a [downloadable font](https://developer.android.com/guide/topics/ui/look-and-feel/downloadable-fonts)

1. Open the `home_fragment.xml` in the **Design** tab.
2. In the **Component Tree** pane, select the `title` text view.
3. In the **Attributes** pane, find the `fontFamily` attribute. You can find it in the **All Attributes** section, or you can just search for it.
4. Click the drop-down arrow.
5. Scroll to **More Fonts** and select it. A **Resources** window opens.

![](https://developer.android.com/codelabs/kotlin-android-training-styles-and-themes/img/c86091d315d7785a.png)

6. In the **Resources** window, search for `lobster`, or just `lo`.
7. In the results, select **Lobster Two**.
8. To the right, below the font name, select the **Create downloadable font** radio button. Click **OK**.
9. Open the Android Manifest file.
10 Near the bottom of the manifest, find the new `<meta-data>` tag with the `name` and `resource` attributes set to `"preloaded_fonts"`. This tag tells Google Play Services that this app wants to use downloaded fonts. When your app runs and requests the Lobster Two font, the font provider downloads the font from the internet, if the font is not already available on the device.

```xml
<meta-data android:name="preloaded_fonts" android:resource="@array/preloaded_fonts"/>
```

11. In the `res/values` folder, find the `preloaded_fonts.xml` file, which defines the array that lists all the downloadable fonts for this app.
12. Likewise, the `res/fonts/lobster_two.xml` file has information about the font.
13. Open `home_fragment.xml` and notice in the code and preview that the Lobster Two font is applied to the `title` `TextView`, and thus applied to the title.

![](https://developer.android.com/codelabs/kotlin-android-training-styles-and-themes/img/7251480f6dd43fd7.png)

14. Open `res/values/styles.xml` and examine the default `AppTheme` theme that was created for the project. It currently looks as shown below. To apply the new Lobster Two font to all the text, you'll need to update this theme.  
    
15. In the `<style>` tag, notice the `parent` attribute. Every style tag can specify a parent and inherit all the parent's attributes. The code specifies the `Theme` defined by the Android libraries. The [`MaterialComponents`](https://material.io/develop/android/docs/theming-guide/) theme that specifies everything from how buttons work to how to draw toolbars. The theme has sensible defaults, so you can customize just the parts you want. The app uses the `Light` version of this theme without the action bar (`NoActionBar`), as you can see in the screenshot above.  

```xml
<!-- Base application theme. -->
<style name="AppTheme" parent="Theme.MaterialComponents.Light.NoActionBar">
   <!-- Customize your theme here. -->
   <item name="colorPrimary">@color/colorPrimary</item>
   <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
   <item name="colorAccent">@color/colorAccent</item>
</style>
```

16. Inside the `AppTheme` style, set the font family to `lobster_two`. You need to set both `android:fontFamily` and `fontFamily`, because the parent theme uses both. You can check `home_fragment.xml` in the **Design** tab to preview your changes.

```xml
<style name="AppTheme"  
...    
        <item name="android:fontFamily">@font/lobster_two</item>
        <item name="fontFamily">@font/lobster_two</item>
```

17. Run the app again. The new font is applied to all the text! Open the navigation drawer and move to the other screens, and you see that the font is applied there as well.

![](https://developer.android.com/codelabs/kotlin-android-training-styles-and-themes/img/b69bcf61b7a2615a.png)
![](https://developer.android.com/codelabs/kotlin-android-training-styles-and-themes/img/5561c237dece0f9c.png)
![](https://developer.android.com/codelabs/kotlin-android-training-styles-and-themes/img/7833662d04f68c0.png)

***

### Adding [Styles](https://developer.android.com/guide/topics/resources/style-resource) to Headers

Themes are great for applying general theming to your app, such as a default font and primary colors. Attributes are great for styling a specific view and adding layout information such as margins, padding, and constraints, which tend to be specific to each screen.

In the middle of the style-hierarchy pyramid are _styles_. Styles are reusable "groups" of attributes that you can apply to views of your choice. In this task, you use a style for the title and subtitle.

#### Step 1: Create a style

1. Open `res/values/styles.xml`.
2. Inside the `<resources>` tag, define a new style using the `<style>` tag, as shown below.

```xml
<style name="TextAppearance.Title" parent="TextAppearance.MaterialComponents.Headline6">
</style>
```

It's important to think about style names as semantic when you name them. Select a style name based on what the style will be used for, not based on the properties that the style affects. For example, call this style `Title`, not something like `LargeFontInGrey`. This style will be used by any title anywhere in your app. As a convention, `TextAppearance` styles are called `TextAppearance.`_`Name`_, so in this case, the name is `TextAppearance.Title`.

The style has a parent, just as a theme can have a parent. But this time, instead of extending a theme, the style extends a style, `TextAppearance.MaterialComponents.Headline6`. This style is a default text style for the `MaterialComponents` theme, so by extending it you modify the default style instead of starting from scratch.

3. Inside the new style, define two items. In one item, set the `textSize` to `24sp`. In the other item, set the `textColor` to the same dark gray used before.

```xml
<item name="android:textSize">24sp</item>
<item name="android:textColor">#555555</item>
```

4. Define another style for the subtitles. Name it `TextAppearance.Subtitle`.
5. Because the only difference from `TextAppearance.Title` will be in the text size, make this style a child of `TextAppearance.Title`.
6. Inside the `Subtitle` style, set the text size to `18sp`. Here's the completed style:

```xml
<style name="TextAppearance.Subtitle" parent="TextAppearance.Title" >
   <item name="android:textSize">18sp</item>
</style>
```

#### Step 2: Apply the style that you created

1. In `home_fragment.xml`, add the `TextAppearance`.`Title` style to the `title` text view. Delete the `textSize` and `textColor` attributes.  
      
    Themes override any `TextAppearance` styling that you set. (The pyramid diagram at the beginning of the codelab shows the order in which styling is applied.) Use the `textAppearance` property to apply the style as a `TextAppearance` so that the font set in the `Theme` overrides what you set here.

```xml
<TextView
       android:id="@+id/title"
       android:textAppearance="@style/TextAppearance.Title"
```

2. Also add the `TextAppearance.Subtitle` style to the `subtitle` text view, and delete the `textSize` and `textColor` attributes. You have to apply this style as a `textAppearance` also, so that the font set in the theme overrides what you set here.

```xml
<TextView
       android:id="@+id/subtitle"
       android:textAppearance="@style/TextAppearance.Subtitle"
```

**Important:** When you have both themes and styles manipulating text, you must apply the text properties as a `textAppearance` attribute if you want the text properties in the theme to override what's set and inherited in the style.

3. Run the app and your text is now consistently styled.

***

### [Material Design](https://developer.android.com/guide/topics/ui/look-and-feel)

Material Design is a cross-platform design system from Google, and is the design system for Android. Material Design provides detailed specs for everything in an app's user interface (UI), from how text should be shown, to how to lay out a screen. The Material Design website at [material.io](https://material.io/) has the complete specifications.

On material.io you can also find a list of additional views that ship with Material Design components. The views include [bottom navigation](https://material.io/develop/android/components/bottom-navigation-view/), a floating action button (FAB) that you use in this lesson, chips that you explore in the next tasks, and [collapsing toolbars](https://material.io/develop/android/components/collapsing-toolbar-layout/).

In addition to these views that you can use to implement Material Design, the Material Design components library exports the `MaterialComponents` theme that your app already uses. The `MaterialComponents` theme implements Material Design for controls, uses theme attributes, and is customizable.

To see examples of apps that use Material Design, open Gmail, or check out the [Google Design](https://design.google/library/) website, in particular the annual [Material Design awards](https://design.google/library/material-design-awards-2018/).

***

### [Adding a Floating Action Button](https://developer.android.com/guide/topics/ui/floating-action-button) ([FAB](https://developer.android.com/reference/android/support/design/widget/FloatingActionButton))

A [FAB](https://material.io/develop/android/components/floating-action-button/) ![](https://developer.android.com/codelabs/kotlin-android-training-material-design-dimens-colors/img/3cce9a1a26e2b4d2.png)is a large round button that represents a primary action, which is the main thing a user should do on the screen. The FAB floats above all other content, as shown in the screenshot on the left below. When the user taps the FAB, they are taken to a list of GDGs, as shown on the right.

#### Step 1: Add a FAB to the home fragment layout

1. In `build.gradle(Module: app)`, verify that the material library is included. To use Material Design components, you need to include this library. Make sure to use the latest library version.

```kotlin
implementation 'com.google.android.material:material:1.2.0'
```

2. Open `res/layout/home_fragment.xml` and switch to the **Text** tab.  
      
    Currently, the home screen layout uses a single `ScrollView` with a `ConstraintLayout` as a child. If you added a FAB to the `ConstraintLayout`, the FAB would be inside the `ConstraintLayout`, not floating over all the content, and it would scroll with the rest of the content of the `ConstraintLayout`. You need a way to float the FAB above your current layout.  
      
    [`CoordinatorLayout`](https://developer.android.com/reference/androidx/coordinatorlayout/widget/CoordinatorLayout) is a view group that lets you stack views on top of each other. While `CoordinatorLayout` doesn't have any fancy layout abilities, it is sufficient for this app. The `ScrollView` should take up the full screen, and the FAB should float near the bottom edge of the screen.

3. In `home_fragment.xml`, add a `CoordinatorLayout` around the `ScrollView`.

```xml
<androidx.coordinatorlayout.widget.CoordinatorLayout
       android:layout_height="match_parent"
       android:layout_width="match_parent">
...

</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

4. Replace <`ScrollView>` with <`androidx.core.widget.NestedScrollView>`. Coordinator layout knows about scrolling, and you need to use `NestedScrollView` inside another view with scrolling for scrolling to work correctly.

```xml
androidx.core.widget.NestedScrollView
```

5. Inside and at the bottom of the `CoordinatorLayout`, below the `NestedScrollView`, add a `FloatingActionButton`.

```xml
<com.google.android.material.floatingactionbutton.FloatingActionButton
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"/>
```

6. Run your app. You see a colored dot in the top-left corner.

![](https://developer.android.com/codelabs/kotlin-android-training-material-design-dimens-colors/img/f6113287311da873.png)

7. Tap the button. Notice that the button visually responds.
8. Scroll the page. Notice that the button stays put.

#### Step 2: Style the FAB

In this step, you move the FAB to the bottom-right corner and add an image that indicates the FAB's action.

1. Still in `home_fragment.xml`, add the `layout_gravity` attribute to the FAB and move the button to the `bottom` and `end` of the screen. The `layout_gravity` attribute tells a view to lay out on the top, bottom, start, end, or center of the screen. You can combine positions using a vertical bar.

```xml
android:layout_gravity="bottom|end"
```

2. Check in the **Preview** pane that the button has moved.
3. Add a `layout_margin` of `16dp` to the FAB to offset it from the edge of the screen.

```xml
android:layout_margin="16dp"
```

4. Use the provided `ic_gdg` icon as the image for the FAB. After you add the code below, you see a chevron inside the FAB.

```xml
app:srcCompat="@drawable/ic_gdg"
```

5. Run the app, and it should look like the screenshot below.

![](https://developer.android.com/codelabs/kotlin-android-training-material-design-dimens-colors/img/644ebea1e6111ae0.png)

#### Step 3: Add a click listener to the FAB

In this step, you add a click handler to the FAB that takes the user to a list of GDGs. You've added click handlers in previous codelabs, so the instructions are terse.

1. In `home_fragment.xml`, in the `<data>` tag, define a variable `viewModel` for the provided `HomeViewModel`.

```xml
<variable
   name="viewModel"
   type="com.example.android.gdgfinder.home.HomeViewModel"/>
```

2. Add the `onClick` listener to the FAB and call it `onFabClicked()`.

```xml
android:onClick="@{() -> viewModel.onFabClicked()}"
```

3. In the **home** package, open the provided `HomeViewModel` class and look at the navigation live data and functions, which are also shown below. Notice that when the FAB is clicked, the `onFabClicked()`click handler is called, and the app triggers navigation.

```kotlin
private val _navigateToSearch = MutableLiveData<Boolean>()
val navigateToSearch: LiveData<Boolean>
   get() = _navigateToSearch

fun onFabClicked() {
   _navigateToSearch.value = true
}

fun onNavigatedToSearch() {
   _navigateToSearch.value = false
}
```

4. In the **home** package, open the `HomeFragment` class. Notice that `onCreateView()` creates the `HomeViewModel` and assigns it to `viewModel`.
5. Add the `viewModel` to the `binding` in `onCreateView()`.

```kotlin
binding.viewModel = viewModel
```

6. To eliminate the error, clean and rebuild your project to update the binding object.
7. Also in `onCreateView()`, add an observer that navigates to the list of GDGs. Here is the code:  
    
```kotlin
viewModel.navigateToSearch.observe(viewLifecycleOwner,
    Observer<Boolean> { navigate ->
        if(navigate) {
            val navController = findNavController()
            navController.navigate(R.id.action_homeFragment_to_gdgListFragment)
            viewModel.onNavigatedToSearch()
       }
     })
```

8. Make the necessary imports from `androidx`. Import this `findNavController` and `Observer`:

```kotlin
import androidx.navigation.fragment.findNavController
import androidx.lifecycle.Observer
```

9. Run your app.
10. Tap the FAB and it takes you to the GDG list. If you are running the app on a physical device, you are asked for the location permission. If you are running the app in an emulator, you may see a blank page with the following message:  
    ![](https://developer.android.com/codelabs/kotlin-android-training-material-design-dimens-colors/img/a024bbdad6b8fc51.png)  
      
    If you get this message running on the emulator, make sure you are connected to the internet and have location settings turned on. Then open the Maps app to enable location services. You may also have to restart your emulator.
    
***

### Styling in a material world

To get the most out of Material Design components, use theme attributes. Theme attributes are variables that point to different types of styling information, such as the primary color of the app. By specifying theme attributes for the `MaterialComponents` theme, you can simplify your app styling. Values you set for colors or fonts apply to all widgets, so you can have consistent design and branding.

Sometimes you may want to change portions of your screen to a different theme, but not all of it. For example, you could make the toolbar use the dark Material components theme. You do this using theme overlays.

A `Theme` is used to set the global theme for the entire app. A `ThemeOverlay` is used to override (or "overlay") that theme for specific views, especially the toolbar.

Theme overlays are "thin themes" that are designed to overlay an existing theme, like icing on top of a cake. They are useful when you want to change a subsection of your app, for example, change the toolbar to be dark, but continue using a light theme for the rest of the screen. You apply a theme overlay to a view, and the overlay applies to that view and all its children.

You do this by applying the desired theme to the root view of the view hierarchy for which you want to use it. This does not change anything yet! When you want a view in the hierarchy to use a particular attribute that's defined in the overlay theme, you specifically set the attribute on the view and set the value to `?attr/`_`valuename`_.

***

### Add Material Attributes and Overlay

#### Step 1: Use Material theme attributes

In this step, you change the styling of the title headers on the home screen to use Material Design theme attributes to style your views. This helps you follow the Material style guide while customizing your app's style.

1. Open the Material web page for [typography theming](https://material.io/develop/android/theming/typography/). The page shows you all the styles available with Material themes.
2. On the page, search or scroll to find `textAppearanceHeadline5` (Regular 24sp) and `textAppearanceHeadline6` (Regular 20sp). These two attributes are good matches for your app.
3. In `home_fragment.xml`, replace the current style (`android:textAppearance="@style/TextAppearance.Title"`) of the `title` `TextView` with `style="?attr/textAppearanceHeadline5"`. The syntax `?attr` is a way to look up a theme attribute and apply the value of Headline 5, as defined in the current theme.

```xml
<TextView
       android:id="@+id/title"
       style="?attr/textAppearanceHeadline5"
```

4. Preview your changes. Notice that when the style is applied, the font of the title changes. This happens because the style set in the view overrides the style set by the theme, as shown by the style-priority pyramid diagram below.

![](https://developer.android.com/codelabs/kotlin-android-training-material-design-dimens-colors/img/e7851427054b568d.png)

In the pyramid diagram, `TextAppearance` is below theme. `TextAppearance` is an attribute on any view that applies text styling. It's not the same as a style, and it only lets you define things about how to display text. All the text styles in Material Design components can also be used as `textAppearance`; that way, any theme attributes that you define take priority.

5. In the `title` `TextView`, replace the styling you just added with a `textAppearance`.

6. Preview your changes, or run the app to see the difference. For comparison, the screenshots below show the differences when the Material style is applied to the title and overrides the `Title` style.

|     |     |
| --- | --- |
| MATERIAL STYLE:<br><br>`style="?attr/textAppearanceHeadline5"` | TEXT APPEARANCE<br><br>`android:textAppearance="?attr/textAppearanceHeadline5"` |
| ![](https://developer.android.com/codelabs/kotlin-android-training-material-design-dimens-colors/img/4176fbf73f75c176.png) | ![](https://developer.android.com/codelabs/kotlin-android-training-material-design-dimens-colors/img/4422770cb6fa4154.png) |

#### Step 2: Change the style in the Material theme

The `textAppearanceHeadline6` would be a good Material choice for the `subtitle`, but its default size is 20sp, not 18sp as defined in the `Title` style of the app. Instead of overriding the size in each subtitle view, you can modify the Material theme and override its default style.

1. Open `styles.xml`.
2. Delete the `Title` and `Subtitle` styles. You are already using `textAppearanceHeadline5` instead of `Title`, and you are going to remove the need for `Subtitle`.
3. Define a `CustomHeadline6` style with a `textSize` of 18sp. Give it a `parent` of `TextAppearance.MaterialComponents.Headline6` so that it inherits everything that you are not explicitly overriding.

```xml
<style name="TextAppearance.CustomHeadline6" parent="TextAppearance.MaterialComponents.Headline6">
   <item name="android:textSize">18sp</item>
</style>
```

4. Inside this style, override the default `textAppearanceHeadline6` of the theme, by defining an item that styles `textAppearanceHeadline6` with the custom style that you just added.

```xml
<item name="textAppearanceHeadline6">@style/TextAppearance.CustomHeadline6</item>
```

5. In `home_fragment.xml`, apply `textAppearanceHeadline6` to the `subtitle` view and reformat your code (**Code > Reformat code**).

```xml
android:textAppearance="?attr/textAppearanceHeadline6"
```

#### Step 3: Use theme overlays

The `MaterialComponents` theme does not have an option for a dark toolbar on a light screen. In this step, you change the theme for just the toolbar. You make the toolbar dark by applying the `Dark` theme, which is available from `MaterialComponents`, to the toolbar as an overlay.

1. Open `activity_main.xml` and find where the `Toolbar` is defined (`androidx.appcompat.widget.Toolbar`). The `Toolbar` is part of Material Design and allows more customization than the app bar that activities use by default.
2. To change the toolbar and any of its children to the dark theme, start by setting the theme of the `Toolbar` to the `Dark.ActionBar` theme. Make sure you do this in the `Toolbar`, not in the `ImageView`.

```xml
<androidx.appcompat.widget.Toolbar
    android:theme="@style/ThemeOverlay.MaterialComponents.Dark.ActionBar"
```

3. Change the background of the `Toolbar` to `colorPrimaryDark`.

```xml
android:background="?attr/colorPrimaryDark"
```

Your styled `Toolbar` should look like this:

```xml
<androidx.appcompat.widget.Toolbar
        android:id="@+id/toolbar"
        android:theme="@style/ThemeOverlay.MaterialComponents.Dark.ActionBar"
        android:background="?colorPrimaryDark"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize">
```

![](https://developer.android.com/codelabs/kotlin-android-training-material-design-dimens-colors/img/bc61db18159a630b.png)

Notice the image in the header (`drawable/logo.png`), which includes both the colored arrow and the gray "GDG Finder" text on a transparent background. With the changed background color, the text does not stand out as much anymore. You can create a new image, or to increase contrast without creating a new image, you can set a tint on the `ImageView`. That causes the whole `ImageView` to be "tinted" to the specified color. The `colorOnPrimary` attribute is a color that meets accessibility guidelines for text or iconography when drawn on top of the primary color.

4. In the `ImageView` inside the `Toolbar`, set the tint to `colorOnPrimary`. Because the drawable includes both the image and the GDG Finder text, both will be light.

```xml
android:tint="?attr/colorOnPrimary"
```

5. Run the app and notice the dark header from the theme. The `tint` is responsible for the light logo image, which includes the icon and the "GDG Finder" text.

***

### Consistent Styling

A professional-looking app has a consistent look and feel. It has the same colors and similar layouts on every screen. This makes the app not only look better, but makes it easier for users to understand and interact with the screens.

Dimens, or dimensions, allow you to specify reusable measurements for your app. Specify things like margins, heights, or padding using dp. Specify font sizes using sp.

* Android provides a dimens or dimensions file that you can use to specify the dimensions of your app.
* You might use dimensions for things like the left margin or the height of a view that you regularly build in your app. You can even use dimens to change the size of things depending on the screen.
* For example, you might make a button a completely different size on a tablet and a phone.
* By declaring your dimensions once instead of on every view, if you end up changing one,
it's easy to apply that change throughout your app.

Android also has a color resource for specifying colors.

* So far this lesson, we have hard-coded colors in the app, which is a pretty messy way to maintain code.
* It's really easy to forget to update one place if a color changes.
* By using colors and material theming, it's easy to apply consistent colors throughout your app.

***

### Use Dimens Resources

#### Step 1: Examine your code

1. Open `home_fragment xml`.
2. Look at the **Design** tab and make sure the blueprint is visible.
3. In the **Component Tree**, select the `start_guideline` and `end_guideline`. They should be slightly highlighted in the blueprint.

|     |     |
| --- | --- |
| ![](https://developer.android.com/codelabs/kotlin-android-training-material-design-dimens-colors/img/5a54ef7eb2a44476.png) | ![](https://developer.android.com/codelabs/kotlin-android-training-material-design-dimens-colors/img/c1acf731025b38b7.png) |

4. Notice the number 16 on the left and the number 26 on the right, indicating the inset of the start and end guidelines, respectively.
5. Switch to the **Text** tab.
6. Notice that at the bottom of the `ConstraintLayout`, two `Guidelines` are defined. Guidelines define vertical or horizontal lines on your screen that define the edges of your content. Everything is placed inside these lines except for full-screen images.

```xml
<androidx.constraintlayout.widget.Guideline
   android:id="@+id/start_guideline"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:orientation="vertical"
   app:layout_constraintGuide_begin="16dp" />

<androidx.constraintlayout.widget.Guideline
   android:id="@+id/end_guideline"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:orientation="vertical"
   app:layout_constraintGuide_end="26dp" />
```

The `layout_constraintGuide_begin="16dp"`  is correct according to the Material specifications. But `app:layout_constraintGuide_end="26dp"`  should be 16dp also. You could just fix this here manually. However, it is better to create a dimension for these margins and then apply them consistently throughout your app.

#### Step 2: Create a dimension

1. In `home_fragment xml`, using the **Text** view, put your cursor on the `16dp` of `app:layout_constraintGuide_begin="16dp"`.
2. Open the intentions menu and select **Extract dimension resource.**

![](https://developer.android.com/codelabs/kotlin-android-training-material-design-dimens-colors/img/d2bc010a46311da8.png)

3. Set the **Resource Name** of the dimension to `spacing_normal`. Leave everything else as given and click **OK**.
4. Fix `layout_constraintGuide_end` to also use the `spacing_normal` dimension.

```xml
<androidx.constraintlayout.widget.Guideline
       android:id="@+id/end_grid"
       app:layout_constraintGuide_end="@dimen/spacing_normal"
```

5. In Android Studio, open **Replace All** (`Cmd+Shift+R` on the Mac or `Ctrl+Shift+R` on Windows).
6. Search for 16dp and replace all occurrences, except the one in `dimens.xml`, with `@dimen/spacing_normal`.

![](https://developer.android.com/codelabs/kotlin-android-training-material-design-dimens-colors/img/73840e6c21e577fb.png)

7. Open `res/values/dimens.xml` and notice the new `spacing_normal` dimension. Make sure you didn't accidentally replace the 16dp with a self-reference!
8. Run your app.
9. Notice that the spacing on the left and right of the text is now the same. Also, notice that the spacing affects line breaks, and the text in the right screenshot is one line shorter.

***

### Add Material Colors

By using color resources and Material theming, you can apply consistent colors throughout your app. Effective use of color can dramatically improve the usability of your app. Picking the best colors and color combinations can be challenging, but there are tools to help.

One of the available tools is the [Color Tool.](https://material.io/tools/color/) To get a full Material color scheme for your app from the tool, you choose two main base colors, and then the tool creates the remaining colors appropriately.

In this task, you create a Material color scheme and apply it to your app.

#### Step 1: Create a Material color scheme

1. Open [https://material.io/tools/color/](https://material.io/tools/color/). You can use this tool to explore color combinations for your UI.

![](https://developer.android.com/codelabs/kotlin-android-training-material-design-dimens-colors/img/c9d10c153ac5a3cb.png)

2. Scroll down in the **MATERIAL PALETTE** on the right to see more colors.
3. Click **Primary**, then click a color. You can use any color you like.
4. Click **Secondary** and click a color.
5. Click **Text** to choose a text color if you want a color that is different from the one the tool has calculated for you. Pick various text colors to explore contrast.
6. Click the **ACCESSIBILITY** tab. You will see a report like the one below. This gives you a report about how readable the currently selected color choices are.

![](https://developer.android.com/codelabs/kotlin-android-training-material-design-dimens-colors/img/50955d37bfabb701.png)

7. Look for the triangular exclamation mark icons.

![](https://developer.android.com/codelabs/kotlin-android-training-material-design-dimens-colors/img/c2848a477cbac4d9.png)

**Note:** The tool bases its evaluation upon how good humans are at seeing, which takes into account various forms of vision impairment. For many impairments, better contrast makes it easier to read text. Even if you have no trouble reading your text, when you ship your app to millions of users, this tool will help you know that your users can read the text too!

8. In the tool, switch to the **CUSTOM** tab and enter the following two colors.

* **Primary: #669df6**
* **Secondary: #a142f4**

The primary color is a blue that's based on the color used in the GDG logo. The secondary color is based on the balloons in the image on the home screen. Type the colors into the **Hex Color** field. Keep the black and white fonts suggested by the tool.

Note that this color scheme still has some accessibility warnings. Most color schemes do. You work around this in the next step.

9. On the top-right of the window, select **EXPORT** and **ANDROID**. The tool initiates a download.
10. Save the `colors.xml` file in a convenient location.

#### Step 2: Apply the Material color scheme to your app

1. Open the downloaded `colors.xml` file in a text editor.
2. In Android Studio, open `values`/`colors.xml`.
3. Replace the resources of `values`/`colors.xml` with the contents of the downloaded `colors.xml` file.
4. Open `styles.xml.`
5. Inside the `AppTheme`, delete the colors that show as errors: `colorPrimary`, `colorPrimaryDark`, and `colorAccent`.
6. Inside `AppTheme`, define 6 attributes with these new colors, as shown in the code below. You can find the list in the [Color Theming](https://material.io/develop/android/theming/color/) guide.

```xml
<item name="colorPrimary">@color/primaryColor</item>
<item name="colorPrimaryDark">@color/primaryDarkColor</item>
<item name="colorPrimaryVariant">@color/primaryLightColor</item>
<item name="colorOnPrimary">@color/primaryTextColor</item>
<item name="colorSecondary">@color/secondaryColor</item>
<item name="colorSecondaryVariant">@color/secondaryDarkColor</item>
<item name="colorOnSecondary">@color/secondaryTextColor</item>
```
7.  Run the app. This looks pretty good...

![](https://developer.android.com/codelabs/kotlin-android-training-material-design-dimens-colors/img/d5bc37efa248cafe.png)

8. However, notice that `colorOnPrimary` isn't light enough for the logo tint (including the "GDG Finder text") to stand out sufficiently when displayed on top of `colorPrimaryDark`.
9. In `activity_main.xml`, find the `Toolbar`. In the `ImageView`, change the logo tint to `colorOnSecondary`.
10. Run the app.

##### Notes:

* `colorOnPrimary`: A color that passes accessibility guidelines for text and iconography when drawn on top of the primary color.
* `colorOnSecondary`: A color that passes accessibility guidelines for text and iconography when drawn on top of the secondary color.
* For more information, see [Color Theming](https://material.io/develop/android/theming/color/).

***

### Design for everyone

To make an app usable by the most users makes sense, whether you are developing for the joy of it, or for business purposes. There are multiple dimensions to accomplishing that.

* **Support RTL Languages.** European and many other languages read from left to right, and apps originating from those locales are commonly designed to fit well for those languages. Many other languages with a large number of speakers read from right to left, such as Arabic. Make your app work with right-to-left (RTL) languages to increase your potential audience.
* **Scan for accessibility.** Guessing at how someone else may experience your app is an option with pitfalls. The [Accessibility Scanner](https://play.google.com/store/apps/details?id=com.google.android.apps.accessibility.auditor) app takes the guesswork out of the loop and analyses your app, identifying where you could improve its accessibility.
* **Design for TalkBack with content descriptions.** Visual impairments are more common than one might think, and many users, not just blind ones, use a screen reader. Content descriptions are phrases for a screen reader to say when a user interacts with an element of the screen.
* **Support night mode**. For many visually impaired users, changing the screen colors improves contrast and helps them visually work with your app. Android makes it straightforward to support night mode, and you should always support night mode to give users a simple alternative to the default screen colors.

***

### RTL languages

The main difference between left-to-right (LTR) and right-to-left (RTL) languages is the direction of the displayed content. When the UI direction is changed from LTR to RTL (or vice-versa), it is often called mirroring. Mirroring affects most of the screen, including text, text field icons, layouts, and icons with directions (such as arrows). Other items are not mirrored, such as numbers (clock, phone numbers), icons which do not have direction (airplane mode, WiFi), playback controls, and most charts and graphs.

Languages that use the RTL text direction are used by more than one billion people worldwide. Android developers are all over the world, and so a GDG Finder app needs to support RTL languages.

#### RTL APIs

Some `View` components need further customization to behave properly with RTL. In order to have more precise control over the UI there are 4 different APIs that you can use:

* [`android:layoutDirection`](https://developer.android.com/reference/android/util/LayoutDirection) to set the direction of a component's layout.
* [`android:textDirection`](https://developer.android.com/reference/android/view/View.html#attr_android:textDirection) to set the direction of a component's text.
* [`android:textAlignment`](https://developer.android.com/reference/android/view/View.html#attr_android:textAlignment) to set the alignment of a component's text.
* [`getLayoutDirectionFromLocale()`](https://developer.android.com/reference/android/support/v4/text/TextUtilsCompat#getlayoutdirectionfromlocale) is a method to programatically get the locale that specifies direction.

***

### Adding support for right-to-left (RTL) [languages](https://developer.android.com/training/basics/supporting-devices/languages)

#### Step 1: Add RTL support

1. Open the Android Manifest. In the `<application>` section, add the following code to specify that the app supports RTL.

```xml
<application
             ...        
             android:supportsRtl="true">
```

2. Open **activity_main.xml** in the **Design** tab. From the **Locale for Preview** dropdown menu, choose **Preview Right to Left**. (If you don't find this menu, widen the pane or close the **Attributes** pane to uncover it.)

![5ead4984f44f7f70.png](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/5ead4984f44f7f70.png)

**Tip:** You don't need to switch languages on your device to check how your app presents visually in RTL.

3. In the **Preview**, notice that the header "GDG Finder" has moved to the right, and the rest of the screen remains pretty much the same. Overall, this screen is passable. But the alignment in the text view is now wrong, because it is aligned to the left, instead of to the right.

![4e579e9e6ad9c160.png](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/4e579e9e6ad9c160.png)

4. To make this work on your device, in your device or emulator **Settings**, in **Developer Options**, select **Force RTL layout**. (If you need to turn on **Developer Options**, find the **Build Number** and click it until you get a toast indicating you are a developer. This varies by device and version of the Android system.)

![e3bb163b6e6d2043.png](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/e3bb163b6e6d2043.png)

5. Run the app and verify on the device that the main screen appears the same as in the **Preview**. Notice that the FAB is now switched to the left, and the **Hamburger** menu to the right!
6. In the app, open the navigation drawer and navigate to the **Search** screen. As shown below, the icons are still on the left, and no text is visible. It turns out that the text is off the screen, to the left of the icon. This is because the code uses left/right screen references in the view properties and layout constraints.

|     |     |
| --- | --- |
| ![](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/a116467373f381c2.png) | ![](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/5bb4809b1f086567.png) |

#### Step 2: Use start and end instead of left and right

"Left" and "right" on the screen (when you face the screen) don't change, even if the direction of the text changes. For example, `layout_constraintLeft_toLeftOf` always constrains the left side of the element to the left side of the screen. In your app's case, the text is off the screen in RTL languages, as shown in the screenshot above.

To fix this, instead of "left" and "right," use `Start` and `End` terminology. This terminology sets the start of the text and the end of the text appropriately for the direction of the text in the current language, so that margins and layouts are in the correct areas of the screens.

1. `Open` **list_item.xml**.
2. Replace any references to `Left` and `Right` with references to `Start` and `End`.

```xml
app:layout_constraintStart_toStartOf="parent"

app:layout_constraintStart_toEndOf="@+id/gdg_image"
app:layout_constraintEnd_toEndOf="parent"

```

3. Replace the `ImageView`'s `layout_marginLeft` with `layout_marginStart`. This moves the margin to the correct place to move the icon away from the edge of the screen.

```xml
android:layout_marginStart=""
```

4. Open `fragment_gdg_list.xml`. Check the list of GDGs in the **Preview** pane. Notice that the icon is still pointing in the wrong direction because it is mirrored (If the icon is not mirrored, make sure you are still viewing the right to left preview). According to the Material Design [guidelines](https://material.io/design/usability/bidirectionality.html%20for%20guidelines.), icons should not be mirrored.
5. Open **res/drawable/ic_gdg.xml**.
6. In the first line of XML code, find and delete `android:autoMirrored="true"` to disable mirroring.
7. Check the **Preview** or run the app again and open the Search GDG screen. The layout should be fixed now!

![a8222a7075614230.png](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/a8222a7075614230.png)

**Tip:** Android Studio gives you yellow-highlight tips to encourage the use of start and end attributes. Use search-and-replace with case matching to find all occurrences of `Left` and `Right` in your XML layouts.

#### Step 3: Let Android Studio do the work for you

In the previous exercise, you took your first steps to support RTL languages. Fortunately, Android Studio can scan your app and set up a lot of basics for you.

1. In **list_item.xml**, in the `TextView`, change `layout_marginStart` back to `layout_marginLeft`, so that the scanner has something to find.

```xml
<TextView
android:layout_marginLeft="@dimen/spacing_normal"
```

2. In Android Studio, choose **Refactor > Add RTL support where possible** and check the boxes for updating the manifest, and the layout files to use start and end properties.

![d1d8be2fce55cbc9.png](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/d1d8be2fce55cbc9.png)

3. In the **Refactoring Preview** pane, find the **app** folder, and expand until it is open to all the details.
4. Under the app folder, notice that the `layout_marginLeft` that you just changed is listed as code to refactor.

![32cc0cd2156afd37.png](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/32cc0cd2156afd37.png)

5. Notice that the preview also lists system and library files. Right-click on **layout** and l**ayout-watch-v20** and any other folders that are not part of **app**, and choose **Exclude** from the context menu.

![ee9efe287e5ffadd.png](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/ee9efe287e5ffadd.png)

6. Go ahead and do the refactor now. (If you get a popup about system files, make sure you have excluded all folders that are not part of your **app** code.)
7. Notice that `layout_marginLeft` has been changed back to `layout_marginStart`.

#### Step 3: Explore folders for locales

So far, you've just changed the direction of the default language used for the app. For a production app, you would send the **strings.xml** file to a translator to have it translated to a new language. For this codelab, the app provides a **strings.xml** file in Spanish (we used Google Translate to generate the translations, so they're not perfect.).

1. In Android Studio, switch the project view to **Project Files.**
2. Expand the **res** folder, and notice folders for **res/values** and **res/values-es**. The "es" in the folder name is the [language code](https://www.loc.gov/standards/iso639-2/php/code_list.php) for Spanish. The **values-"language code"** folders contain values for each supported language. The **values** folder without an extension contains the default resources that apply otherwise.

![cc381e15232c5f85.png](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/cc381e15232c5f85.png)

3. In **values-es**, open **strings.xml** and notice that all the strings are in Spanish.
4. In Android Studio, open `activity_main.xml` in the **Design** tab.
5. In the **Locale for Preview** dropdown choose **Spanish**. Your text should now be in Spanish.

![1d05d877bfe738f1.png](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/1d05d877bfe738f1.png)

6. **\[Optional\]** If you are proficient in an RTL language, create a **values** folder and a **strings.xml** in that language and test how it appears on your device.
7. **\[Optional\]** Change the language settings on your device and run the app. Make sure you don't change your device to a language you don't read, as it makes it a bit challenging to undo!

***

### Install and Use the Scanner

The [Accessibility Scanner app](https://play.google.com/store/apps/details?id=com.google.android.apps.accessibility.auditor) is your best ally when it comes to [making](https://developer.android.com/guide/topics/ui/accessibility/testing) your app accessible. It scans the visible screen on your target device and suggests improvements, such as making touch targets larger, increasing contrast, and providing descriptions for images to make your app more accessible. Accessibility Scanner is made by Google and you can install it from Play Store.

1. Open the Play Store and sign in if necessary. You can do this on your physical device or the emulator. This codelab uses the emulator.

**Note:** If you don't have the Play Store on your emulator, you might need to create a new emulator. Make sure you select an option that has the "Play Store" icon enabled when creating the emulator.

![36b956b4317153de.png](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/36b956b4317153de.png)

![786f10d839b88a3.png](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/786f10d839b88a3.png)

**Important:** To use the Play Store from the emulator, you have to log in with an account. You can use your regular account, or you can create an account just for testing with the emulator. Any account you create is real, and you are connecting to the real Play Services, not an emulated version of it!

**Important:** You can **Skip** setting up billing options!

2. In the Play Store, search for [**Accessibility Scanner by Google LLC**](https://play.google.com/store/apps/details?id=com.google.android.apps.accessibility.auditor). Make sure you get the correct app, issued by Google, as any scanning requires a lot of permissions!

![ce1df86f749f67fb.png](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/ce1df86f749f67fb.png)

3. Install the scanner on the emulator.
4. Once installed, click **Open**.
5. Click **Get Started**.
6. Click **OK** to start Accessibility Scanner setup in Settings.

![55977701e95a0f2.png](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/55977701e95a0f2.png)

7. Click the Accessibility Scanner to go to the device's **Accessibility** settings.

![7bf5aa22a0bf73b6.png](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/7bf5aa22a0bf73b6.png)

8. Click **Use service** to enable it.

![88df77fec258fab6.png](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/88df77fec258fab6.png)

9. Follow the on-screen instructions and grant all permissions.
10. Then click **OK Got it**, and return to the home screen. You may see a blue button with a check mark somewhere on the screen. Clicking this button triggers testing for the app in the foreground. You can reposition the button by dragging it. This button stays on top of any apps, so you can trigger testing at any time.

![d10b3cb15ee273b3.png](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/d10b3cb15ee273b3.png)

To turn off the scanner, go to **Settings** and turn the Accessibility Scanner off.

11. Open or run your app.
12. Click the blue button and accept additional security warnings and permissions.

The first time you click the Accessibility Scanner icon, the app asks for permission to get everything displayed on your screen. This seems like a very scary permission, and it is.

You should almost never grant a permission like this one, because the permission lets apps read your email or even grab your bank account info! However, for Accessibility Scanner to do its work, it needs to examine your app the way a user would—that's why it needs this permission.

**Important:** Before you enable this permission, double-check to make sure that the app you installed is [Accessibility Scanner by Google LLC](https://play.google.com/store/apps/details?id=com.google.android.apps.accessibility.auditor).

13. Click the blue button and wait for the analysis to complete. You will see something like the screenshot below, with the title and FAB boxed in red. This indicates two suggestions for accessibility improvements on this screen.

![96f3c0da0fa2e27.png](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/96f3c0da0fa2e27.png)

14. Click on the box surrounding GDG Finder. This opens a pane with additional information, as shown below, indicating issues with the image contrast.
15. Expand the **Image Contrast** information, and the tool suggests remedies.
16. Click the arrows to the right to get information for the next item.

|     |     |
| --- | --- |
| ![](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/fc7d26173e57dc16.png) | ![](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/bb251499f78515da.png) |

17. In your app, navigate to the **Apply for GDG** screen and scan it with the Accessibility Scanner app. This gives quite a few suggestions, as shown below on the left. 12, to be exact. To be fair, some of those are duplicates for similar items.
18. Click the "stack" ![804718d5989bce70.png](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/804718d5989bce70.png) icon in the bottom toolbar to get a list of all suggestions, as shown below on the right screenshot. You address all of these issues in this codelab.

 ![](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/ac4eb1b2948b674b.png) 
 ![](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/88887a1afe7c61e9.png)  ![](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/134124a45c47dffd.png) 
 
 ***
 
### Design for TalkBack
The [Android Accessibility Suite](https://play.google.com/store/apps/details?id=com.google.android.marvin.talkback), a collection of apps by Google, includes tools to help make apps more accessible. It includes tools such as TalkBack. TalkBack is a screen reader that offers auditory, haptic, and spoken feedback, which allows users to navigate and consume content on their devices without using their eyes.

It turns out TalkBack is not only used by blind people, but by many people who have visual impairments of some sort. Or even people who just want to rest their eyes!

So, Accessibility is for everyone! In this task, you try out TalkBack and update your app to work well with it.

TalkBack comes pre-installed on many physical devices, but on an emulator, you need to install it.

1. Open the Play Store.
2. Find the [Accessibility Suite](https://play.google.com/store/apps/details?id=com.google.android.marvin.talkback). Make sure it is the correct app by Google.
3. If it is not installed, install the Accessibility Suite.
4. To enable TalkBack on the device, navigate to **Settings > Accessibility** and turn TalkBack on by selecting **Use Service**. Just like the accessibility scanner, TalkBack requires permissions in order to read the contents on the screen. Once you accept the permission requests, TalkBack welcomes you with a list of tutorials to teach you how to use TalkBack effectively.
5. Pause here and take the tutorials, if for no other reason than to learn how to turn TalkBack back off when you are done.
6. To leave the tutorial, click the back button to select it, then double-tap anywhere on the screen.

**Tip:** If you can't hear anything...turn on the volume!

**Tip:** To click an item, double-tap it; to scroll or drag, double-tap and drag.

7. Explore using the GDG Finder app with TalkBack. Notice places where TalkBack gives you no useful information about the screen or a control. You are going to fix this in the next exercise.

***

### Adding Content Descriptions
Content descriptors are descriptive labels that explain the meaning of views. Most of your views should have content descriptions.

#### Step 1: Add a content description

1. With the GDG Finder app running and Talkback enabled, navigate to the **Apply to run GDG** screen.
2. Tap on the main image ... and nothing happens.
3. Open **add\_gdg\_fragment.xml**.
4. In the `ImageView`, add a content descriptor attribute as shown below. The `stage_image_description` string is provided for you in **strings.xml**.

```xml
android:contentDescription="@string/stage_image_description"
```

5. Run your app.
6. Navigate to **Apply to run GDG** and click on the image. You should now hear a short description of the image.
7. **\[Optional\]** Add content descriptions for the other images in this app. In a production app, all images need to have content descriptions.

#### Step 2: Add hints to editable text fields

For editable elements, such as an `EditText`, you can use `android:hint` in the XML to help users figure out what to type. A hint always shows in the UI as it is the default text in an input field.

1. Still in **add\_gdg\_fragment.xml**.
2. Add content descriptions and hints, using the code below as guidance.

Add to `textViewIntro`:

```xml
android:contentDescription="@string/add_gdg"
```

Add to the edit texts respectively:

```xml
android:hint="@string/your_name_label"

android:hint="@string/email_label"

android:hint="@string/city_label"

android:hint="@string/country_label"

android:hint="@string/region_label"
```

3. Add a content description to `labelTextWhy`.

```xml
android:contentDescription="@string/motivation"
```

4. Add a hint to `EditTextWhy`. Once you have labeled the edit boxes, add a content description to the label and also a hint to the box.

```xml
android:contentDescription="@string/motivation"
```

5. Add a content description for the submit button. All buttons need to have a description of what happens if they are pressed.

```xml
android:contentDescription="@string/submit_button_description"
```

6. Run your app with Talkback enabled, and fill out the form to apply to run a GDG.

***

### Adding Content Grouping

For UI controls that TalkBack should treat as a group, you can use content grouping. Related content which is grouped together, is announced together. Users of assistive technology then won't need to swipe, scan, or wait as often to discover all information on the screen. This does not affect how the controls appear on the screen.

To group UI components, wrap them into a `ViewGroup`, such as a `LinearLayout`. In the GDG Finder app, the `labelTextWhy` and `editTextWhy` elements are excellent candidates for grouping since they belong together semantically.

1. Open **add\_gdg\_fragment.xml**.
2. Wrap a `LinearLayout` around `LabelTextWhy` and `EditTextWhy` to create a content group. Copy and paste the code below. This `LinearLayout` already contains some of the styling that you need. (Make sure the `button` is OUTSIDE the `LinearLayout`.)

```xml
<LinearLayout android:id="@+id/contentGroup" android:layout_width="match_parent"
            android:layout_height="wrap_content" android:focusable="true"
            app:layout_constraintTop_toBottomOf="@id/EditTextRegion"
            android:orientation="vertical" app:layout_constraintStart_toStartOf="@+id/EditTextRegion"
            app:layout_constraintEnd_toEndOf="@+id/EditTextRegion"
            android:layout_marginTop="16dp" app:layout_constraintBottom_toTopOf="@+id/button"
            android:layout_marginBottom="8dp">

     <!-- label and edit text here –>

</LinearLayout>
```

3. Choose **Code > Reformat code** to properly indent all the code.
4. Remove all layout margins from `labelTextWhy` and `editTextWhy`.
5. In `labelTextWhy`, change the `layout_constraintTop_toTopOf` constraint to `contentGroup`.

```xml
app:layout_constraintTop_toTopOf="@+id/contentGroup" />
```

6. In `editTextWhy` change the `layout_constraintBottom_toBottomOf` constraint to `contentGroup`.

```xml
app:layout_constraintBottom_toBottomOf="@+id/contentGroup"
```

7. Constrain `EditTextRegion` and the `Button` to the `contentGroup` to get rid of the errors.

```xml
app:layout_constraintBottom_toTopOf="@+id/contentGroup"
```

8. Add margins to the `LinearLayout`. Optionally, extract this margin as a dimension.

```xml
android:layout_marginStart="32dp"
android:layout_marginEnd="32dp"
```

If you need help, check your code against the [`add_gdg_fragment.xml`](https://github.com/google-developer-training/android-kotlin-fundamentals-apps/blob/master/GDGFinderFinal/app/src/main/res/layout/add_gdg_fragment.xml) in the solution code.

9. Run your app and explore the **Apply to run GDG** screen with TalkBack.

***

### Adding a Live Region

Currently, the label on the submit button is **OK**. It would be better if the button had one label and description before the form is submitted, and dynamically changed to a different label and content description after the user clicks and the form has been submitted. You can do this using a live region.

A live region indicates to accessibility services whether the user should be notified when a view changes. For example, informing the user about an incorrect password or a network error is a great way to make your app more accessible. In this example, to keep it simple, you inform the user when the submit button changes its state.

1. Open **add\_gdg\_fragment.xml**.
2. Change the button's text assignment to **Submit** using the provided `submit` string resource.

```xml
android:text="@string/submit"
```

3. Add a live region to the button by setting the `android:accessibilityLiveRegion` attribute. As you type, you have several options for its value. Depending on the importance of the change, you can choose whether to interrupt the user. With the value "assertive", the accessibility services interrupt ongoing speech to immediately announce changes to this view. If you set the value to "none", no changes are announced. Set to "polite", the accessibility services announce changes, but wait their turn. Set the value to "polite".

![811c1b498f177251.png](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/811c1b498f177251.png)

```xml
android:accessibilityLiveRegion="polite"
```

4. In the **add** package, open **AddGdgFragment.kt**.
5. Inside the `showSnackBarEvent` `Observer`, after you are done showing the `SnackBar`, set a new content description and text for the button.

```kotlin
binding.button.contentDescription=getString(R.string.submitted)
binding.button.text=getString(R.string.done)
```

6. Run your app and click the button. Unfortunately, the button and font are too small!

#### Fix the button styling

1. In **add\_gdg\_fragment.xml**, change the button's `width` and `height` to `wrap_content`, so the full label is visible and the button is a good size.

```xml
android:layout_width="wrap_content"android:layout_height="wrap_content"
```

2. Delete the `backgroundTint`, `textColor`, and `textSize` attributes from the button so that the app uses the better theme styling.
3. Delete the `textColor` attribute from `textViewIntro`. The theme colors should provide good contrast.
4. Run the app. Notice the much more usable **Submit** button. Click **Submit** and notice how it changes to **Done**.

 ![](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/aa96e903f0d84f29.png)  ![](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/7b1eae8148504314.png) 
 
 ***
 
### Using drawables

Android drawables let you draw images, shapes, and animations on the screen, and they can have a fixed size or be dynamically sized. You can use images as drawables, such as the images in the GDG app. And you can use vector drawings to draw anything you can imagine. There is also a resizable drawable called a [9-patch drawable](https://developer.android.com/guide/topics/graphics/drawables#nine-patch), that is not covered in this course. The GDG logo, in **drawable/ic_gdg.xml**, is another drawable.

Drawables are not views, so you cannot put a drawable directly inside a `ConstraintLayout`, you need to put it inside an `ImageView`. You can also use drawables to provide a background for a text view or a button, and the background draws behind the text.

#### Chips

Chips are compact elements that represent an attribute, text, entity, or action. They allow users to enter information, select a choice, filter content, or trigger an action.

The `Chip` widget is a thin view wrapper around the `ChipDrawable`, which contains all of the layout and draw logic. The extra logic exists to support touch, mouse, keyboard, and accessibility navigation. The main chip and close icon are considered to be separate logical sub-views, and contain their own navigation behavior and state.

![94b3efe32f8d486b.png](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/94b3efe32f8d486b.png)

***

### Add a ChipGroup

#### Step 1: Add chips to the list of GDGs

The checked chip below uses three drawables. The background and the check mark are each a drawable. Touching the chip creates a ripple effect, which is done with a special [RippleDrawable](https://developer.android.com/reference/android/graphics/drawable/RippleDrawable) that shows a ripple effect in response to state changes.

![871c97948c735089.png](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/871c97948c735089.png)

In this task, you add chips to the list of GDGs, and have them change state when they are selected. In this exercise, you add a row of buttons called _chips_ to the top of the **Search** screen. Each button filters the GDG list so that the user receives only results from the selected region. When a button is selected, the button changes its background and shows a check mark.

1. Open **fragment\_gdg\_list.xml**.
2. Create a `com.google.android.material.chip.ChipGroup` inside the `HorizontalScrollView.`Set its `singleLine` property to `true`, so that all chips are lined up on one horizontally scrollable line. Set the `singleSelection` property to `true` so that only one chip in the group can be selected at a time. Here is the code.

```xml
<com.google.android.material.chip.ChipGroup
    android:id="@+id/region_list"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:singleSelection="true"
    android:padding="@dimen/spacing_normal"/>
```

3. In the **layout** folder, create a new layout resource file called **region.xml**, for defining the layout for one `Chip`.
4. In **region.xml**, replace all the code with the layout for one `Chip` as given below. Notice that this `Chip` is a Material component. Also notice that you get the check mark by setting the property `app:checkedIconVisible`. You will get an error for the missing `selected_highlight` color.

```xml
<?xml version="1.0" encoding="utf-8"?>

<com.google.android.material.chip.Chip
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        style="@style/Widget.MaterialComponents.Chip.Choice"
        app:chipBackgroundColor="@color/selected_highlight"
        app:checkedIconVisible="true"
        tools:checked="true"/>
```

5. To create the missing `selected_highlight` color, put the cursor on `selected_highlight`, bring up the intention menu, and **create color resource for selected highlight**. The default options are fine, so just click **OK**. The file is created in the **res/color** folder.
6. Open **res/color/selected_highlight.xml**. In this color state list, encoded as a `<selector>`, you can provide different colors for different states. Each state and associated color is encoded as an `<item>`. See [Color Theming](https://material.io/develop/android/theming/color/) for more on these colors.
7. Inside the `<selector>`, add an item with a default color `colorOnSurface` to the state list. In state lists, it's important to always cover all the states. One way to do this is by having a default color.

```xml
<item android:alpha="0.18" android:color="?attr/colorOnSurface"/>
```

8. Above the default color, add an `item` with a color of `colorPrimaryVariant`, and constrain its use to when the selected state is `true`. State lists are worked through from the top to the bottom, like a case statement. If none of the states match, the default state applies.

```xml
<item android:color="?attr/colorPrimaryVariant"
         android:state_selected="true" />
```

#### Step 2: Display the row of chips

The GDG app creates a list of chips showing regions that have GDGs. When a chip is selected, the app filters results to only display GDG results for that region.

**Note:** How this list is created, and how the results are filtered, is not part of this codelab. If you would like to know how the list of GDGs is obtained, and how the filtering is implemented, examine the files in the **search** and **network** packages. In particular, examine the `GdgListViewModel`.

1. In the **search** package, open **GdgListFragment.kt**.
2. In `onCreateView()`, just above the `return` statement, add an observer on `viewModel.regionList` and override `onChanged()`. When the list of regions provided by the view model changes, the chips needs to be recreated. Add a statement to return immediately if the supplied `data` is `null`.

```kotlin
viewModel.regionList.observe(viewLifecycleOwner, object: Observer<List<String>> {
        override fun onChanged(data: List<String>?) {
             data ?: return
        }
})
```

3. Inside `onChanged()`, below the null test, assign `binding.regionList` to a new variable called `chipGroup` to cache the `regionList`.

```kotlin
val chipGroup = binding.regionList
```

4. Below, create a new `layoutInflator` for inflating chips from `chipGroup.context`.

```kotlin
val inflator = LayoutInflater.from(chipGroup.context)
```

5. Clean and rebuild your project to get rid of the databinding error.

Below the inflator, you can now create the actual chips, one for each region in the `regionList`.

6. Create a variable, `children`, for holding all the chips. Assign it a mapping function on the passed in `data` to create and return each chip.

```kotlin
val children = data.map {}
```

7. Inside the map lambda, for each `regionName`, create and inflate a chip. The completed code is below.

```kotlin
val children = data.map { regionName ->
   val chip = inflator.inflate(R.layout.region, chipGroup, false) as Chip
   chip.text = regionName
   chip.tag = regionName
   // TODO: Click listener goes here.
   chip
}
```

8. Inside the lambda, just before returning the `chip`, add a click listener. When the `chip` is clicked, set its state to `checked`. Call `onFilterChanged()` in the `viewModel`, which triggers a sequence of events that fetches the result for this filter.

```kotlin
chip.setOnCheckedChangeListener { button, isChecked ->
   viewModel.onFilterChanged(button.tag as String, isChecked)
}
```

9. At the end of the lambda, remove all current views from the `chipGroup`, then add all the chips from `children` to the `chipGroup`. (You can't update the chips, so you have to remove and recreate the contents of the `chipGroup`.)

```kotlin
chipGroup.removeAllViews()

for (chip in children) {
   chipGroup.addView(chip)
}
```

Your completed observer should be as follows:

```kotlin
viewModel.regionList.observe(viewLifecycleOwner, object: Observer<List<String>> {
   override fun onChanged(data: List<String>?) {
       data ?: return

       val chipGroup = binding.regionList
       val inflator = LayoutInflater.from(chipGroup.context)

       val children = data.map { regionName ->
           val chip = inflator.inflate(R.layout.region, chipGroup, false) as Chip
           chip.text = regionName
           chip.tag = regionName
           chip.setOnCheckedChangeListener { button, isChecked ->
               viewModel.onFilterChanged(button.tag as String, isChecked)
           }
           chip
       }
       chipGroup.removeAllViews()

       for (chip in children) {
           chipGroup.addView(chip)
       }
   }
})
```

10. Run your app and search for GDGS to open the **Search** screen to use your new chips. As you click each chip, the app will display the filter groups below it.

![9e881cff459d0d4a.png](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/9e881cff459d0d4a.png)

**Important.** If starting maps does not help you get the list, to make this work on the emulator, close any running emulators. Run the app, start a new emulator, and start the maps app on that emulator, then run the app.

***

### Introducing Dark Mode

Night mode lets your app change its colors to a dark theme, for example, when the device settings have been set to enable night mode. In night mode, apps change their default light backgrounds to be dark, and change all the other screen elements accordingly.

* To enable dark theme, you change from the light theme to a theme called DayNight.
* It will use the same thing as light theme normally, but when the system asks for a dark theme, that will be used.
* Of course, changing system settings to try out dark theme would be pretty annoying, so there's a way to set the app to dark theme in code.
* Then for anything you want to change in dark mode, you can create a folder with the night qualifier, for example, you can have a different color theme in dark mode by creating a folder called values-night.

***

### Adding Dark Mode Support

#### Step 1: Enable night mode

To provide the dark theme for your app, you change its theme from the `Light` theme to a theme called `DayNight`. The `DayNight` theme appears light or dark, depending on the mode.

1. In `styles.xml,` change the `AppTheme` parent theme from `Light` to `DayNight`.

```xml
<style name="AppTheme" parent="Theme.MaterialComponents.DayNight.NoActionBar">
```

2. In `MainActivity`'s `onCreate()` method, call `AppCompatDelegate.setDefaultNightMode()` to programmatically turn on the dark theme.

```kotlin
AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_YES)
```

3. Run the app and verify it has switched to the dark theme.

![1e1198910f1a8b73.png](https://developer.android.com/codelabs/kotlin-android-training-design-for-everyone/img/1e1198910f1a8b73.png)

#### Step 2: Generate your own dark theme color palette

To customize the dark theme, create folders with the `-night` qualifier for the dark theme to use. For example, you can have specific colors in night mode by creating a folder called `values-night`.

1. Visit the [material.io color picker tool](https://material.io/tools/color/#!/) and create a night-theme color palette. For example, you could base it on a dark blue color.
2. Generate and download the **colors.xml** file.
3. Switch to **Project Files** view to list all the folders in your project.
4. Find the **res** folder and expand it.
5. Create a **res/values-night** resource folder.
6. Add the new **colors.xml** file to the **res/values-night** resource folder.
7. Run your app, still with night mode enabled, and the app should use the new colors you defined for **res/values-night**. Notice that the chips use the new secondary color.

***

### Summary

#### Styles and Themes

* Use themes, styles, and attributes on views to change the appearance of views.
* Themes affect the styling of your whole app and come with many preset values for colors, fonts, and font sizes.
* An attribute applies to the view in which the attribute is set. Use attributes if you have styling that applies to only one view, such as padding, margins, and constraints.
* Styles are groups of attributes that can be used by multiple views. For example, you can have a style for all your content headers, buttons, or text views.
* Themes and styles inherit from their parent theme or style. You can create a hierarchy of styles.
* Attribute values (which are set in views) override styles. Styles override the default style. Styles override themes. Themes override any styling set by the `textAppearance` property.

![](https://developer.android.com/codelabs/kotlin-android-training-styles-and-themes/img/e7851427054b568d.png)

* Define styles in the `styles.xml` resource file using the `<style>` and `<item>` tags.

```xml
<style name="TextAppearance.Subtitle" parent="TextAppearance.Title" >
   <item name="android:textSize">18sp</item>
</style>
```
Using downloadable fonts makes fonts available to users without increasing the size of your APK. To add a downloadable font for a view:

1. Select the view in the **Design** tab, and select **More fonts** from the drop-down menu of the `fontFamily` attribute.
2. In the **Resources** dialog, find a font and select the **Create downloadable font** radio button.
3. Verify that the Android manifest includes a meta-data tag for preloaded fonts.

When the app first requests a font, and the font is not already available, the font provider downloads it from the internet.

#### Material Design, dimens, and colors

**Floating action buttons**

* The floating action button (FAB) floats above all other views. It is usually associated with a primary action the user can take on the screen. You add a click listener to a FAB in the same way as to any other UI element.
* To add a FAB to your layout, use a `CoordinatorLayout` as the top-level view group, then add the FAB to it.
* To scroll content inside a `CoordinatorLayout`, use the `androidx.core.widget.NestedScrollView.`

**Material Design**

* Material Design provides themes and styles that make your app beautiful and easier to use. Material Design provides detailed specs for everything, from how text should be shown, to how to lay out a screen. See [material.io](https://material.io/) for the complete specifications.
* To use Material components, you need to include the Material library in your gradle file. Make sure to use the latest library version.

```kotlin
implementation 'com.google.android.material:material:1.2.0'
```

* Use `?attr` to apply material theme attributes to a view: `style="?attr/textAppearanceHeadline5"`

**Themes and styles**

* Use styles to override theme attributes.
* Use theme overlays to apply a theme to just a view and its children. Apply the theme to the parent view. For example:

```xml
android:theme="@style/ThemeOverlay.MaterialComponents.Dark.ActionBar"
```

Then use the `?attr` notation to apply attributes to a view. For example:

```xml
android:background="?attr/colorPrimaryDark"
```

**Colors**

* The [Color Tool](https://material.io/tools/color) helps you create a Material palette for your app and download the palette as a `color.xml` file.
* Setting a tint on the `ImageView` causes the `ImageView` to be "tinted" to the specified color. For example, you might use `android:tint="?attr/colorOnPrimary"`. The `colorOnPrimary` color is designed to look good on `colorPrimary`.

**Dimensions**

* Use dimension for measurements that apply to your whole app, such as margins, guidelines, or spacing between elements.

#### Design for everyone

**Support RTL Languages**

* In the Android Manifest, set `android:supportsRtl="true"`.
* You can preview RTL in the emulator, and you can use your own language to check screen layouts. On a device or emulator, open **Settings**, and in **Developer Options**, select **Force RTL layout.**
* Replace references to `Left` and `Right` with references to `Start` and `End`.
* Disable mirroring for drawables by deleting `android:autoMirrored="true"`.
* Choose **Refactor > Add RTL support where possible** to let Android Studio do the work for you.
* Use **values-"language code"** folders to store language-specific resources.

**Scan for accessibility**

* In the Play Store, get the [Accessibility Scanner by Google LLC](https://play.google.com/store/apps/details?id=com.google.android.apps.accessibility.auditor) and run it to scan for screen elements to improve.

**Design for TalkBack with content descriptions**

* Install the [Android Accessibility Suite](https://play.google.com/store/apps/details?id=com.google.android.marvin.talkback), by Google, which includes TalkBack.
* Add content descriptions to all UI elements. For example: `android:contentDescription="@string/stage_image_description"`
* For an editable element such as an `EditText`, use an `android:hint` attribute in the XML to provide a hint to the user about what to type.
* Create content groups by wrapping related elements into a view group.
* Create a live region to give users additional feedback with `android:accessibilityLiveRegion`.

**Use Chips to implement a filter**

* _Chips_ are compact elements that represent an attribute, text, entity, or action.
* To create a group of chips, use a `com.google.android.material.chip.ChipGroup`.
* Define the layout for one `com.google.android.material.chip.Chip`.
* If you want your chips to change colors, provide a color state list as a `<selector>` with stateful colors: `<item android:color="?attr/colorPrimaryVariant" android:state_selected="true" />`
* Bind the chips to live data by adding an observer to the data in the view model.
* To display the chips, create an inflator for the chip group: `LayoutInflater.from(chipGroup.context)`
* Create the chips, add a click listener that triggers the desired action, and add the chips to the chip group.

**Support dark mode**

* Use the `DayNight` `AppTheme` to support dark mode.
* You can set dark mode programmatically: `AppCompatDelegate.setDefaultNightMode()`
* Create a **res/values-night** resource folder to provide custom colors and values for dark mode.