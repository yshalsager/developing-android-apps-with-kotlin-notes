## Lesson 05: App Architecture (UI Layer)

> With Architecture Components you'll have the power to design even the 
> most complicated app ideas. Combine ViewModels with LiveData to build 
> this super fun "[Guess it](https://github.com/udacity/andfun-kotlin-guess-it)" game.

- Many Android teams will follow an application architecture, which is a great upon set of design rules.
- Architecture provides the basic bones and structure of your app.
- A well-thought-out architecture can make your code maintainable for years and help your team collaborate.

***

### Tour of the App

- [`MainActivity`](https://github.com/udacity/andfun-kotlin-guess-it/blob/starter-code/app/src/main/java/com/example/android/guesstheword/MainActivity.kt) - This class does very little. It’s just a container for the [NavHostFragment](https://developer.android.com/reference/kotlin/androidx/navigation/fragment/NavHostFragment). The fragments that go in the NavHostFragment do most of the heavy lifting. This is similar to what we did in [lesson 3](https://github.com/yshalsager/developing-android-apps-with-kotlin-notes/blob/master/Lesson%2003%20-%20App%20Navigation.md)
- [`res/navigation/main_navigation.xml`](https://github.com/udacity/andfun-kotlin-guess-it/blob/starter-code/app/src/main/res/navigation/main_navigation.xml) - This is the navigation graph for this app. The navigation flow goes from the TitleFragment to the GameFragment to the ScoreFragment. From the ScoreFragment, you can play the game again by going back to the GameFragment. Note that the action from the GameFragment to the 
  ScoreFragment has a Pop To attribute that removes the game from the backstack. This makes it so that you can never press the back button from the ScoreFragment to go back to a finished game. Instead, you’ll go to the title screen.
- [`TitleFragment`](https://github.com/udacity/andfun-kotlin-guess-it/blob/starter-code/app/src/main/java/com/example/android/guesstheword/screens/title/TitleFragment.kt) - This is a simple fragment showing the title screen. Has a single button that takes you to the GameFragment.
- [`GameFragment`](https://github.com/udacity/andfun-kotlin-guess-it/blob/starter-code/app/src/main/java/com/example/android/guesstheword/screens/game/GameFragment.kt) - This fragment contains all the logic for the GuessIt game itself. It contains:
  - `word` - A variable for the current word to guess.
  - `score` - A variable for the current score.
  - `wordList` - A variable for a mutable list of all the words you need to guess. This list is created and immediately shuffled using `resetList()` so that you get a new order of words every time.
  - `resetList()` - Method that creates and shuffles the list of words.
  - `onSkip()`/`onCorrect()` - Methods for when you press the **Skip**/**Got It** buttons. They modify the score and then go to the next word in your `worldList`.
  - `nextWord()` - A method for moving to the next word to 
    guess. If there are still words in your mutable list of words, remove the current word, and then set `currentWord` to whatever is next in the list. This will finish the game if there are no more words to guess.
  - `gameFinished()` - A method that is called to finish the game. This passes your current score to the ScoreFragment using SafeArgs.
- [`ScoreFragment`](https://github.com/udacity/andfun-kotlin-guess-it/blob/starter-code/app/src/main/java/com/example/android/guesstheword/screens/score/ScoreFragment.kt) - This fragment gets the score passed in from the argument bundle and displays it. There’s also a button for playing again, that takes you back to the GameFragment.

***

### What is Architecture

- A bloated class that does all sorts of different functions is a bad idea.

- **Application architecture** is a way of designing your app's classes and the relationship between them, such that the code base is more organized, performative particular scenarios, and easier to work.

- There's no one right way to architect an application much like there's no empirically right way to architect a house.

- Architecture is heavily dependent on your circumstance needs and tastes.

- There are many different architectural styles for Android apps, each with different strengths and weaknesses. Some of these styles may even overlap. They fit different application needs team sizes team dynamics and more.

- For this lesson, you're gonna be learning about a single, multi-purpose architectural pattern that we're gonna build on for the rest of this course. It's loosely based on an architecture called `MVVM`, which is model view viewmodel. The reason I'm choosing this style is because it's officially endorsed by Google and it leverages the lifecycle classes.

#### Separation of Concerns

Divide your code into classes, each with separate, well-defined responsibilities.

***

### Our App Architecture

- Architecture gives you guidelines to figure out which classes should have what responsibilities in your app.

- We're going to be working with three different classes: the `UI Controller`, the `ViewModel` and `LiveData`.

The first class is the **UI Controller**:

- `UI Controller` is the word that I'm using to describe what activities and fragments are.

- `UI Controller` is responsible for any user interface related tasks like displaying views and capturing user input.

- By design, `UI Controllers` execute all of the draw commands that put our views on the screen.

- Also when an operating system event happens like when a user presses a button it's the `UI Controller` that get notified first.

- You should limit `UI Controller` to only user interface related tasks and take any sort of decision-making power out of `UI Controller` so while `UI Controller` is responsible for drawing a text to you to the screen. It is not responsible for the calculations or processing that decides what actual text to draw.

- While `UI Controller` is what knows that a button has been pressed it's going to immediately pass that information along to the second class in our architecture the `ViewModel`.

**[ViewModel](https://developer.android.com/reference/kotlin/androidx/lifecycle/ViewModel)** will do the actual decision-making.

- The purpose of the `ViewModel` is to hold the specific data needed to display the fragment or activity it's associated with.

- Also, `ViewModels` may do simple calculations and transformations on that data so that it's ready to be displayed by the `UI Controller`.

- The `ViewModel` class will contain instances of a third-class `LiveData`.

**LiveData** classes are crucial for communicating information for the `ViewModel` to `UI Controller` that it should update and redraw the screen.

In our case, the `UI Controllers` will be our three fragments. Let's take game fragment as an example:

- Game fragment will be responsible for drawing the game elements to the screen and knowing when the user presses the buttons nothing more.

- When the buttons are pressed game, fragment will pass that information to the game `ViewModel`.

- The game `ViewModel` will hold data like the score value, the list of words, and the current word to be displayed because that's the data that is needed to know what to display on the screen. It'll also be in charge of simple calculations to decide the current state of the data. For example what the current word is in the list of words and what the current score should be.

***

### ViewModel

- `ViewModel` is an abstract class that you will extend and then implement. It holds your apps UI data and survives configuration changes.

- Instead of having the UI data the fragment move it to your `ViewModel` and have the fragment reference the `ViewModel`.

- `ViewModel` survives configuration changes so while the fragment is destroyed and then remade all of that juicy data that you need to display in the fragment like the score the current word and so on remain in the `ViewModel`.

- If you reconnect your recreated fragment to the same `ViewModel` all of the data is just right there for you.

- Unlike the `onSavedInstanceState` bundle the `ViewModel` has no restrictions on size so you can store lots of data in here without worrying

***

### Adding A ViewModel

1. Add lifecycle-extensions gradle dependency:
   
   In the Module: app `build.gradle` file, add the `lifecycle-extensions` 
   dependency. You can find the most current version of the dependency [here](https://developer.android.com/jetpack/androidx/releases/lifecycle#declaring_dependencies).
   
```groovy
// Lifecycles
implementation 'androidx.lifecycle:lifecycle-extensions:2.0.0'
```

2. Create the `GameViewModel` class, extending `ViewModel`: 
   Create a new file called `GameViewModel.kt` in the `java/com.example.android.guesstheword/game` package. Then in this file, create a class `GameViewModel` that extends `ViewModel`:
   
```kotlin
class GameViewModel : ViewModel()
```

3. Add `init` block and override `onCleared`. Add log statements to both:
   
   Make an `init` block that prints out a log saying “GameViewModel 
   created!”.
   
```kotlin
init {
   Log.i("GameViewModel", "GameViewModel created!")
}
```
   
   Then override `onCleared` so you can track the lifetime of this `ViewModel`. You can use the keyboard shortcut Ctrl + O to do the override. Then add the log statement saying "GameViewModel destroyed!" to `onCleared`.
   
```kotlin
override fun onCleared() {
   super.onCleared()
   Log.i("GameViewModel", "GameViewModel destroyed!")
}
```

4. Create and initialize a `GameViewModel`, using `ViewModelProvider`. Add a log statement:
   
   Back in GameFragment use `lateinit` to create a field for `GameViewModel` called `viewModel`.
   
```kotlin
private lateinit var viewModel: GameViewModel
```
   
   Then in `onCreateView`, request the current `GameViewModel` using the `ViewModelProvider` class:
   
```kotlin
// Get the viewmodel
Log.i("GameFragment", "Called ViewModelProvider")
viewModel = ViewModelProvider(this).get(GameViewModel::class.java)
```

**Note 1: [Sharing UI-related data between destinations with ViewModel](https://developer.android.com/guide/navigation/navigation-programmatic#share_ui-related_data_between_destinations_with_viewmodel) :**
You can save ViewModel state between fragments with navigation by initializing it scoped to a navigation graph.
```kotlin
val viewModel: MyViewModel by navGraphViewModels(R.id.my_graph)
```

**Note 2: `::` Usage in Kotlin**:

`::` is used for Reflection in kotlin:

1. Class Reference `val myClass = MyClass::class`
2. Function Reference `list::isEmpty()`
3. Property Reference `::someVal.isInitialized`
4. Constructor Reference `::MyClass`

***

### What should be moved to the ViewModel?

- `UI Controller` only displays and gets user/OS events. It doesn't make decisions.
- The ViewModel is a stable place to store the data to display in the associated `UI controller`.
- The `Fragment` draws the data on screen and captures input events. It
   should not decide what to display on screen or process what happens during an input event.
- The `ViewModel` never contains references to activities, fragments, or views.

So, in our app, The `score` and `word` field, `wordList` field, and `resetList` and nextWord `methods` should be moved to the `ViewModel`.

1. Move the `word`, `score`, and `wordList` variables to the `GameViewModel`:
   
   In the `GameFragment` find the `word`, `score`, and `wordList` variables, and move them to `GameViewModel`. Make sure you delete the versions of these variables in `GameFragment`.
   
```kotlin
class GameViewModel : ViewModel() {
       // The current word
   var word = ""

   // The current score
   var score = 0

   // The list of words - the front of the list is the next word to guess
   private lateinit var wordList: MutableList<String>
```
   
   Remember, *do not move the binding*! This is because the binding contains references to views.

2. Move methods `resetList`, `nextWord`, `onSkip`, and `onCorrect` to the `GameViewModel`:
   
   In `GameFragment`, find each of the methods: `resetList`, `nextWord`, `onSkip` and `onCorrect`. Move them to `GameViewModel`.
   
   Remove the private modifier `onSkip` and `onCorrect` methods so they can be called from the `GameFragment`.
   
```kotlin
/** Methods for buttons presses **/

fun onSkip() {
   score--
   nextWord()
}

fun onCorrect() {
   score++
   nextWord()
}
```

3. Move the initialization methods to the `GameViewModel`:
   
   Initialization in the `GameFragment` involved calling `resetList` and `nextWord`. Now that they are both in the `GameViewModel`, call them in the `GameViewModel` when it is created.
   
```kotlin
init {
   resetList()
   nextWord()
}
```

4. Update the `onClickListeners` to refer to call methods in the `ViewModel`, and then update the UI:
   
   Now that `onSkip` and `onCorrect` have been moved to the `GameViewModel`, the `OnClickListeners` in the `GameFragment`, refer to method that aren't there. Update the `OnClickListeners` to call the methods in the `GameViewModel`. Then in the `OnClickListeners`, update the score and word texts so that they have the newest data.
   
```kotlin
binding.correctButton.setOnClickListener {
   viewModel.onCorrect()
   updateScoreText()
   updateWordText()
}
binding.skipButton.setOnClickListener {
   viewModel.onSkip()
   updateScoreText()
   updateWordText()
}
```

5. Update the methods to get `word` and `score` from the `viewModel`:
   
   In the `GameViewModel`, remove the private modifier from `word` and `score`.
   
   In the `GameFragment`, update `gameFinished`, `updateWordText` and `updateScoreText` to get the data from the `gameViewModel`.
   
```kotlin
/**
* Called when the game is finished
*/
private fun gameFinished() {
   val action = GameFragmentDirections.actionGameToScore(viewModel.score)
   findNavController(this).navigate(action)
}
```
   
```kotlin
/** Methods for updating the UI **/

private fun updateWordText() {
   binding.wordText.text = viewModel.word

}

private fun updateScoreText() {
   binding.scoreText.text = viewModel.score.toString()
}
```

6. Do final cleanup in the `GameViewModel`:
   
   Once you've copied over the variables and methods, remove any code that refers to the `GameFragment`. In the `GameViewModel`, comment out the reference to `gameFinished` in `nextWord`. You'll deal with this later. You can also clean up the log statements from the prior step.

***

### The Benefits of a Good Architecture

The separation of concerns design principle helps us in two ways:

- The code is more organized, manageable, and debuggable.
- By moving code to the `ViewModel` you protect yourself from having to worry about lifecycle problems and bugs.

Another aspect of this design is:

- The code is very modular.
  `UI Controller` knows about the `ViewModel` but the `ViewModel` doesn't know about the `UI Controller`.
  One of the goals of this architecture is to keep the number of references between classes small. This modularity allows us to swap out a single class for a different implementation without needing to update a bunch of other classes in your app. For example, you could redesign the UI and you wouldn't necessarily need to make changes to the `ViewModel`

- `ViewModel` contains no references to activities fragments or views.
  
  This happens to be helpful for testing.
  Testing on Android differentiates between tests that can be run without emulating the android framework which is in the test folder and tests that are more heavyweight that require emulating the android framework. these tests can be found in the Android test folder.
  
  By designing your app so that these Android classes aren't referenced in the `ViewModel` you can run pure lightweight unit tests that don't depend on the android framework code and therefore run faster and are easier to write.

***

### The Power and Limits of the ViewModel

By adding the `ViewModel` you've actually fixed the rotation issue. The `ViewModel` does preserve data with the configuration changes, but the data is still lost after the app is shut down by the OS.

The [proper way](https://developer.android.com/topic/libraries/architecture/saving-states) to preserve data even if the app is terminated usually involves using a combination of `onSavedInstance` and data persistence.

The second issue is that we have related functions that should be combined in one file, but since we are separating the logic and UI controller it's okay to keep it like this.

Both of these issues boil down to the fact that there's no way to communicate back to the fragment from the view. And since you should not have references to the fragment in your view we need another thing to deal with a situation like this.

***

### [LiveData](https://developer.android.com/topic/libraries/architecture/livedata)

We need a way to communicate for the `ViewModel` back to the UI controller without having the `ViewModel` store references to any views activities or fragments.
For example, we need the `ViewModel` to communicate when data like the score has changed so that the game fragment knows to actually redraw the score.
What would be really great is if we could change the data in the `ViewModel` and then just have the screen magically know when to update itself. Fortunately, live data can do this for us.

**LiveData** is an observable data holder class that is lifecycle-aware.

- `LiveData` holds data this means that it wraps around some data. For example, we'll have live data wrap around the current score.

- `LiveData` is also observable.

The Observer pattern is where you have an object called a Subject. The subject keeps track of a list of other objects known as Observers. Observers watch or observe the subject when the status of the subject changes. it notifies the observers by calling a method in the observer.
In `LiveData`'s case, the subject is the live data object and the observers are the UI controllers. The state change is whenever the data wrapped inside of live data changes.

By setting up this observer relationship you could have the `ViewModel` communicate data changes back to the `UI Controller`.

The `LiveData` object should be val as it will always stay the same. Although the
data is stored within it might change.

#### Adding [LiveData](https://developer.android.com/reference/kotlin/androidx/lifecycle/LiveData) to GameViewModel

1. Wrap `word` and `score` in `MutableLiveData`:
   
   Since `MutableLiveData` is generic  we need to specify the type.
   
```kotlin
val word = MutableLiveData<String>()
val score = MutableLiveData<Int>()
```

2. Initialize `score.value` to 0 because `MutableLiveData` is nullable.
   
```kotlin
init {
   resetList()
   nextWord()
   // Initialize score.value to 0
   score.value = 0
}
```

3. Change references to `score` and `word` to `score.value` and `word.value` and add the required null safety checks:
   
   Check the `onSkip()` and `onCorrect()` methods and change references. 
   Add null safety checks, then call the minus and plus functions, respectively.
   
```kotlin
fun onSkip() {
   // score--
   score.value = (score.value)?.minus(1)
   nextWord()
}

fun onCorrect() {
   // score++
   score.value = (score.value)?.plus(1)
   nextWord()
}
```

4. Set up the observation relationship for the `score` and `word` LiveDatas:
   
   Move over to `GameFragment`. UI controllers are where you'll set up the observation relationship. Get the `LiveData` from your `viewModel` and call the `observe` method. Make sure to pass in `this` and then an observer lambda. Move the code to update the score `TextView` and the word `TextView` to your Observers. Here's the code to set up an observation relationship for the score. This should be in `GameFragment.onCreate`:
   
```kotlin
// Setup the LiveData observation relationship by getting the LiveData from your
// ViewModel and calling observe. Make sure to pass in *this* and then an Observer lambda

/** Setting up LiveData observation relationship **/
viewModel.word.observe(viewLifecycleOwner, { newWord ->
   binding.wordText.text = newWord
})

viewModel.score.observe(viewLifecycleOwner, { newScore ->
   binding.scoreText.text = newScore.toString()
})
```
   
   And now you can remove `updateWordText()` and `updateScoreText()` any references to them completely.

5. Add a null safety check when passing `currentScore` in the `gameFinished` method:
   
   `viewModel.score.value` can possibly be null, so add a null safety check in the `gameFinished` method:
   
```kotlin
val currentScore = viewModel.score.value ?: 0
```

#### Lifecycle-Awareness for LiveData

- LiveData is aware of its associated Ul controller lifecycle state.

- Ul controller off-screen, no updates. (Only updates when it's on the foreground)

- Ul controller back on-screen, get current data. (Immediately send the freshest data when it comes back to foreground)

- New Ul controller observes, get current data. (Immediately send the freshest data when a new Ul controller observes it)

- Ul controller destroyed, it cleans up connection after itself.

#### Adding LiveData Encapsulation to GameViewModel

- **Encapsulation** is the notion of restricting the direct access to objects fields. This way you could expose a public set of methods that modify the private internal fields and you can control the exact ways outside classes can and can't manipulate these internal fields.

- You should try to restrict edit access to your `LiveData`. In this case, only the `ViewModel` should be updating the score and word fields. But for this observation code to work, it's important that you could still get some access to the `LiveData` so it can't be completely private.

- Being able to write and read from `LiveData` is where the distinction between `MutableLiveData` and just plain old `LiveData` comes. `MutableLiveData` is a `LiveData` that can be mutated or modified, whereas `LiveData` on the other hand is `LiveData` that you could read but you cannot call set value on.

- Inside the `ViewModel` we want the `LiveData` to be `MutableLiveData`, but outside the `ViewModel` we want the `MutableLiveData` to only be exposed as `LiveData`. To do that we can use a concept in Kotlin known as a backing property.

- A backing property allows you to return something for a getter other than the exact object.
1. Make internal and external versions of `word` and `score`:
   
   Open up `GameViewModel`. The internal version should be a 
   `MutableLiveData`, has an underscore in front of its name, and be private. The underscore is our convention for marking the variable as the internal version of a variable.
   
   The external version should be a `LiveData` and **not** have an underscore in front of its name.

2. Make a [backing property](https://developer.android.com/kotlin/style-guide#backing-properties) for the external version that returns the internal `MutableLiveData` as a `LiveData`:
   
   [Kotlin automatically makes getters and setters for your fields](https://kotlinlang.org/docs/reference/properties.html#getters-and-setters). If you want to override the getter for `score`, you can do so using a [backing property](https://kotlinlang.org/docs/reference/properties.html#backing-properties). You've actually already defined your backing properties (`_score` and `_word`). Now you can take your public versions of the variables (`score` and `word`) and override `get` to return the backing properties.
   
```kotlin
// Make an internal and external version of the word and score
// The internal version should be a MutableLiveData, have an underscore in front of its' name and be private
// The external version should be a LiveData

// The current word
private val _word = MutableLiveData<String>()
val word: LiveData<String>
   // Make a backing property for the external version that
   // returns the internal MutableLiveData as a LiveData
   get() = _word

// The current score
private val _score = MutableLiveData<Int>()
val score: LiveData<Int>
   get() = _score
```
   
   By making the return type `LiveData` rather than `MutableLiveData`, you've exposed only `score` and `word` as `LiveData`.

3. In the view model, use the internal, mutable versions of `score` and `word`:
   
   In `GameViewModel`, update the code so that you use the mutable versions, `_score` and `_word`, throughout the view model.

***

### Event vs. State

`LiveData` keeps track of data **State**, like button color and score value. But Navigating to another screen is an example of an **Event**.

An **Event** happens once and it's done until it's triggered again. For example:

- A notification
- A sound playing when a button is pressed
- Navigating to a different screen

***

### LiveData And Events

1. Make a properly encapsulated `LiveData` called `eventGameFinish` in the `GameViewModel` that holds a boolean and will represent game end **Event**:
   
```kotlin
// Event which triggers the end of the game
private val _eventGameFinish = MutableLiveData<Boolean>()
val eventGameFinish: LiveData<Boolean>
   get() = _eventGameFinish
```
   
   Also, initialize its value to false.
   
```kotlin
_eventGameFinish.value = false
```

2. Make the function `onGameFinishComplete` which makes the value of `eventGameFinish` false:
   
   This function simply sets the value of `_eventGameFinish` to false. This is to signal that you've handled the game finish event and that you don't need to handle it again. In this specific example, 
   it's a way to say you've done the navigation.
   
```kotlin
/** Methods for completed events **/
fun onGameFinishComplete() {
   _eventGameFinish.value = false
}
```

3. Set `eventGameFinish` to true, to signify that the game is over:
   
   In the `GameViewModel`, find the condition where the `wordList` is empty. If it's empty, the game is over, so signal that by setting the value of `eventGameFinish` to true.
   
```kotlin
if (wordList.isEmpty()) {
   // Set eventGameFinish to true, to signify that the game is over
   _eventGameFinish.value = true
} else {
   _word.value = wordList.removeAt(0)
}
```

4. Add an observer of `eventGameFinish`:
   
   Back in `GameFragment`, add an observer of `eventGameFinish`.
   
   In the observer lambda you should:
   
   1. Make sure that the boolean holding the current value of `eventGameFinished` is true. This means the game has finished.
   
   2. If the game has finished, call `gameFinished`.
   
   3. Tell the view model that you've handled the game finished event by calling `onGameFinishComplete`.
   
```kotlin
// Add an observer of eventGameFinish which, when eventGameFinish is true, calls gameFinished()
// Make sure to call onGameFinishCompete to tell your viewmodel that the game finish event was dealt with

// Sets up event listening to navigate the player when the game is finished
viewModel.eventGameFinish.observe(viewLifecycleOwner, {isFinished ->
   if (isFinished) {
       gameFinished()
       viewModel.onGameFinishComplete()
   }
})
```

***

### Adding a Timer

Where should the timer go? `GameViewModel`

We put the code for tracking and counting downtime in the fragment that would get destroyed whenever you rotated the phone. The logic for counting down a timer should go in the game view model. The only thing the fragment should worry about is updating this textview as the timer ticks.

1. Copy the provided companion object with the timer constants:
   
   In `GameViewModel`, copy the following [companion object](https://kotlinlang.org/docs/reference/java-to-kotlin-interop.html#static-fields) code. This companion object has constants for our timer:
   
```kotlin
companion object {
   // These represent different important times
   // This is when the game is over
   const val DONE = 0L
   // This is the number of milliseconds in a second
   const val ONE_SECOND = 1000L
   // This is the total time of the game
   const val COUNTDOWN_TIME = 60000L
}
```
   
   Feel free to change the `COUNTDOWN_TIME` constant so that the game doesn't last a whole minute. This can be helpful for running the app to check whether it's working.

2. Create a properly encapsulated `LiveData` for the current time called `currentTime`:
   
   Use the same method as you did earlier to encapsulate `LiveData` for `score` and `word`. The type of `currentTime` should be of type Long.
   
```kotlin
// The current time
private val _currentTime = MutableLiveData<Long>()
val currentTime: LiveData<Long>
   get() = _currentTime
```

3. Create a `timer` field of type [CountDownTimer](https://developer.android.com/reference/kotlin/android/os/CountDownTimer) in the `GameViewModel`:
   You don’t need to worry about initializing it yet. Just declare it and ignore the error for now.
   
```kotlin
private val timer: CountDownTimer
```

4. Copy over the `CountDownTimer` code and then update `currentTime` and `eventGameFinish` appropriately as the timer ticks and finishes:
   
   In the `init` of `GameViewModel`, copy the `CountDownTimer` code below:
   
```kotlin
// Creates a timer which triggers the end of the game when it finishes
timer = object : CountDownTimer(COUNTDOWN_TIME, ONE_SECOND) {
   override fun onTick(millisUntilFinished: Long) {
       _currentTime.value = (millisUntilFinished / ONE_SECOND)
   }
   override fun onFinish() {
       _currentTime.value = DONE
       _eventGameFinish.value = true
   }
}
timer.start()
```

5. Update the logic in the `nextWord` function so that it doesn't end the game:
   
   The game should finish when the timer runs out **not** when there are no words left in the list. If there are no words in the list, you should add the words back to the list and re-shuffle the list.
   
   You can do this using `resetList`. Update the code so that it doesn't end the game, but instead calls `resetList`.
   
```kotlin
private fun nextWord() {
   if (wordList.isEmpty()) {
       // Update this logic so that the game doesn't finish;
       // Instead the list is reset and re-shuffled when you run out of words
       resetList()
   }
   _word.value = wordList.removeAt(0)
}
```

6. Cancel the timer in `onCleared`:
   
   To avoid memory leaks, you should always cancel a `CountDownTimer` if you no longer need it. To do that, you can call:
   
```kotlin
// Cancel the timer in onCleared
override fun onCleared() {
   super.onCleared()
   timer.cancel()
}
```

7. Update the UI:
   
   You want the `timerText` on the screen to show the appropriate time. Figure out how to use `LiveData` from the `GameViewModel` to do this - remember, you've done something similar for `score` and `word`. You'll need to convert the Long for the timer into a String. You can use the DateUtils tool to do that.
   
```kotlin
// Setup an observer relationship to update binding.timerText
// You can use DateUtils.formatElapsedTime to correctly format the long to a time string
viewModel.currentTime.observe(viewLifecycleOwner, { newTime ->
   binding.timerText.text = DateUtils.formatElapsedTime(newTime)
})
```

***

### [ViewModelFactory](https://developer.android.com/reference/kotlin/androidx/lifecycle/ViewModelProvider.Factory.html)

A class that knows how to create ViewModels.

There's one challenge with the score fragment and that is that you get the score data passed in from this arguments bundle, you want to display this immediately so this should probably be given to the `ViewModel` when it's initialized.

There are two ways to do this. One is to make a setter for the score variable in the `ViewModel` and then to call it from `onCreateView` within the score fragment. The other is to add a constructor for the `ViewModel`.

##### ViewModel: Adding a constructor

- Create a `ViewModel` that takes in a constructor parameter
- Make a `ViewModel` Factory for `ViewModel`
- Have factory construct `ViewModel` with constructor parameter
- Add `ViewModel` Factory when using `ViewModel` Providers

#### Adding a ViewModelFactory

In this exercise, you'll pass data into a `ViewModel`. You'll create a [view model factory](https://developer.android.com/reference/kotlin/androidx/lifecycle/ViewModelProvider.Factory.html) that allows you to define a custom constructor for a `ViewModel` that gets called when you use `ViewModelProvider`.

1. Create the `ScoreViewModel` class and have it take in an integer constructor parameter called `finalScore`:
   
   Make sure to create the `ScoreViewModel` class file in the same package as the `ScoreFragment`.
   
```kotlin
class ScoreViewModel(finalScore: Int) : ViewModel() {

}
```

2. Copy over ScoreViewModelFactory:
   
   Create `ScoreViewModel` factory in the same package as the `ScoreFragment`. You can use this code later if you ever need to create a view model factory.
   
   Note that the **constructor** of your view model factory should take any parameters you want to pass into your `ScoreViewModel`. In this case, it takes in the final score.
   
   **In the overridden create method, construct and return an instance of `ScoreViewModel`, passing in `finalScore`:**
   
   The create method's purpose is to create and return your view model. 
   So you should construct a new `ScoreViewModel` and return it. You'll also need to deal with the generics, so the statement will be:
   
```kotlin
return ScoreViewModel(finalScore) as T
```
   
   And the full code will be:
   
```kotlin
class ScoreViewModelFactory(private val finalScore: Int) : ViewModelProvider.Factory {
   @Suppress("unchecked_cast")
   override fun <T : ViewModel?> create(modelClass: Class<T>): T {
       if (modelClass.isAssignableFrom(ScoreViewModel::class.java)) {
           return ScoreViewModel(finalScore) as T
       }
       throw IllegalArgumentException("Unknown ViewModel class")
   }
}
```

3. Create and construct a `ScoreViewModelFactory`:
   
   In `ScoreFragment`, create `viewModelFactory` from `ScoreViewModelFactory`.
   
```kotlin
// Get args using by navArgs property delegate
val scoreFragmentArgs by navArgs<ScoreFragmentArgs>()
// Create and construct a ScoreViewModelFactory
viewModelFactory = ScoreViewModelFactory(scoreFragmentArgs.score)
```

4. Create `ScoreViewModel` by using `ViewModelProvider` as usual, except you’ll also pass in your `ScoreViewModelFactory`:
   
```kotlin
viewModel = ViewModelProvider(this, viewModelFactory)
       .get(ScoreViewModel::class.java)
```
   
   By passing in the `ViewModel` factory, you're telling `ViewModelProvider` to use this factory to create `ScoreViewModel`.
   
**Note : Sharing UI-related data between destinations with ViewModel using ViewModelFactory:**
```kotlin
private val winFragmentArgs by navArgs<WinFragmentArgs>()
private val viewModel: WinViewModel by navGraphViewModels(R.id.navigation) {
    WinViewModelFactory(
        winFragmentArgs.winnerPlayer
    )
}
```

5. Add a `LiveData` for the score and the play again event:
   
   Create LiveData for `score` and `eventPlayAgain` using the best practices for encapsulation and event handling that you've learned. Make sure to initialize `score`’s value to the `finalScore` you pass into the view model.
   
```kotlin
private val _eventPlayAgain = MutableLiveData<Boolean>()
val eventPlayAgain: LiveData<Boolean>
   get() = _eventPlayAgain

private val _score = MutableLiveData<Int>()
val score: LiveData<Int>
   get() = _score

init {
   _score.value = finalScore
}
```

6. Convert `ScoreFragment` to properly observe and use `ScoreViewModel` to update the UI:
   
   In `ScoreFragment`, add observers for `score` and `eventPlayAgain` LiveData. Use them to update the UI.
   
```kotlin
// Add observer for score
viewModel.score.observe(viewLifecycleOwner, { newScore ->
   binding.scoreText.text = newScore.toString()
})
```
   
```kotlin
// Navigates back to title when button is pressed
viewModel.eventPlayAgain.observe(viewLifecycleOwner, { playAgain ->
   if (playAgain) {
       findNavController().navigate(ScoreFragmentDirections.actionRestart())
       viewModel.onPlayAgainComplete()
   }
})
```
   
   In `ScoreViewModel`, helper functions to work LiveData Events.
   
```kotlin
fun onPlayAgain() {
   _eventPlayAgain.value = true
}

fun onPlayAgainComplete() {
   _eventPlayAgain.value = false
}
```

***

### Adding ViewModel to Data Binding

In this exercise, you're going to use data binding in the layout XML code to communicate directly with the `ViewModel`. In particular, we're going to tell the `ViewModel` when various buttons are clicked!

1. Add a `GameViewModel` data binding variable to `GameFragment` layout:
   
   In the layout xml file for the `GameFragment` (`game_fragment.xml`), create a `gameViewModel` variable inside the layout.
   
```xml
<data>
   <variable
       name="gameViewModel"
       type="com.example.android.guesstheword.screens.game.GameViewModel" />
</data>
```
   
   Then pass the `GameViewModel` into the data binding:
   
   In the `GameFragment.onCreate`, pass in the view model to the `GameFragmentBinding`.
   
```kotlin
// Pass the GameViewModel into the data binding - then you can remove the
binding.gameViewModel = viewModel
```

2. In the `GameFragment` layout, use the view model variable and data binding to handle clicking:
   
   In XML, you can define an `onClick` attribute for buttons. Using data binding, you can define a [data binding expression](https://developer.android.com/topic/libraries/data-binding/expressions) which is a [Listener binding](https://developer.android.com/topic/libraries/data-binding/expressions#listener_bindings). Essentially this means that you define the `OnClickListener` in the XML. You also have your view model variable available via data binding. So to create an `onClick` attribute that will call `onSkip` in the view model, you can use:
   
```xml
android:onClick="@{() -> gameViewModel.onSkip()}"
```
   
   Now you can (and should) **remove** the `OnClickListener` setup from the `GameFragment`. Everything should work just as before.

3. Add a `ScoreViewModel` data binding variable to `ScoreFragment` layout:
   
   In `score_fragment.xml`, repeat the process you used for `game_fragment.xml` in Step 1.

4. In the `ScoreFragment` layout, use the view model variable and data binding to handle clicking.

5. Pass the `ScoreViewModel` into the data binding and remove `OnClickListener` setup for `playAgainButton`.

Now instead of defining `OnClickListener` code in the fragment, you're using data binding. Run your app and see how all the buttons still work.

***

### Adding LiveData Data Binding

In this step, you'll use [LiveData to automagically update your layout via data binding](https://developer.android.com/topic/libraries/data-binding/architecture#livedata). This will allow you to remove all of your observation lambdas for simple UI updates.

1. Call `binding.setLifecycleOwner` to make the data binding lifecycle aware:
   
   Open `GameFragment`. To make your data binding lifecycle aware and to have it play nicely with `LiveData`, you need to set `binding.setLifecycleOwner` to `this` -- which refers to `GameFragment`. This looks like:
   
```kotlin
// Specify the current activity as the lifecycle owner of the binding. This is used so that
// the binding can observe LiveData updates
binding.lifecycleOwner = this
```

2. For `score_text` and `word_text` use the `LiveData` from `GameViewModel` to set the text attribute:
   
   Open up the `game_fragment` layout. You can use the word `LiveData` to set the text for the `word_text` and `score_text` TextViews. For example, for `word_text`:
   
```xml
android:text="@{gameViewModel.word}"
```
   
   You can also use text formatting.
   
```xml
android:text="@{@string/quote_format(gameViewModel.word)}"
```
   
```xml
android:text="@{@string/score_format(gameViewModel.score)}"
```

3. Remove the score and word observers:
   
   Note that we'll fix the `currentTime` observation in the next exercise. Once you've removed the score and word observers, you should be able to run your app and see that the score and word text still updates.

4. Use the `LiveData` from `ScoreViewModel` to set the `score_text` text attribute:
   
   This is very similar to what you just did with `word_text` and `score_text`. Repeat in `score_fragment.xml`. Note that since score is an Int, you'll need to use `String.valueOf()` in your binding expression to convert the Int to a String.

5. Call `binding.setLifecycleOwner` and remove the score observer.
   
   Repeat steps 2 and 3 in the `ScoreFragment` code. Run your app again to make sure it compiles and runs.

***

### LiveData Map Transformation

One of the easiest ways to do simple data manipulations to `LiveData` such as changing an integer to a string is by using a method called `Transformation.map()`.

The map function takes the output of one `LiveData` which I'll call `LiveData` A and does some sort of conversion on it and then outputs the result to another `LiveData` which I'll call `LiveData` B.

Observers can then observe `LiveData` B if they want. The conversion is defined in a function. So in our case `LiveData` A can output along representing how much time has passed. And then we could do a conversion function on it to format it as a string showing the elapsed time, and then that string would be output from `LiveData` B which in turn would update the game fragment.

In this step you'll use a [Tranformations.map](https://developer.android.com/reference/kotlin/androidx/lifecycle/Transformations#map\(androidx.lifecycle.LiveData,%20androidx.arch.core.util.Function) to convert the current time into a formatted String.

1. In `GameViewModel` create a new `LiveData` called `currentTimeString` and Use Transformation.map to take `currentTime` to a String output from `currentTimeString`:
   
   This will store the String version of `currentTime`.
   
   What you want is to use `DateUtils` to convert the `currentTime` number output into a String. Then we want to emit that from the 
   `currentTimeString` `LiveData`.
   
```kotlin
// Create a new LiveData called currentTimeString.
// Use Transformation.map to take the number output from currentTime, and transform
// it into a String using DateUtils.

// The String version of the current time
val currentTimeString = Transformations.map(currentTime) { time ->
   DateUtils.formatElapsedTime(time)
}
```

2. Set `timer_text` to the value of `currentTimeString`:
   
   In the `game_fragment.xml`, find the `timer_text` view and set the text attribute to the value of `currentTimeString` (not currentTime!) in a binding expression.
   
```xml
android:text="@{gameViewModel.currentTimeString}"
```

3. Delete the observer for `currentTime` from GameFragment:
   
   We don't need it anymore! As always, run your code!

***

### Optional Exercise: Adding the Buzzer

You'll need some logic that determines when the phone should buzz. Where should that code go? `ViewModel`

Is this an Event or a State? `Event`

Let's add a buzzer! Before you start, check the reference documentation on how to use [Vibration on Android](https://developer.android.com/reference/android/os/Vibrator). This will mostly be a self-guided exercise, but we'll give you a few steps to get started.

1. Add the Vibrate permission:
   
   In the `AndroidManifest.xml` file, *above* the `application` tag, add the following tag:
   
```xml
<!-- Add the Vibrate permission -->
<uses-permission android:name="android.permission.VIBRATE" />
```
   
   This provides a [permission](https://developer.android.com/guide/topics/permissions/overview) that lets us vibrate the phone. We will describe Permissions in greater detail later in this course - suffice it to say that without this, our app cannot cause the phone to vibrate on its own.

2. You can also optionally lock the screen to landscape:
   
   While you're in the manifest, you can also optionally lock the phone to landscape mode. That is done by adding the following lines to the `MainActivity` tag:
   
```xml
<activity
   android:name=".MainActivity"
   android:configChanges="keyboardHidden|orientation|screenSize"
   android:screenOrientation="landscape">
```

3. Copy over the different buzz pattern Long array constants:
   
   Vibration is controlled by passing in an array representing the number of milliseconds each interval of buzzing and non-buzzing takes. So the array [0, 200, 100, 300] will wait 0 milliseconds, then buzz for 200ms, then wait 100ms, then buzz fo 300ms. Here are some example buzz patterns you can copy over:
   
```kotlin
private val CORRECT_BUZZ_PATTERN = longArrayOf(100, 100, 100, 100, 100, 100)
private val PANIC_BUZZ_PATTERN = longArrayOf(0, 200)
private val GAME_OVER_BUZZ_PATTERN = longArrayOf(0, 2000)
private val NO_BUZZ_PATTERN = longArrayOf(0)
```
   
   Put these in the `GameViewModel`, above the class.

4. Make an enum called `BuzzType` in `GameViewModel`.
   
   This enum will represent the different types of buzzing that can occur:
   
```kotlin
enum class BuzzType(val pattern: LongArray) {
   CORRECT(CORRECT_BUZZ_PATTERN),
   GAME_OVER(GAME_OVER_BUZZ_PATTERN),
   COUNTDOWN_PANIC(PANIC_BUZZ_PATTERN),
   NO_BUZZ(NO_BUZZ_PATTERN)
}
```

5. Create a properly encapsulated LiveData for a buzz event - its' type should be `BuzzType`.
   
```kotlin
// Event that triggers the phone to buzz using different patterns, determined by BuzzType
private val _eventBuzz = MutableLiveData<BuzzType>()
val eventBuzz: LiveData<BuzzType>
   get() = _eventBuzz
```

6. Set the value of buzz event to the correct `BuzzType` when the buzzer should fire. This should happen when the game is over when the user gets a correct answer, and on each tick when countdown buzzing starts.
   
```kotlin
override fun onTick(millisUntilFinished: Long) {
   _currentTime.value = (millisUntilFinished / ONE_SECOND)
   if (millisUntilFinished / ONE_SECOND <= COUNTDOWN_PANIC_SECONDS) {
   // COUNTDOWN_PANIC buzz
       _eventBuzz.value = BuzzType.COUNTDOWN_PANIC
   }
}

override fun onFinish() {
   _currentTime.value = DONE
   // GAME_OVER buzz
   _eventBuzz.value = BuzzType.GAME_OVER
   _eventGameFinish.value = true
}
```

7. Add a function `onBuzzComplete` for telling the view model when the buzz event has completed.
   
```kotlin
fun onBuzzComplete() {
    _eventBuzz.value = BuzzType.NO_BUZZ
}
```

8. Copy over the `buzz` method.
   
   Given a pattern, this method will actually perform the buzz. It uses the activity to get a system service, so you should put this in your `GameFragment`:
   
```kotlin
private fun buzz(pattern: LongArray) {
   val buzzer = activity?.getSystemService<Vibrator>()

   buzzer?.let {
       if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
           buzzer.vibrate(VibrationEffect.createWaveform(pattern, -1))
       } else {
           //deprecated in API 26
           buzzer.vibrate(pattern, -1)
       }
   }
}
```

9. Created an observer for the buzz event which calls the buzz method with the correct pattern. Remember to call `onBuzzComplete`!
   
```kotlin
// Buzzes when triggered with different buzz events
viewModel.eventBuzz.observe(viewLifecycleOwner, { buzzType ->
   if (buzzType != GameViewModel.BuzzType.NO_BUZZ) {
       buzz(buzzType.pattern)
       viewModel.onBuzzComplete()
   }
})
```

##### Extra: How to force reload the layout?
You can force reload the layout by calling `binding.invalidateAll()`.

***

### Summary

* The Android [app architecture](https://developer.android.com/jetpack/docs/guide) guidelines recommend separating classes that have different responsibilities.
* A _UI controller_ is UI-based class like [`Activity`](https://developer.android.com/reference/android/app/Activity.html) or [`Fragment`](https://developer.android.com/reference/android/app/Fragment.html). UI controllers should only contain logic that handles UI and operating system interactions; they shouldn't contain data to be displayed in the UI. Put that data in a `ViewModel`.
* The [`ViewModel`](https://developer.android.com/reference/android/arch/lifecycle/ViewModel.html) class stores and manages UI-related data. The `ViewModel` class allows data to survive configuration changes such as screen rotations.
* `ViewModel` is one of the recommended [Android Architecture Components](https://developer.android.com/jetpack/#architecture-components).
* `ViewModelProvider.Factory` is an interface you can use to create a `ViewModel` object.

The table below compares UI controllers with the `ViewModel` instances that hold data for them:

|     |     |
| --- | --- |
| **UI controller** | **ViewModel** |
| An example of a UI controller is the `ScoreFragment` that you created in this codelab. | An example of a `ViewModel` is the `ScoreViewModel` that you created in this lesson. |
| Doesn't contain any data to be displayed in the UI. | Contains data that the UI controller displays in the UI. |
| Contains code for displaying data, and user-event code such as click listeners. | Contains code for data processing. |
| Destroyed and re-created during every configuration change. | Destroyed only when the associated UI controller goes away permanently—for an activity, when the activity finishes, or for a fragment, when the fragment is detached. |
| Contains views. | Should never contain references to activities, fragments, or views, because they don't survive configuration changes, but the `ViewModel` does. |
| Contains a reference to the associated `ViewModel`. | Doesn't contain any reference to the associated UI controller. |

#### LiveData

* [`LiveData`](https://developer.android.com/topic/libraries/architecture/livedata) is an observable data holder class that is lifecycle-aware, one of the [Android Architecture Components](https://developer.android.com/topic/libraries/architecture).
* You can use `LiveData` to enable your UI to update automatically when the data updates.
* `LiveData` is observable, which means that an observer like an activity or an fragment can be notified when the data held by the `LiveData` object changes.
* `LiveData` holds data; it is a wrapper that can be used with any data.
* `LiveData` is lifecycle-aware, meaning that it only updates observers that are in an active lifecycle state such as [`STARTED`](https://developer.android.com/reference/android/arch/lifecycle/Lifecycle.State.html#STARTED) or [`RESUMED`](https://developer.android.com/reference/android/arch/lifecycle/Lifecycle.State.html#RESUMED).

##### To add LiveData

* Change the type of the data variables in `ViewModel` to `LiveData` or [`MutableLiveData`](https://developer.android.com/reference/android/arch/lifecycle/MutableLiveData).

`MutableLiveData` is a `LiveData` object whose value can be changed. `MutableLiveData` is a generic class, so you need to specify the type of data that it holds.

* To change the value of the data held by the `LiveData`, use the `setValue()` method on the `LiveData` variable.

##### To encapsulate LiveData

* The `LiveData` inside the `ViewModel` should be editable. Outside the `ViewModel`, the `LiveData` should be readable. This can be implemented using a Kotlin [backing property](https://kotlinlang.org/docs/reference/properties.html#backing-properties).
* A Kotlin backing property allows you to return something from a getter other than the exact object.
* To encapsulate the `LiveData`, use `private` `MutableLiveData` inside the `ViewModel` and return a `LiveData` backing property outside the `ViewModel`.

##### Observable LiveData

* `LiveData` follows an observer pattern. The "observable" is the `LiveData` object, and the observers are the methods in the UI controllers, like fragments. Whenever the data wrapped inside `LiveData` changes, the observer methods in the UI controllers are notified.
* To make the `LiveData` observable, attach an observer object to the `LiveData` reference in the observers (such as activities and fragments) using the [`observe()`](https://developer.android.com/reference/android/arch/lifecycle/LiveData.html#observe%28android.arch.lifecycle.LifecycleOwner,%0Aandroid.arch.lifecycle.Observer%3CT%3E%29) method.
* This `LiveData` observer pattern can be used to communicate from the `ViewModel` to the UI controllers.

#### Data binding with ViewModel and LiveData

* The Data Binding Library works seamlessly with Android Architecture Components like `ViewModel` and `LiveData`.
* The layouts in your app can bind to the data in the Architecture Components, which already help you manage the UI controller's lifecycle and notify about changes in the data.

##### ViewModel data binding

* You can associate a [`ViewModel`](https://developer.android.com/reference/android/arch/lifecycle/ViewModel) with a layout by using data binding.
* `ViewModel` objects hold the UI data. By passing `ViewModel` objects into the data binding, you can automate some of the communication between the views and the `ViewModel` objects.

How to associate a `ViewModel` with a layout:

* In the layout file, add a data-binding variable of the type `ViewModel`.

```xml
<data>
   <variable
       name="gameViewModel"
       type="com.example.android.guesstheword.screens.game.GameViewModel" />
</data>
```

* In the `GameFragment` file, pass the `GameViewModel` into the data binding.  
    
```kotlin
binding.gameViewModel = viewModel
```

##### Listener bindings

* [_Listener bindings_](https://developer.android.com/topic/libraries/data-binding/expressions#listener_bindings) are binding expressions in the layout that run when click events such as `onClick()` are triggered.
* Listener bindings are written as lambda expressions.
* Using listener bindings, you replace the click listeners in the UI controllers with listener bindings in the layout file.
* Data binding creates a listener and sets the listener on the view.

```xml
android:onClick="@{() -> gameViewModel.onSkip()}"
```

#### Adding LiveData to data binding

* [`LiveData`](https://developer.android.com/reference/android/arch/lifecycle/LiveData) objects can be used as a data-binding source to automatically notify the UI about changes in the data.
* You can bind the view directly to the `LiveData` object in the `ViewModel`. When the `LiveData` in the `ViewModel` changes, the views in the layout can be automatically updated, without the observer methods in the UI controllers.

```xml
android:text="@{gameViewModel.word}"
```

* To make the `LiveData` data binding work, set the current activity (the UI controller) as the lifecycle owner of the `binding` variable in the UI controller.

```kotlin
binding.lifecycleOwner = this
```

#### String formatting with data binding

* Using data binding, you can format a string resource with placeholders like `%s` for strings and `%d` for integers.
* To update the `text` attribute of the view, pass in the `LiveData` object as an argument to the formatting string.

```xml
android:text="@{@string/quote_format(gameViewModel.word)}"
```

#### LiveData transformations

**Transforming LiveData**

* Sometimes you want to transform the results of `LiveData`. For example, you might want to format a `Date` string as "hours:mins:seconds," or return the number of items in a list rather than returning the list itself. To perform transformations on `LiveData`, use helper methods in the [`Transformations`](https://developer.android.com/reference/androidx/lifecycle/Transformations.html) class.
* The [`Transformations.map()`](https://developer.android.com/reference/androidx/lifecycle/Transformations.html#map%28androidx.lifecycle.LiveData%3CX%3E,%20androidx.arch.core.util.Function%3CX,%20Y%3E%29) method provides an easy way to perform data manipulations on the `LiveData` and return another `LiveData` object. The recommended practice is to put data-formatting logic that uses the `Transformations` class in the `ViewModel` along with the UI data.

**Displaying the result of a transformation in a** `TextView`

* Make sure the source data is defined as `LiveData` in the `ViewModel`.
* Define a variable, for example `newResult`. Use `Transformation.map()` to perform the transformation and return the result to the variable.

```kotlin
val newResult = Transformations.map(someLiveData) { input ->
   // Do some transformation on the input live data
   // and return the new value
}
```

* Make sure the layout file that contains the `TextView` declares a `<data>` variable for the `ViewModel`.

```xml
<data>
   <variable
       name="MyViewModel"
       type="com.example.android.something.MyViewModel" />
</data>
```

* In the layout file, set the `text` attribute of the `TextView` to the binding of the `newResult` of the `ViewModel`. For example:

```xml
android:text="@{SomeViewModel.newResult}"
```

**Formatting dates**

* The [`DateUtils.formatElapsedTime()`](https://developer.android.com/reference/android/text/format/DateUtils.html#formatElapsedTime%28long%29) utility method takes a `long` number of milliseconds and formats the number to use a `MM:SS` string format.