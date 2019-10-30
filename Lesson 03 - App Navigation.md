## Lesson 03: App Navigation

> Building multiple screens on Android has never been easier with the Navigation library. You'll get to build a fun [trivia app](https://github.com/udacity/andfun-kotlin-android-trivia) using multiple fragments and conditional navigation.

### Navigation Terms

- **Action Bar**: Appears at the top of the application screen. Contains application branding and navigation features such as the overflow menu and the application drawer button.

- **Navigation Drawer**: A menu with a header that slides out from the side of the app.

- **Navigation Graph**: All of the destinations -- the screens that can be navigated to from a single activity are contained in this.

- **Overflow Menu**: A drop down list of items within the Action Bar that can contain navigation destinations.

- **Up Button**: Appears in the action bar and takes us back through previous screens the user has navigated to within the app.

- **The Activity layout**: UI Fragments contain a layout and occupy a place within.

- **context**: Use this property from within a Fragment to get access to string and image resources.

### Fragments

- Android introduced fragments in android 3.0 (API level 11) primarily to support more dynamic and flexible UI designs on large screens such as tablets.

- The activity operates as a frame that contains the UI fragments and can provide UI elements that surround the fragment.

- UI fragments generally operate like a view within the activity's layout but like an activity you must create a subclass of fragment to use it this gives you a layout along with the  convenient place to put UI logic the foundation of a reusable UI component.

- With fragments you can pretty much treat activities as the operating system's entry point to the app since most of your UI ends up being implemented in the fragments but the OS can only open activities.

- Within an activity you tell Android which layout to use by calling `setContentView()` in `onCreate`. The activity then inflates the layout and places it correctly within the activity's layout hierarchy.

- With fragments you manually inflate and return the inflated layout within the `onCreateView()` method which is independent of `onCreate`.

- Another difference since activities inherit from the `context` class but fragments do not. You'll need to use the `context` property within a  fragment to have access to app data typically associated with the context such as string and image resources.

### Navigation between fragments and activities

- You can navigate between different activities and between fragments in an activity.

- As you navigate through activities in Android the previous activities from within the app as well as from previous apps are arranged in a stack that we call the back stack.

- This back stack is ordered based upon the order in which each activity is opened so an application that has a title screen as the first screen followed by a trivia game activity would have the trivia game activity at the top of the stack. Hitting the system back key would pop the trivia game activity off the stack and return to the title screen activity and hitting it again would exit the app finishing the activity and return us to the previous app probably the launcher.

- Fragments can have a similar back stack, the entire stack is contained within the activity all of this is controlled by a class called the **fragment manager**.

- When a fragment is instantiated by the fragment manager the fragment transaction can optionally be added to the fragment back stack if our example trivia app was written using fragments and we were in the game screen the fragment back stack would contain a transaction to return us to the title screen fragment. Hitting the back key would execute this transaction effectively popping the game screen fragment off of the stack and replacing it with the title screen fragment. Hitting it again would pass the back operation through to the activity back stack finishing the activity and exiting the app.

### Choosing navigation pattern

Ultimately you can design your navigation with either an activity containing a series of fragments, a series of activities, or a combination of both techniques.
For this class we're focusing on the single activity multiple fragment model which allows you to visualize your entire apps navigation within a single graph.

### Creating And Adding A Fragment

1. Select File->New->Fragment->Fragment (Blank). Uncheck `include fragment factory methods`, and `include interface callbacks`.

2. As we used `DataBindingUtil.setContentView` in our Activity to get the binding class from a layout, in a fragment we need to call `DataBindingUtil.inflate` in `onCreateView` with the provided layout inflater, the layout resource ID, the provided viewgroup it will be hosted by, and false to not attach it to the viewgroup. Return `binding.root`.
   
   ```kotlin
   //Inflating and Returning the View with DataBindingUtil
   override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?,
                            savedInstanceState: Bundle?): View? {
      val binding = DataBindingUtil.inflate<FragmentTitleBinding>(inflater, R.layout.fragment_title, container, false)
      return binding.root
   }
   ```

3. Add the fragment to the activity layout.
   
   ```xml
               <fragment
                   android:id="@+id/titleFragment"
                   android:name="com.example.android.navigation.TitleFragment"
                   android:layout_width="match_parent"
                   android:layout_height="match_parent" />
   ```

### Principles of Navigation

The navigation component simplifies many navigation tasks more importantly it does so in a consistent way helping developers follow android's principles of navigation.

- The first principle is that apps have a **fixed starting destination** which is the screen the user sees when they launch your app from the launcher. Apps that require login can have one of exceptions to this rule. This destination should also be the last screen the user sees when they return to the launcher after pressing the back button. **(There's always a starting place)**

- The second principle of navigation is the navigation state of your app should be represented with a last in first, out structure. It's that back stack we talked about before. This navigation stack has the start destination at the bottom of the stack and the current destination at the top of the stack. Operations that change the navigation stack should always operate on the top of the navigation stack either by pushing a new destination on to the top of the stack or popping the top most destination off the stack. **(You can always go back)**

- The third principle of navigation involves the way the user gets back to a previous destination. The UP button in the action bar and the system back button both work the same way when navigating within your apps task. The back button will also navigate out of your app and to other apps typically the launcher. correspondingly if the users at the start destination the UP button should not be shown. **(Up goes back, mostly)**

It's important to have principles such as these implemented across a wide range of Android apps to provide a consistent and predictable experience for users.

### Starting Navigation

1. Adding the [Navigation Components](https://developer.android.com/jetpack/androidx/releases/navigation) to the Project:
   
   In `app/build.gradle` file, add the dependencies for navigation fragment ktx and [navigation UI](https://developer.android.com/topic/libraries/architecture/navigation/navigation-ui) ktx. You can see which version is the latest on [this page on developer.android.com](https://developer.android.com/topic/libraries/architecture/adding-components)
   
   ```groovy
   dependencies {
       ...
       // Navigation
       implementation "android.arch.navigation:navigation-fragment-ktx:$version_navigation"     
       implementation "android.arch.navigation:navigation-ui-ktx:$version_navigation"
   }
   ```

2. Adding the Navigation Graph to the Project:
   
   In the Project window, right-click on the res directory and select New > Android resource file. The New Resource dialog appears.
   
   Select Navigation as the resource type, and give it the file name of `navigation`. Make sure it has no qualifiers. Select the `navigation.xml` file in the new navigation directory under `res`, and make sure the design tab is selected.

3. Replace the Title Fragment with the Navigation Host Fragment in the Activity Layout:
   
   ```xml
        <fragment
            android:id="@+id/myNavHostFragment"
            android:name="androidx.navigation.fragment.NavHostFragment"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />
   ```

4. Add `app:navGraph` pointing to the `@navigation/navigation` resource.
   
   ```xml
   app:navGraph="@navigation/navigation"
   ```

5. Set `defaultNavHost` to ture.
   
   ```xml
   app:defaultNavHost="true"
   ```

6. Adding Fragments to the Navigation Graph:
   
   Within the navigation editor, click the add button. A list of fragments and activities will drop down. Add `fragment_title` first, as it is the start destination. (you’ll see that it will automatically be set as the Start Destination for the graph.) Next, add the `fragment_game`.
   
   ```xml
   <navigation xmlns:android="http://schemas.android.com/apk/res/android"
       xmlns:app="http://schemas.android.com/apk/res-auto"
       xmlns:tools="http://schemas.android.com/tools"
       android:id="@+id/nav_root"
       app:startDestination="@+id/titleFragment">
       <fragment
           android:id="@+id/titleFragment"
           android:name="com.example.android.navigation.TitleFragment"
           android:label="@string/android_trivia"
           tools:layout="@layout/fragment_title">
           <action
               android:id="@+id/action_titleFragment_to_gameFragment"
               app:destination="@id/gameFragment" />
       </fragment>
       <fragment
           android:id="@+id/gameFragment"
           android:name="com.example.android.navigation.GameFragment"
           android:label="@string/android_trivia"
           tools:layout="@layout/fragment_game">
       </fragment>
   </navigation>
   ```

7. Connecting Fragments with an Action:
   
   Begin by hovering over the `titleFragment`. You’ll see a circular connection point on the right side of the fragment view. Click on the connection point and drag it to `gameFragment` to add an Action that connects the two fragments.

8. Navigating when the Play Button is Hit:
   
   Return to `onCreateView` in the `TitleFragment` Kotlin code. The binding class has been exposed, so you just call `binding.playButton.setOnClickListener` with a new anonymous function, otherwise known as a lambda. Inside our lambda, use `view.findNavcontroller` to get the navigation controller for our Navigation Host Fragment. Then, use the `navController` to navigate using the `titleFragment` to `gameFragment` action, by calling `navigate(R.id.action_titleFragment_to_gameFragment)`.
   
   ```kotlin
           binding.playButton.setOnClickListener {view: View ->
               Navigation.findNavController(view).navigate(R.id.action_titleFragment_to_gameFragment)
           }
   ```
   
   Or
   
   ```kotlin
   //The complete onClickListener with Navigation
   binding.playButton.setOnClickListener { view: View ->
           view.findNavController().navigate(R.id.action_titleFragment_to_gameFragment)
   }
   ```
   
   Or, use `Navigation` to create the `onClick` listener
   
   ```kotlin
   binding.playButton.setOnClickListener(
           Navigation.createNavigateOnClickListener(R.id.action_titleFragment_to_gameFragment))
   ```

### Back Stack Manipulation

- **PopTo Inclusive**: Pops off everything on the back stack, including the referenced fragment transaction.

- **PopTo Not-Inclusive**: Pops off everything on the back stack until it finds the referenced fragment transaction.

From the navigation editor, select the action for navigating from the `GameFragment` to the `GameOverFragment`. Select `PopTo` `GameFragment` in the attributes pane with the `inclusive` flag to tell the Navigation component to pop fragments off of the fragment back stack until it finds the `GameFragment`, and then pop off the `gameFragment` transaction.

### Adding Support for the Up Button

To add support for the up button, we first need to make sure our Activity has an `ActionBar`.

In `MainActivity` we’ll use the alternate method of finding the controller from the ID of our NavHostFragment using the KTX extension function.

```kotlin
val navController = this.findNavController(R.id.myNavHostFragment)
```

Then link the `NavController` to our `ActionBar`.

```kotlin
NavigationUI.setupActionBarWithNavController(this, navController)
```

Finally, we need to have the Activity handle the `navigateUp` action from our Activity. To do this we override `onSupportNavigateUp`, find the nav controller, and then we call `navigateUp().` 

```kotlin
override fun onSupportNavigateUp(): Boolean {
   val navController = this.findNavController(R.id.myNavHostFragment)
   return navController.navigateUp()
}
```

### Menus

#### Menu Attributes

- **title**: String displayed in the menu.

- **onCreateOptionsMenu**: Where you inflate your menu.

- **onOptionsItemSelected**: Called when a menu item is selected.

- **setHasOptionsMenu**: Tells Android that the Fragment has a menu.

#### Adding a Menu

1. Create new menu resource.
   
   Right click on the `res` folder within the Android project and select New Resource File. We’ll call this one `overflow_menu`, with resource type of `Menu`. Click on the `overflow_menu` within the menu directory, to view our new (empty) menu.

2. Create “`About`” menu item with ID of `aboutFragment` destination.
   
   Drag a menu item from the palette into the component tree. Move to the attributes pane. Set the new item's id to `aboutFragment`, its destination. That's the id you used when adding the About fragment to the navigation graph. For title, we can use `@string/about`. The rest of the attributes should be left as their defaults.

3. Call `setHasOptionsMenu()` in `onCreateView` of `TitleFragment`:
   
   ```kotlin
   override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?,
                            savedInstanceState: Bundle?): View? {
      ...
      setHasOptionsMenu(true)
      return binding.root
   }
   ```

4. Override `onCreateOptionsMenu` and inflate menu resource:
   
   ```kotlin
   override fun onCreateOptionsMenu(menu: Menu?, inflater: MenuInflater?) {
      super.onCreateOptionsMenu(menu, inflater)
      inflater?.inflate(R.menu.overflow_menu, menu)
   }
   ```

5. Override `onOptionsItemSelected` and call `NavigationUI.onNavDestinationSelected`:
   
   ```kotlin
   override fun onOptionsItemSelected(item: MenuItem?): Boolean {
      return NavigationUI.onNavDestinationSelected(item!!,
              view!!.findNavController())
              || super.onOptionsItemSelected(item)
   }
   ```

### Safe Arguments

Safeargs is a Gradle plugin which generates simple object and builder classes for type-safe access to arguments specified for destinations and actions. It’s built on top of the Bundle approach and requires little more code for type safety.

#### What are advantages we get from using safe arguments?

- We get type safety, as navigation generates the action and the argument class from the navigation graph.

- We get argument enforcement, as non-default arguments are required parameters in the action.

#### Adding Safe Arguments

1. In project `build.gradle` add the classpath for the navigation-safe-args-gradle-plugin.
   
   ```kotlin
   classpath "android.arch.navigation:navigation-safe-args-gradle-plugin:$version_navigation"
   ```

2. In `app/build.gradle` at the top and after all of the other plugins, add the apply plugin statement with the androidx navigation safeargs plugin.
   
   ```kotlin
   // Adding the apply plugin statement for safeargs
   apply plugin: 'kotlin-kapt'
   apply plugin: 'androidx.navigation.safeargs'
   ```

3. Switch the Fragment to use generated `NavDirections` when navigating to the other fragments.
   
   ```kotlin
   // Using directions to navigate to the GameWonFragment
   // view.findNavController().navigate(R.id.action_gameFragment_to_gameWonFragment)
   view.findNavController().navigate(GameFragmentDirections.actionGameFragmentToGameWonFragment())
   ```

4. Add the Arguments using the navigation editor.
   
   Go to the navigation editor and select the `GameWon` fragment. Click the little triangle next to arguments to expand the argument section. Add a `numQuestions` and a `numCorrect` argument, both with integer type.

5. Add the parameters from the first Fragment to second Fragment action.
   
   ```kotlin
   // Adding the parameters to the Action
   view.findNavController().navigate(GameFragmentDirections.actionGameFragmentToGameWonFragment(numQuestions, questionIndex))
   ```

6. Fetch the args and expand into a class in `onCreate` within the second Fragment.
   
   ```kotlin
   val args = GameWonFragmentArgs.fromBundle(arguments!!)
   ```

7. Display the arguments using a `Toast`.
   
   ```kotlin
   Toast.makeText(context, "NumCorrect: ${args.numCorrect}, NumQuestions: ${args.numQuestions}", Toast.LENGTH_LONG).show()
   ```

### Intents and Sharing

#### Intent

An Intent is an "intention" to perform an action; a messaging object you can use to request an action from another [app component](https://developer.android.com/guide/components/fundamentals.html#Components), it has two types:

- **Explicit Intents**: Launches an Activity based upon its class name. It's used to call a specific component. When you know which component you want to launch and you do not want to give the user free control over which component to use.

- **Implicit Intents**: Launches an Activity based upon parameters, such as <u>action</u> (The type of thing that the app wants to have done on its behalf), data, and data type (MIME). It's used when you have an idea of what you want to do, but you do not know which component should be launched. Or if you want to give the user an option to choose between a list of components to use.

#### Adding Sharing with an Intent

1. Create new menu resource, which has icon, title and [showAsAction](https://developer.android.com/guide/topics/resources/menu-resource) attribute. 
   
   ```xml
   <menu xmlns:app="http://schemas.android.com/apk/res-auto"
       xmlns:android="http://schemas.android.com/apk/res/android">
       <item
           android:id="@+id/share"
           android:enabled="true"
           android:icon="@drawable/share"
           android:title="@string/share"
           android:visible="true"
           app:showAsAction="ifRoom" />
   </menu>
   ```

2. Add `setHasOptionsMenu(true)` to `onCreateView() `in our the Fragment.
   
   ```kotlin
   // Declaring that our Fragment has a Menu
   setHasOptionsMenu(true)
   ```

3. Create a `getShareIntent` method. Get the args and build the `shareIntent` inside.
   
   ```kotlin
   private fun getShareIntent(): Intent {
       val args = GameWonFragmentArgs.fromBundle(arguments!!)
       val shareIntent = Intent(Intent.ACTION_SEND)
       shareIntent.setType("text/plain").putExtra(
           Intent.EXTRA_TEXT,
           getString(R.string.share_success_text, args.numCorrect, args.numQuestions)
       )
       return shareIntent
   }
   ```
   
   Or even better, using `IntentBuilder`
   
   ```kotlin
   private fun getShareIntent(): Intent {
       val args = GameWonFragmentArgs.fromBundle(arguments!!)
       return ShareCompat.IntentBuilder.from(activity)
           .setText(getString(R.string.share_success_text, args.numCorrect, args.numQuestions)).setType("text/plain")
           .intent
   }
   ```

4. Create a `shareSuccess` method that starts the activity from the share Intent.
   
   ```kotlin
   // Starting an Activity with our new Intent
   private fun shareSuccess() {
      startActivity(getShareIntent())
   }
   ```

5. Override `onCreateOptionsMenu` and inflate the menu xml.
   
   ```kotlin
       override fun onCreateOptionsMenu(menu: Menu?, inflater: MenuInflater?) {
           super.onCreateOptionsMenu(menu, inflater)
           inflater?.inflate(R.menu.winner_menu, menu)
       }
   ```

6. Override `onOptionsIemSelected` to link the menu to the `shareSuccess` action.
   
   ```kotlin
   // Sharing from the Menu
   override fun onOptionsItemSelected(item: MenuItem?): Boolean {
      when (item!!.itemId) {
          R.id.share -> shareSuccess()
      }
      return super.onOptionsItemSelected(item)
   }
   ```

7. Hide the sharing menu item if the sharing intent doesn’t resolve to an Activity.
   
   Get the shareIntent using `getShareIntent()` and call `resolveActivity` using the `packageManger` to make sure our shareIntent resolves to an activity. If the result equals null, which means that it doesn’t resolve, we find our sharing menu item from the inflated menu and set its visibility to false.
   
   ```kotlin
   // Showing the Share Menu Item Dynamically
   override fun onCreateOptionsMenu(menu: Menu?, inflater: MenuInflater?) {
      super.onCreateOptionsMenu(menu, inflater)
      inflater?.inflate(R.menu.winner_menu, menu)
      // check if the activity resolves
      if (null == getShareIntent().resolveActivity(activity!!.packageManager)) {
          // hide the menu item if it doesn't resolve
          menu?.findItem(R.id.share)?.setVisible(false)
      }
   }
   ```
   
   <u>**Alternative:**</u> You can just catch the exception and show a toast.
   
   ```kotlin
           try {
               startActivity(shareIntent)
           } catch (ex: ActivityNotFoundException) {
               Toast.makeText(this, getString(R.string.sharing_not_available),
                       Toast.LENGTH_LONG).show()
           }
   ```

### Adding the Navigation Drawer

1. Add Material Design to `app/build.gradle`.
   
   ```groovy
       implementation "com.google.android.material:material:$version_material"
   ```

2. Add the `RulesFragment` to the navigation graph.
   
   Go to the navigation editor and click the "add" button. Add the rules fragment.

3. Create the [navDrawer](https://developer.android.com/guide/navigation/navigation-ui#add_a_navigation_drawer) menu with the `rulesFragment` and `aboutFragment` menu items.
   
   Create the `navdrawer_menu`. Add two menu items by dragging menu items into the component tree. The first item should have the id of the `RulesFragment`, the rules string and drawable. The second item should have the ID of the `AboutFragment`, the about string and the android drawable.

4. Add the `DrawerLayout` into the `activity_main` layout containing the `LinearLayout` and `navHostFragment`.
   
   ```xml
   <layout xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:app="http://schemas.android.com/apk/res-auto">
   
      <androidx.drawerlayout.widget.DrawerLayout
          android:id="@+id/drawerLayout"
          android:layout_width="match_parent"
          android:layout_height="match_parent"
      >
   ```

5. Add the `NavigationView` at the bottom of the the `DrawerLayout`.
   
   ```xml
   <com.google.android.material.navigation.NavigationView
      android:id="@+id/navView"
      android:layout_width="wrap_content"
      android:layout_height="match_parent"
      android:layout_gravity="start"
      app:menu="@menu/navdrawer_menu" />
   ```

6. In `MainActivity` and add private lateinit vars for `drawerLayout` and `appBarConfiguration`.
   
   ```kotlin
   private lateinit var drawerLayout: DrawerLayout
   private lateinit var appBarConfiguration: AppBarConfiguration
   ```

7. Initialize the `drawerLayout` from the binding variable.
   
   ```kotlin
           val binding = DataBindingUtil.setContentView<ActivityMainBinding>(this, R.layout.activity_main)
           // Initialize drawerLayout var from binding
           drawerLayout = binding.drawerLayout
   ```

8. Add the `DrawerLayout` as the third parameter to `setupActionBarWithNavController`.
   
   ```kotlin
           // Add the DrawerLayout as the second parameter to setupActionBarWithNavController
           // NavigationUI.setupActionBarWithNavController(this, navController)
           NavigationUI.setupActionBarWithNavController(this, navController, drawerLayout)
   ```

9. Create an `appBarConfiguration` with the `navController.graph` and `drawerLayout`.
   
   ```kotlin
   appBarConfiguration = AppBarConfiguration(navController.graph, drawerLayout)
   ```

10. Hook up the navigation UI up to the navigation view.
    
    ```kotlin
    NavigationUI.setupWithNavController(binding.navView, navController)
    ```

11. In `onSupportNavigateUp`, replace `navController.navigateUp` with `NavigationUI.navigateUp` with `appBarConfiguration` as parameter.
    
    ```kotlin
       override fun onSupportNavigateUp(): Boolean {
           val navController = this.findNavController(R.id.myNavHostFragment)
           // Replace navController.navigateUp with NavigationUI.navigateUp with drawerLayout param
           // return navController.navigateUp()
           return NavigationUI.navigateUp(navController, appBarConfiguration)
       }
    ```

12. To make the navigation header looks better, in the `NavigationView` at the bottom of the `DrawerLayout` within the main activity layout file, add the nav header as the `headerLayout`.
    
    ```xml
       app:headerLayout="@layout/nav_header"
    ```

#### How to Navigate

- **App Drawer navigation**: Defaults to popping everything off the backstack except for the start destination.

- **DrawerLayout**: Provides the foundation for the sliding behavior of the navigation drawer.

- **Menu navigation**: Adds to the backstack from the current position.

- **NavigationView**: Material Design container that provides the look, feel, and functionality of the Navigation Drawer.

### Using Navigation Listeners

- Navigation listeners are interfaces that contains a single method that gets called every time we navigate.

- The allow us to react and do something during navigation, or in our case block the draw from coming out after navigating away from the start destination.

- Using navigation listener within onCreate to get called whenever the destination changes.

To prevent the drawer from being swiped anywhere other than the `startDestination`, call `addOnDestinationChangedListener` with a lambda that sets the `DrawerLockMode` depending on what destination we’re navigating to. When the id of our `NavDestination` matches the `startDestination` of our graph, we’ll unlock the `drawerLayout`; otherwise, we’ll lock and close the `drawerLayout`.

```kotlin
// prevent nav gesture if not on start destination
navController.addOnDestinationChangedListener { nc: NavController, nd: NavDestination, args: Bundle? ->
   if (nd.id == nc.graph.startDestination) {
       drawerLayout.setDrawerLockMode(DrawerLayout.LOCK_MODE_UNLOCKED)
   } else {
       drawerLayout.setDrawerLockMode(DrawerLayout.LOCK_MODE_LOCKED_CLOSED)
   }
}
```

### Animation with Navigation

- One more cool thing that we can do with navigation is to apply animated transitions. A restrained and artful use of these transitions can not only make application stand out from the crowbut can help the user understand the
  flow of the application.

- These transitions are controlled by XML animation resources. There are several different kinds of animations in Android but for now we're going to focus on view animations which describe a transformation of a single view or view  group such as the view group contained by our fragment.

- We're going to cover alpha animations and translation animations in this lesson, but the framework also includes rotation and scale animations. The framework includes predefined animation resources and the navigation component also includes some default animations, but it's fun to create our own.

#### Alpha Animations

Alpha describes how transparent something is. At 0% alpha the view would be invisible, while at 100% alpha the view would be opaque.

#### Translate Animations

- Translate animations can use twice the information as alpha animations because we control the change in X and the change in Y.

- Changes in X and y are in percent offset.

- Where 100% X is off-screen right and negative 100% X is off-screen left, while 0% is not offset.

- Changing the Y from negative 100% to 0% will slide the fragment in from off-screen top, while changing the Y from 100% to 0% will slide the fragment in from off-screen bottom.

#### Animation Attributes

- **Enter Transition**: Played for the destination to be navigated to when it is entered.

- **Exit Transition**: Played for the destination to be navigated to when another destination replaces the current one.

- **Pop Enter Transition**: Played when the destination is returned to view from the back stack.

- **Pop Exit Transition**: Played when the current destination is popped off the back stack.

#### Creating Animations

- To create fade in animation resource right click on the anim folder with New->Animation resource file and name it `fade_in`.

- When we open the newly created fade_in.xml file, it will contain an empty animation set:
  
  ```xml
  <set xmlns:android="http://schemas.android.com/apk/res/android">
  </set>
  ```

- We need to add an alpha animation in there, going from 0 to 1 (fading in) and taking the medium animation time. We do that by setting `fromAlpha` to 0, `toAlpha` to 1, and the duration to `@android:integer/config_mediumAnimTime`.
  
  ```xml
  <!-- Fade In Animation -->
  <set xmlns:android="http://schemas.android.com/apk/res/android">
     <alpha
         android:duration="@android:integer/config_mediumAnimTime"
         android:fromAlpha="0.0"
         android:toAlpha="1.0" />
  </set>
  ```

- To create slide in left animation resource we need to do the same thing for the `slide_in_left` animation. We start by creating the file, and get an empty animation set. We then need to add a translate animation in there, going from -100% off screen in the X axis to 0% offscreen in the X axis. Let’s do this one with a short duration. For the Y axis, we’ll be at 0 (onscreen) at the start and end of the animation.
  
  ```xml
  <set xmlns:android="http://schemas.android.com/apk/res/android">
     <translate
         android:fromXDelta="-100%"
         android:toXDelta="0%"
         android:fromYDelta="0%"
         android:toYDelta="0%"
         android:duration="@android:integer/config_shortAnimTime" />
  </set>
  ```

#### Adding transitions to actions

Our first action connects the `TitleFragment` to the `GameFragment`. We’ll have it slide in from the left and out to the right when entering and exiting, and have it slide in from the right and out to the left when pop entering and pop exiting. Edit them from Design editor.

Repeat the same set of animations for the game fragment to the game won fragment, from the `gameWonFragment` to the `gameFragment`, and from the `gameOverFragment` to the `gameFragment`. For the transition to the `gameOverFragment`, we’ll fade in and out instead.
