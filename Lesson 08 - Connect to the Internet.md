## Lesson 08: Connect to the Internet

> Need some live data in your app? In this lesson you'll use Retrofit to communicate with any API service out there. You'll also use Glide to display images from the web for [Mars Real Estate](https://github.com/udacity/andfun-kotlin-mars-real-estate) app!

In this lesson, you use community developed libraries to build the network layer. This greatly simplifies fetching the data and images, and also helps the app conform to some Android best practices, such as loading images on a background thread and caching loaded images. For the asynchronous or non-blocking sections within the code, such as talking to the web services layer, you will modify the app to use Kotlin's coroutines. You will also update the app's user interface if the internet is slow or unavailable to let the user know what's going on.

**What you'll learn**

* What a REST web service is.
* Using the [Retrofit](https://square.github.io/retrofit/) library to connect to a REST web service on the internet and get a response.
* Using the [Moshi](https://github.com/square/moshi) library to parse the JSON response into a data object.

**What you'll do**

* Modify a starter app to make a web service API request and handle the response.
* Implement a network layer for your app using the Retrofit library.
* Parse the JSON response from the web service into your app's live data with the Moshi library.
* Use Retrofit's support for coroutines to simplify the code.

In this lesson, you work with a starter app called MarsRealEstate, which shows properties for sale on Mars. This app connects to a web service to retrieve and display the property data, including details such as the price and whether the property is available for sale or rent. The images representing each property are real-life photos from Mars captured from NASA's Mars rovers.

***

### RESTful Services

The Mars real estate data is stored on a web server, as a [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) web service. Web services use the REST architecture are built using standard web components and protocols.

You make a request to a web service in a standardized way via URIs. The familiar web URL is actually a type of URI, and both are used interchangeably throughout this course. For example, in the app for this lesson, you retrieve all the data from the following server:

https://mars.udacity.com/realestate
https://android-kotlin-fun-mars-server.appspot.com

If you type the following URL in your browser, you get a list of all available real estate properties on Mars!

https://mars.udacity.com/realestate
[https://android-kotlin-fun-mars-server.appspot.com/realestate](https://android-kotlin-fun-mars-server.appspot.com/realestate)

The response from a web service is commonly formatted in [JSON](https://www.json.org/), an interchange format for representing structured data. You learn more about JSON in the next task, but the short explanation is that a JSON object is a collection of key-value pairs, sometimes called a _dictionary_, a _hash map_, or an _associative array_. A collection of JSON objects is a JSON array, and it's the array you get back as a response from a web service.

```json
[
 {
 "price": 450000,
 "id": "424905",
 "type": "buy",
 "img_src": "http://mars.jpl.nasa.gov/msl-raw-images/msss/01000/mcam/1000MR0044631300503690E01_DXXX.jpg"
 },
 {
 "price": 8000000,
 "id": "424906",
 "type": "rent",
 "img_src": "http://mars.jpl.nasa.gov/msl-raw-images/msss/01000/mcam/1000ML0044631300305227E03_DXXX.jpg"
 }
]
```

To get this data into the app, your app needs to establish a network connection and communicate with that server, and then receive and parse the response data into a format the app can use. In this lesson, you use a REST client library called [Retrofit](https://square.github.io/retrofit/) to make this connection.

#### Definitions:

* **URI Query Parameter**: A name and a value separated by an equals sign. For example: `https://mars.udacity.com/realestate?filter=rent&size=2500`
* **URL**: A URL is considered a type of URI.
* **GET, POST, PUT, DELETE**: The basic operations that a RESTful service uses.
* **JSON**: Common format for structured web data.

***

### Libraries

* In this lesson, we will be using some very useful, commonly used community developed libraries with our Android app.
* These libraries are like extensions to the core Android SDK, and they benefit from the collective work of the massive Android community around the world, empowering all the Android developers like you to more easily build better apps.
* Using community built and maintained libraries can be a huge time saver, but it's important to choose these libraries wisely because your app is ultimately responsible for what the code does in these libraries.

Examples of things to look for:
* It doesn't access unexpected APIs and respects privacy.
* It supports the latest Android Target Platform.
* It has GitHub Stars, Forks, Watchers, Community.
* There are closed issues, automated tests.
* There's documentation, samples, test app.

***

### App Walkthrough and Starter Code

The architecture for the MarsRealEstate app has two main modules:

* An overview fragment, which contains a grid of thumbnail property images, built with a `RecyclerView`.
* A detail view fragment, containing information about each property.

We use the Navigation component to both navigate between the two fragments, and to pass the selected property as an argument.

The app has a `ViewModel` for each fragment. For this lesson, you create a layer for the network service, and the `ViewModel` communicates directly with that network layer. This is similar to what you did in previous lessons when the `ViewModel` communicated with the `Room` database.

The overview `ViewModel` is responsible for making the network call to get the Mars real estate information. The detail `ViewModel` holds details for the single piece of Mars real estate that's displayed in the detail fragment. For each `ViewModel`, you use `LiveData` with lifecycle-aware data binding to update the app UI when the data changes.

![c5333bd8ad9bc1d5.png](https://developer.android.com/codelabs/kotlin-android-training-internet-data/img/c5333bd8ad9bc1d5.png)

#### Explore fragments and navigation

* Examine `app/java/MainActivity.kt`. The app uses fragments for both screens, so the only task for the activity is to load the activity's layout.
* Examine `app/res/layout/activity_main.xml`. The activity layout is the host for the two fragments, defined in the navigation file. This layout instantiates a `NavHostFragment` and its associated navigation controller with the `nav_graph` resource.
* Open `app/res/navigation/nav_graph.xml`. Here you can see the navigation relationship between the two fragments. The navigation graph `StartDestination` points to the `overviewFragment`, so the overview fragment is instantiated when the app is launched.

#### Explore Kotlin source files and data binding

1. In the **Project** pane, expand **app** \> **java**. Notice that the MarsRealEstate app has three package folders: `detail`, `network`, and `overview`. These correspond to the three major components of your app: the overview and detail fragments, and the code for the network layer.

![](https://developer.android.com/codelabs/kotlin-android-training-internet-data/img/b0afdfe05eee4fd3.png)

2. Open `app/java/overview/OverviewFragment.kt`. The `OverviewFragment` lazily initializes the `OverviewViewModel`, which means the `OverviewViewModel` is created the first time it is used.
3. Examine the `onCreateView()` method. This method inflates the `fragment_overview` layout using data binding, sets the binding lifecycle owner to itself (`this`), and sets the `viewModel` variable in the `binding` object to it. Because we've set the lifecycle owner, any `LiveData` used in data binding will automatically be observed for any changes, and the UI will be updated accordingly.
4. Open `app/java/overview/OverviewViewModel`. Because the response is a `LiveData` and we've set the lifecycle for the binding variable, any changes to it will update the app UI.
5. Examine the `init` block. When the `ViewModel` is created, it calls the `getMarsRealEstateProperties()` method.
6. Examine the `getMarsRealEstateProperties()` method. In this starter app, this method contains a placeholder response. The goal for this lesson is to update the response `LiveData` within the `ViewModel` using real data you get from the internet.
7. Open `app/res/layout/fragment_overview.xml`. This is the layout for the overview fragment you work with in this lesson, and it includes the data binding for the view model. It imports the `OverviewViewModel` and then binds the response from the `ViewModel` to a `TextView`. Later, you replace the text view with a grid of images in a `RecyclerView`.
8. Compile and run the app. All you see in the current version of this app is the starter response—"Set the Mars API Response here!"

***

### [Connecting to the Internet](https://developer.android.com/training/basics/network-ops/connecting)

* The first step in exploring our Mars app is to use the retrofit library to talk to the Mars web service and display the raw JSON response as a string.
* After this exercise, the app will set the content of the `TextView` to either the return JSON string or a message indicating a connection error.
* Start with the code from the link provided in the instructor notes. First, we need to include the community developer library that we will be using.

#### Step 1: Add Retrofit dependencies to Gradle

1. Open **build.gradle (Module: app)**.
2. In the `dependencies` section, add these lines for the Retrofit libraries:

```groovy
implementation "com.squareup.retrofit2:retrofit:$version_retrofit"
implementation "com.squareup.retrofit2:converter-scalars:$version_retrofit"
```
Notice that the version numbers are defined separately in the project Gradle file. The first dependency is for the Retrofit 2 library itself, and the second dependency is for the Retrofit scalar converter. This converter enables Retrofit to return the JSON result as a `String`. The two libraries work together.

3. Click **Sync Now** to rebuild the project with the new dependencies.

#### Step 2: Add support for Java 8 language features

Many third party libraries including Retrofit2 use Java 8 language features. The Android Gradle plugin provides built-in support for using certain Java 8 language features. To use these built-in features, update the module's `build.gradle` file, as shown below:

```groovy
android {
 ...

 compileOptions {
 sourceCompatibility JavaVersion.VERSION_1_8
 targetCompatibility JavaVersion.VERSION_1_8
 }
 
 kotlinOptions {
 jvmTarget = JavaVersion.VERSION_1_8.toString()
 }
}
```

#### Step 3: Implement `MarsApiService`

Retrofit creates a network API for the app based on the content from the web service. It fetches data from the web service and routes it through a separate converter library that knows how to decode the data and return it in the form of useful objects. Retrofit includes built-in support for popular web data formats such as XML and JSON. Retrofit ultimately creates most of the network layer for you, including critical details such as running the requests on background threads.

The `MarsApiService` class holds the network layer for the app; that is, this is the API that your `ViewModel` will use to communicate with the web service. This is the class where you will implement the Retrofit service API.

1. Open `app/java/network/MarsApiService.kt`. Right now the file contains only one thing: a constant for the base URL for the web service.

```kotlin
private const val BASE_URL = "https://mars.udacity.com/"
```

2. Just below that constant, use a Retrofit builder to create a Retrofit object. Import `retrofit2.Retrofit` and `retrofit2.converter.scalars.ScalarsConverterFactory` when requested.

Retrofit needs at least two things available to it to build a web services API: the base URI for the web service, and a converter factory. The converter tells Retrofit what do with the data it gets back from the web service. In this case, you want Retrofit to fetch a JSON response from the web service, and return it as a `String`. Retrofit has a `ScalarsConverter` that supports strings and other primitive types, so you call `addConverterFactory()` on the builder with an instance of `ScalarsConverterFactory`. Finally, you call `build()` to create the Retrofit object.

```kotlin
// Use Retrofit Builder with ScalarsConverterFactory and BASE_URL
/**
 * Use the Retrofit builder to build a retrofit object using a Moshi converter with our Moshi
 * object pointing to the desired URL
 */
private val retrofit = Retrofit.Builder()
  .addConverterFactory(ScalarsConverterFactory.create())
  .baseUrl(BASE_URL)
  .build()
```

3. Just below the call to the Retrofit builder, define an interface that defines how Retrofit talks to the web server using HTTP requests. Import `retrofit2.http.GET` and `retrofit2.Call` when requested.

Right now the goal is to get the JSON response string from the web service, and you only need one method to do that: `getProperties()`. To tell Retrofit what this method should do, use a `@GET` annotation and specify the path, or endpoint, for that web service method. In this case the endpoint is called `realestate`. When the `getProperties()` method is invoked, Retrofit appends the endpoint `realestate` to the base URL (which you defined in the Retrofit builder), and creates a `Call` object. That `Call` object is used to start the request.

```kotlin
/**
 * A public interface that exposes the [getProperties] method
 */
interface MarsApiService {
 /**
  * Returns a Retrofit callback that delivers a String
  * The @GET annotation indicates that the "realestate" endpoint will be requested with the GET
  * HTTP method
  */
 @GET("realestate")
 fun getProperties():
 Call<String>
}
```

4. Below the `MarsApiService` interface, define a public object called `MarsApi` to initialize the Retrofit service.

The Retrofit `create()` method creates the Retrofit service itself with the `MarsApiService` interface. Because this call is expensive, and the app only needs one Retrofit service instance, you expose the service to the rest of the app using a public object called `MarsApi`, and lazily initialize the Retrofit service there. Now that all the setup is done, each time your app calls `MarsApi.retrofitService`, it will get a singleton Retrofit object that implements `MarsApiService`.

```kotlin
/**
 * A public Api object that exposes the lazy-initialized Retrofit service
 */
object MarsApi {
 val retrofitService : MarsApiService by lazy { retrofit.create(MarsApiService::class.java) }
}
```

#### Step 4: Call the web service in `OverviewViewModel`

1. Open `app/java/overview/OverviewViewModel.kt`. Scroll down to the `getMarsRealEstateProperties()` method.

This is the method where you'll call the Retrofit service and handle the returned JSON string. Right now there's just a placeholder string for the response.

```kotlin
private fun getMarsRealEstateProperties() {
  _response.value = "Set the Mars API Response here!"
}
```

2. Delete the placeholder line that sets the response to "Set the Mars API Response here!"
3. Inside `getMarsRealEstateProperties()`, add the code shown below. Import `retrofit2.Callback` and `com.example.android.marsrealestate.network.MarsApi` when requested.

The `MarsApi.retrofitService.getProperties()` method returns a `Call` object. Then you can call `enqueue()` on that object to start the network request on a background thread.

```kotlin
MarsApi.retrofitService.getProperties().enqueue( 
  object: Callback<String> {
})
```

4. Click on the word `object`, which is underlined in red. Select **Code > Implement methods**. Select both `onResponse()` and `onFailure()` from the list.

```kotlin
override fun onFailure(call: Call<String>, t: Throwable) {
  TODO("not implemented") 
}

override fun onResponse(call: Call<String>, 
  response: Response<String>) {
  TODO("not implemented") 
}
```

5. In `onFailure()`, delete the TODO and set the `_response` to a failure message, as shown below. The `_response` is a `LiveData` string that determines what's shown in the text view. Each state needs to update the `_response` `LiveData`.

The `onFailure()` callback is called when the web service response fails. For this response, set the `_response` status to `"Failure: "` concatenated with the message from the `Throwable` argument.

```kotlin
override fun onFailure(call: Call<String>, t: Throwable) {
  _response.value = "Failure: " + t.message
}
```

6. In `onResponse()`, delete the TODO and set the `_response` to the response body. The `onResponse()` callback is called when the request is successful and the web service returns a response.

```kotlin
override fun onResponse(call: Call<String>, 
  response: Response<String>) {
 _response.value = response.body()
}
```

#### Step 5: Define the internet permission

1. Compile and run the MarsRealEstate app. Note that the app closes immediately with an error.
2. Click the **Logcat** tab in Android Studio and note the error in the log, which starts with a line like this:

```
Process: com.example.android.marsrealestate, PID: 10646
java.lang.SecurityException: Permission denied (missing INTERNET permission?) 
```

The error message tells you that your app might be missing the `INTERNET` permission. Connecting to the internet introduces security concerns, which is why apps do not have internet connectivity by default. You need to explicitly tell Android that the app needs access to the internet.

3. Open `app/manifests/AndroidManifest.xml`. Add this line just before the `<application>` tag:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

4. Compile and run the app again. If everything is working correctly with your internet connection, you see JSON text containing Mars Property data.
5. Tap the **Back** button in your device or emulator to close the app.
6. Put your device or emulator into airplane mode, and then reopen the app from the Recents menu, or restart the app from Android Studio.
7. Turn airplane mode off again.

***

### [Permissions](https://developer.android.com/guide/topics/permissions/overview)

* The purpose of permissions are to protect the privacy of an Android user.
* Android apps must [request](https://developer.android.com/training/permissions/requesting) permissions to access sensitive user data, such as contacts or call logs, as well as use certain system features, such as camera or Internet.
* Each app publicizes the permissions it requires by including users permission tags in the Android manifest file.
* Android libraries can have their own Android manifest file, which can be used to publish permissions that the library requires.
* Android 6.0, Marshmallow [introduced](https://developer.android.com/training/permissions/usage-notes) runtime permission requests for sensitive permissions, such as accessing contacts or the device camera. If your app targets Android 6.0 API level 23 or higher, your app will have to both declare these permissions in the manifest and ask the user to grant the permission at runtime.
* There are also highly sensitive special permissions that the user can only give the app within system settings.
* If the runtime permission isn't granted, or the feature is missing from the manifest, trying to use the feature will result in a security exception.
* It's also possible to [define a custom app permissions](https://developer.android.com/guide/topics/permissions/defining).

***

### Parsing the JSON Response

Now you're getting a JSON response from the Mars web service, which is a great start. But what you really need are Kotlin objects, not a big JSON string. There's a library called [Moshi](https://github.com/square/moshi), which is an Android JSON parser that converts a JSON string into Kotlin objects. Retrofit has a converter that works with Moshi, so it's a great library for your purposes here.

In this task, you use the Moshi library with Retrofit to parse the JSON response from the web service into useful Mars Property Kotlin objects. You change the app so that instead of displaying the raw JSON, the app displays the number of Mars Properties returned.

#### Step 1: Add Moshi library dependencies

1. Open **build.gradle (Module: app)**.
2. In the dependencies section, add the code shown below to include the Moshi dependency. As with Retrofit, `$version_moshi` is defined separately in the project-level Gradle file. This dependency adds support for the Moshi JSON library with Kotlin support.

```kotlin
implementation "com.squareup.moshi:moshi-kotlin:$version_moshi"
```

3. Locate the lines for the Retrofit scalar converter in the `dependencies` block:

```kotlin
implementation "com.squareup.retrofit2:retrofit:$version_retrofit"
implementation "com.squareup.retrofit2:converter-scalars:$version_retrofit"
```

4. Change these lines to use `converter-moshi`:

```kotlin
implementation "com.squareup.retrofit2:converter-moshi:$version_retrofit"
```

5. Click **Sync Now** to rebuild the project with the new dependencies.

**Note:** The project may show compiler errors related to the removed Retrofit scalar dependency. You fix those in the next steps.

#### Step 2: Implement the `MarsProperty` data class

A sample entry of the JSON response you get from the web service looks something like this:

```json
[{"price":450000,
"id":"424906",
"type":"rent",
"img_src":"http://mars.jpl.nasa.gov/msl-raw-images/msss/01000/mcam/1000ML0044631300305227E03_DXXX.jpg"},
...]
```

The JSON response shown above is an array, which is indicated by the square brackets. The array contains JSON objects, which are surrounded by curly braces. Each object contains a set of name-value pairs, separated by colons. Names are surrounded by quotes. Values can be numbers or strings, and strings are also surrounded by quotes. For example, the `price` for this property is $450,000 and the `img_src` is a URL, which is the location of the image file on the server.

In the example above, notice that each Mars property entry has these JSON key and value pairs:

* `price`: the price of the Mars property, as a number.
* `id`: the ID of the property, as a string.
* `type`: either `"rent"` or `"buy"`.
* `img_src`: The image's URL as a string.

Moshi parses this JSON data and converts it into Kotlin objects. To do this, it needs to have a Kotlin data class to store the parsed results, so the next step is to create that class.

1. Open `app/java/network/MarsProperty.kt`.
2. Replace the existing `MarsProperty` class definition with the following code:

```kotlin
import com.squareup.moshi.Json

/**
 * This data class defines a Mars property which includes an ID, the image URL, the type (sale
 * or rental) and the price (monthly if it's a rental).
 * The property names of this data class are used by Moshi to match the names of values in JSON.
 */
data class MarsProperty(
        val id: String,
        // used to map img_src from the JSON to imgSrcUrl in our class
        @Json(name = "img_src") val imgSrcUrl: String,
        val type: String,
        val price: Double) 
```

Notice that each of the variables in the `MarsProperty` class corresponds to a key name in the JSON object. To match the types in the JSON, you use `String` objects for all the values except `price`, which is a `Double`. A `Double` can be used to represent any JSON number.

When Moshi parses the JSON, it matches the keys by name and fills the data objects with appropriate values.

3. Replace the line for the `img_src` key with the line shown below. Import `com.squareup.moshi.Json` when requested.

```kotlin
@Json(name = "img_src") val imgSrcUrl: String,
```

Sometimes the key names in a JSON response can make confusing Kotlin properties, or may not match your coding style—for example, in the JSON file the `img_src` key uses an underscore, whereas Kotlin properties commonly use upper and lowercase letters ("camel case").

To use variable names in your data class that differ from the key names in the JSON response, use the `@Json` annotation. In this example, the name of the variable in the data class is `imgSrcUrl`. The variable is mapped to the JSON attribute `img_src` using `@Json(name = "img_src")`.

#### Step 3: Update `MarsApiService` and `OverviewViewModel`

With the `MarsProperty` data class in place, you can now update the network API and `ViewModel` to include the Moshi data.

1. Open `network/MarsApiService.kt`. You may see missing-class errors for `ScalarsConverterFactory`. This is because of the Retrofit dependency change you made in Step 1. You fix those errors soon.
2. At the top of the file, just before the Retrofit builder, add the following code to create the Moshi instance. Import `com.squareup.moshi.Moshi` and `com.squareup.moshi.kotlin.reflect.KotlinJsonAdapterFactory` when requested.

```kotlin
/**
 * Build the Moshi object that Retrofit will be using, making sure to add the Kotlin adapter for
 * full Kotlin compatibility.
 */
private val moshi = Moshi.Builder()
        .add(KotlinJsonAdapterFactory())
        .build()
```

Similar to what you did with Retrofit, here you create a `moshi` object using the Moshi builder. For Moshi's annotations to work properly with Kotlin, add the `KotlinJsonAdapterFactory`, and then call `build()`.

3. Change the Retrofit builder to use the [MoshiConverterFactory](http://square.github.io/retrofit/2.x/converter-moshi/index.html?retrofit2/converter/moshi/MoshiConverterFactory.html) instead of the `ScalarConverterFactory`, and pass in the `moshi` instance you just created. Import `retrofit2.converter.moshi.MoshiConverterFactory` when requested.

```kotlin
private val retrofit = Retrofit.Builder()
   .addConverterFactory(MoshiConverterFactory.create(moshi))
   .baseUrl(BASE_URL)
   .build()
```

4. Delete the import for `ScalarConverterFactory` as well.

```kotlin
import retrofit2.converter.scalars.ScalarsConverterFactory
```

5. Update the `MarsApiService` interface to have Retrofit return a list of `MarsProperty` objects, instead of returning `Call<String>`.

```kotlin
interface MarsApiService {
   @GET("realestate")
   fun getProperties():
      Call<List<MarsProperty>>
}
```

6. Open `OverviewViewModel.kt`. Scroll down to the call to `getProperties().enqueue()` in the `getMarsRealEstateProperties()` method.
7. Change the argument to `enqueue()` from `Callback<String>` to `Callback<List<MarsProperty>>`. Import `com.example.android.marsrealestate.network.MarsProperty` when requested.

```kotlin
MarsApi.retrofitService.getProperties().enqueue( 
   object: Callback<List<MarsProperty>> {
```

8. In `onFailure()`, change the argument from `Call<String>` to `Call<List<MarsProperty>>`:

```kotlin
override fun onFailure(call: Call<List<MarsProperty>>, t: Throwable) {
```

9. Make the same change to both the arguments to `onResponse()`:

```kotlin
override fun onResponse(call: Call<List<MarsProperty>>, 
   response: Response<List<MarsProperty>>) {
```

10. In the body of `onResponse()`, replace the existing assignment to `_response.value` with the assignment shown below. Because the `response.body()` is now a list of `MarsProperty` objects, the size of that list is the number of properties that were parsed. This response message prints that number of properties:

```kotlin
_response.value = "Success: ${response.body()?.size} Mars properties retrieved"
```

The whole function should be like:

```kotlin
/**
 * Sets the value of the response LiveData to the Mars API status or the successful number of
 * Mars properties retrieved.
 */
private fun getMarsRealEstateProperties() {
    MarsApi.retrofitService.getProperties().enqueue( object: Callback<List<MarsProperty>> {
        override fun onFailure(call: Call<List<MarsProperty>>, t: Throwable) {
            _response.value = "Failure: " + t.message
        }

        override fun onResponse(call: Call<List<MarsProperty>>, response: Response<List<MarsProperty>>) {
            _response.value = "Success: ${response.body()?.size} Mars properties retrieved"
        }
    })
}
```

11. Make sure airplane mode is turned off. Compile and run the app. This time the message should show the number of properties returned from the web service.

**Note:** If your internet connection is not working, make sure that you turned off airplane mode on your device or emulator.

***

### Coroutines and Deferred

Now the Retrofit API service is running, but it uses a callback with two callback methods that you had to implement. One method handles success and another handles failure, and the failure result reports exceptions. Your code would be more efficient and easier to read if you could use coroutines with exception handling, instead of using callbacks. In this task, you convert your network service and the `ViewModel` to use coroutines.

#### Step 1: Update MarsApiService and OverviewViewModel

1. In `MarsApiService`, make `getProperties()` a suspend function. Change `Call<List<MarsProperty>>` to `List<MarsProperty>`. The `getProperties()` method looks like this:

```kotlin
@GET("realestate")
suspend fun getProperties(): List<MarsProperty>
```

2. In the `OverviewViewModel.kt` file, delete all the code inside `getMarsRealEstateProperties()`. You'll use coroutines here instead of the call to `enqueue()` and the `onFailure()` and `onResponse()` callbacks.
3. Inside `getMarsRealEstateProperties()`, launch the coroutine using `viewModelScope.` A [`ViewModelScope`](https://developer.android.com/topic/libraries/architecture/coroutines#viewmodelscope) is the built-in coroutine scope defined for each `ViewModel` in your app. Any coroutine launched in this scope is automatically canceled if the `ViewModel` is cleared.

```kotlin
viewModelScope.launch { 

}
```

4. Inside the launch block, add a `try`/`catch` block to handle exceptions:

```kotlin
try {

} catch (e: Exception) {
  
}
```

5. Inside the `try {}` block, call `getProperties()` on the `retrofitService` object. Calling `getProperties()` from the `MarsApi` service creates and starts the network call on a background thread.

```kotlin
val listResult = MarsApi.retrofitService.getProperties()
```

6. Also inside the `try {}` block, update the response message for the successful response:

```kotlin
_response.value = "Success: ${listResult.size} Mars properties retrieved"
```

7. Inside the `catch {}` block, handle the failure response:

```kotlin
_response.value = "Failure: ${e.message}"
```

The complete `getMarsRealEstateProperties()` method now looks like this:

```kotlin
/**
 * Sets the value of the response LiveData to the Mars API status or the successful number of
 * Mars properties retrieved.
 */
private fun getMarsRealEstateProperties() {
    coroutineScope.launch {
        // Get the Deferred object for our Retrofit request
        var getPropertiesDeferred = MarsApi.retrofitService.getProperties()
        try {
            // Await the completion of our Retrofit request
            var listResult = getPropertiesDeferred.await()
            _response.value = "Success: ${listResult.size} Mars properties retrieved"
        } catch (e: Exception) {
            _response.value = "Failure: ${e.message}"
        }
    }
}
```

***

#### Getting data from the internet Summary

#### REST web services

* A _web service_ is a service on the internet that enables your app to make requests and get data back.
* Common web services use a [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) architecture. Web services that offer REST architecture are known as _RESTful_ services. RESTful web services are built using standard web components and protocols.
* You make a request to a REST web service in a standardized way, via URIs.
* To use a web service, an app must establish a network connection and communicate with the service. Then the app must receive and parse response data into a format the app can use.
* The [Retrofit](https://square.github.io/retrofit/) library is a client library that enables your app to make requests to a REST web service.
* Use converters to tell Retrofit what do with data it sends to the web service and gets back from the web service. For example, the `ScalarsConverter` converter treats the web service data as a `String` or other primitive.
* To enable your app to make connections to the internet, add the `"android.permission.INTERNET"` permission in the Android manifest.

#### JSON parsing

* The response from a web service is often formatted in [JSON](https://www.json.org/), a common interchange format for representing structured data.
* A JSON object is a collection of key-value pairs. This collection is sometimes called a _dictionary_, a _hash map_, or an _associative array_.
* A collection of JSON objects is a JSON array. You get a JSON array as a response from a web service.
* The keys in a key-value pair are surrounded by quotes. The values can be numbers or strings. Strings are also surrounded by quotes.
* The [Moshi](https://github.com/square/moshi) library is an Android JSON parser that converts a JSON string into Kotlin objects. Retrofit has a converter that works with Moshi.
* Moshi matches the keys in a JSON response with properties in a data object that have the same name.
* To use a different property name for a key, annotate that property with the `@Json` annotation and the JSON key name.


***

### Display an Internet Image

Displaying a photo from a web URL might sound straightforward, but there is quite a bit of engineering to make it work well. The image has to be downloaded, buffered, and decoded from its compressed format to an image that Android can use. The image should be cached to an in-memory cache, a storage-based cache, or both. All this has to happen in low-priority background threads so the UI remains responsive. Also, for best network and CPU performance, you might want to fetch and decode more than one image at once. Learning how to effectively load images from the network could be a class in itself.

Fortunately, you can use a community-developed library called [Glide](https://github.com/bumptech/glide) to download, buffer, decode, and cache your images. Glide leaves you with a lot less work than if you had to do all of this from scratch.

Glide basically needs two things:
* The URL of the image you want to load and show.
* An `ImageView` object to display that image.

In this task, you learn how to use Glide to display a single image from the real estate web service. You display the image that represents the first Mars property in the list of properties that the web service returns.

#### Step 1: Add Glide dependency

Open **build.gradle (Module: app)**.In the `dependencies` section, add this line for the Glide library then click **Sync Now** to rebuild the project with the new dependency.:

```kotlin
implementation "com.github.bumptech.glide:glide:$version_glide"
```

#### Step 2: Update the view model

Next you update the `OverviewViewModel` class to include live data for a single Mars property.

1. Open `overview/OverviewViewModel.kt`. Just below the `LiveData` for the `_response`, add both internal (mutable) and external (immutable) live data for a single `MarsProperty` object. Import the `MarsProperty` class (`com.example.android.marsrealestate.network.MarsProperty`) when requested.

```kotlin
private val _property = MutableLiveData<MarsProperty>()

val property: LiveData<MarsProperty>
   get() = _property
```

2. Update `getMarsRealEstateProperties()` to set property to the first `MarsProperty` from `listResult`:

```kotlin
if (listResult.isNotEmpty()) {
    _property.value = listResult[0]
}
```

3. Change the error response to a status value.

```kotlin
_status.value = "Failure: ${e.message}"
```

The complete `try/catch {}` block now looks like this:

```kotlin
private fun getMarsRealEstateProperties() {
    viewModelScope.launch {
        try {
            var listResult = MarsApi.retrofitService.getProperties()
            if (listResult.isNotEmpty()) {
                _property.value = listResult[0]
            }
        } catch (e: Exception) {
            _status.value = "Failure: ${e.message}"
        }
    }
}
```

4. Open the `res/layout/fragment_overview.xml` file. In the `<TextView>` element, change `android:text` to bind to the `imgSrcUrl` component of the `property` `LiveData`:

```xml
android:text="@{viewModel.property.imgSrcUrl}"
```

5. Run the app. The `TextView` displays only the URL of the image in the first Mars property. All you've done so far is set up the view model and the live data for that URL.

#### Step 3: Create a binding adapter and call Glide

Now you have the URL of an image to display, and it's time to start working with Glide to load that image. In this step, you use a [binding adapter](https://developer.android.com/topic/libraries/data-binding/binding-adapters) to take the URL from an XML attribute associated with an `ImageView`, and you use Glide to load the image. Binding adapters are extension methods that sit between a view and bound data to provide custom behavior when the data changes. In this case, the custom behavior is to call Glide to load an image from a URL into an `ImageView`.

1. Open `BindingAdapters.kt`. This file will hold the binding adapters that you use throughout the app.
2. Create a `bindImage()` function that takes an `ImageView` and a `String` as parameters. Annotate the function with `@BindingAdapter`. The `@BindingAdapter` annotation tells data binding that you want this binding adapter executed when an XML item has the `imageUrl` attribute. Import `androidx.databinding.BindingAdapter` and `android.widget.ImageView` when requested.

```kotlin
@BindingAdapter("imageUrl")
fun bindImage(imgView: ImageView, imgUrl: String?) {

}
```

3. Inside the `bindImage()` function, add a `let {}` block for the `imgUrl` argument:

```kotlin
imgUrl?.let { }
```

4. Inside the `let {}` block, add the line shown below to convert the URL string (from the XML) to a `Uri` object. Import `androidx.core.net.toUri` when requested. You want the final `Uri` object to use the HTTPS scheme, because the server you pull the images from requires that scheme. To use the HTTPS scheme, append `buildUpon.scheme("https")` to the `toUri` builder. The `toUri()` method is a Kotlin extension function from the Android KTX core library, so it just looks like it's part of the `String` class.

```kotlin
val imgUri = imgUrl.toUri().buildUpon().scheme("https").build()
```

5. Still inside `let {}`, call `Glide.with()` to load the image from the `Uri` object into the `ImageView`. Import `com.bumptech.glide.Glide` when requested.

```kotlin
Glide.with(imgView.context)
       .load(imgUri)
       .into(imgView)
```

The complete code should be like:

```kotlin
import android.widget.ImageView
import androidx.core.net.toUri
import androidx.databinding.BindingAdapter
import com.bumptech.glide.Glide
import com.bumptech.glide.request.RequestOptions

/**
 * Uses the Glide library to load an image by URL into an [ImageView]
 */
@BindingAdapter("imageUrl")
fun bindImage(imgView: ImageView, imgUrl: String?) {
    imgUrl?.let {
        val imgUri = imgUrl.toUri().buildUpon().scheme("https").build()
        Glide.with(imgView.context)
                .load(imgUri)
                .into(imgView)
    }
}
```

#### Step 4: Update the layout and fragments

Although Glide has loaded the image, there's nothing to see yet. The next step is to update the layout and the fragments with an `ImageView` to display the image.

1. Open `res/layout/gridview_item.xml`. This is the layout resource file you'll use for each item in the `RecyclerView` later in the lesson. You use it temporarily here to show just the single image.
2. Above the `<ImageView>` element, add a `<data>` element for the data binding, and bind to the `OverviewViewModel` class:

```xml
<data>
   <variable
       name="viewModel"
       type="com.example.android.marsrealestate.overview.OverviewViewModel" />
</data>
```

3. Add an `app:imageUrl` attribute to the `ImageView` element to use the new image loading binding adapter:

```xml
app:imageUrl="@{viewModel.property.imgSrcUrl}"
```

4. Open `overview/OverviewFragment.kt`. In the `onCreateView()` method, comment out the line that inflates the `FragmentOverviewBinding` class and assigns it to the binding variable. This is only temporary; you'll go back to it later.

```kotlin
//val binding = FragmentOverviewBinding.inflate(inflater)
```

5. Add a line to inflate the `GridViewItemBinding` class instead. Import `com.example.android.marsrealestate. databinding.GridViewItemBinding` when requested.

**Note:** This change may result in data-binding errors in Android Studio. To resolve those errors, you may need to clean and rebuild the app.

```xml
val binding = GridViewItemBinding.inflate(inflater)
```

6. Run the app. Now you should see a photo of the image from the first `MarsProperty` in the result list.

#### Step 5: Add simple loading and error images

Glide can improve the user experience by showing a placeholder image while loading the image and an error image if the loading fails, for example if the image is missing or corrupt. In this step, you add that functionality to the binding adapter and to the layout.

1. Open `res/drawable/ic_broken_image.xml`, and click the **Preview** tab on the right. For the error image, you're using the broken-image icon that's available in the built-in icon library. This vector drawable uses the `android:tint` attribute to color the icon gray.

![467c213c859e1904.png](https://developer.android.com/codelabs/kotlin-android-training-internet-images/img/467c213c859e1904.png)

2. Open `res/drawable/loading_animation.xml`. This drawable is an animation that's defined with the `<animate-rotate>` tag. The animation rotates an image drawable, `loading_img.xml`, around the center point. (You don't see the animation in the preview.)

![6c1f87d1c932c762.png](https://developer.android.com/codelabs/kotlin-android-training-internet-images/img/6c1f87d1c932c762.png)

3.  Return to the `BindingAdapters.kt` file. In the `bindImage()` method, update the call to `Glide.with()` to call the `apply()` function between `load()` and `into()`. Import `com.bumptech.glide.request.RequestOptions` when requested.

```kotlin
.apply(RequestOptions()
        .placeholder(R.drawable.loading_animation)
        .error(R.drawable.ic_broken_image))
```

This code sets the placeholder loading image to use while loading (the `loading_animation` drawable). The code also sets an image to use if image loading fails (the `broken_image` drawable). The complete `bindImage()` method now looks like this:

```kotlin
@BindingAdapter("imageUrl")
fun bindImage(imgView: ImageView, imgUrl: String?) {
    imgUrl?.let {
        val imgUri = imgUrl.toUri().buildUpon().scheme("https").build()
        Glide.with(imgView.context)
                .load(imgUri)
                .apply(RequestOptions()
                        .placeholder(R.drawable.loading_animation)
                        .error(R.drawable.ic_broken_image))
                .into(imgView)
    }
}
```

4. Run the app. Depending on the speed of your network connection, you might briefly see the loading image as Glide downloads and displays the property image. But you won't see the broken-image icon yet, even if you turn off your network—you fix that in the last part of the lesson.

***

### Display Images in a Grid

Your app now loads property information from the internet. Using data from the first `MarsProperty` list item, you've created a `LiveData` property in the view model, and you've used the image URL from that property data to populate an `ImageView`. But the goal is for your app to display a grid of images, so you want to use a `RecyclerView` with a `GridLayoutManager`.

First, add the Gradle dependency for the `RecyclerView`:

```groovey
implementation "androidx.recyclerview:recyclerview:$version_recyclerview
```

#### Step 1: Update the view model

Right now the view model has a `_property` `LiveData` that holds one `MarsProperty` object—the first one in the response list from the web service. In this step, you change that `LiveData` to hold the entire list of `MarsProperty` objects.

1. Open `overview/OverviewViewModel.kt`.
2. Change the private `_property` variable to `_properties`. Change the type to be a list of `MarsProperty` objects.

```kotlin
private val _properties = MutableLiveData<List<MarsProperty>>()
```

3. Replace the external `property` live data with `properties`. Add the list to the `LiveData` type here as well:

```kotlin
val properties: LiveData<List<MarsProperty>>
    get() = _properties
```

4. Scroll down to the `getMarsRealEstateProperties()` method. Update it to return the entire list instead of just one item.

```kotlin
_properties.value = listResult
```

The entire `try/catch` block now looks like this:

```kotlin
try {
    var listResult = MarsApi.retrofitService.getProperties()
    if (listResult.isNotEmpty()) {
        _property.value = listResult
    }
} catch (e: Exception) {
    _status.value = "Failure: ${e.message}"
}
```

#### Step 2: Update the layouts and fragments

The next step is to change the app's layout and fragments to use a recycler view and a grid layout, rather than the single image view.

1. Open `res/layout/gridview_item.xml`. Change the data binding from the `OverviewViewModel` to `MarsProperty`, and rename the variable to `"property"`.

```xml
<variable
   name="property"
   type="com.example.android.marsrealestate.network.MarsProperty" />
```

2. In the `<ImageView>`, change the `app:imageUrl` attribute to refer to the image URL in the `MarsProperty` object:

```xml
app:imageUrl="@{property.imgSrcUrl}"
```

3. Open `overview/OverviewFragment.kt`. In `onCreateview()`, uncomment the line that inflates `FragmentOverviewBinding`. Delete or comment out the line that inflates `GridViewBinding`. These changes undo the temporary changes you made in the last task.

```kotlin
val binding = FragmentOverviewBinding.inflate(inflater)
// val binding = GridViewItemBinding.inflate(inflater)
```

4. Open `res/layout/fragment_overview.xml`. Delete the entire `<TextView>` element.
5. Add this `<RecyclerView>` element instead, which uses a `GridLayoutManager` and the `grid_view_item` layout for a single item:

```xml
<androidx.recyclerview.widget.RecyclerView
    android:id="@+id/photos_grid"
    android:layout_width="0dp"
    android:layout_height="0dp"
    android:clipToPadding="false"
    android:padding="6dp"
    app:layoutManager="androidx.recyclerview.widget.GridLayoutManager"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintLeft_toLeftOf="parent"
    app:layout_constraintRight_toRightOf="parent"
    app:layout_constraintTop_toTopOf="parent"
    app:spanCount="2"
    tools:itemCount="16"
    tools:listitem="@layout/grid_view_item" />
```

### Step 3: Add the photo grid adapter

Now the `fragment_overview` layout has a `RecyclerView` while the `grid_view_item` layout has a single `ImageView`. In this step, you bind the data to the `RecyclerView` through a `RecyclerView` adapter.

**Note:** This might be a good time to review the `RecyclerView` codelabs!

1. Open `overview/PhotoGridAdapter.kt`.
2. Create the `PhotoGridAdapter` class, with the constructor parameters shown below. The `PhotoGridAdapter` class extends `ListAdapter`, whose constructor needs the list item type, the view holder, and a `DiffUtil.ItemCallback` implementation. Import the `androidx.recyclerview.widget.ListAdapter` and `com.example.android.marsrealestate.network.MarsProperty` classes when requested. In the following steps, you implement the other missing parts of this constructor that are producing errors.

```kotlin
class PhotoGridAdapter : ListAdapter<MarsProperty,
        PhotoGridAdapter.MarsPropertyViewHolder>(DiffCallback) {
}
```

3. Click anywhere in the `PhotoGridAdapter` class and press `Control+i` to implement the `ListAdapter` methods, which are `onCreateViewHolder()` and `onBindViewHolder()`.

```kotlin
override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): PhotoGridAdapter.MarsPropertyViewHolder {
   TODO("not implemented") 
}

override fun onBindViewHolder(holder: PhotoGridAdapter.MarsPropertyViewHolder, position: Int) {
   TODO("not implemented") 
}
```

4. At the end of the `PhotoGridAdapter` class definition, after the methods you just added, add a companion object definition for `DiffCallback`, as shown below. Import `androidx.recyclerview.widget.DiffUtil` when requested.
The `DiffCallback` object extends `DiffUtil.ItemCallback` with the type of object you want to compare—`MarsProperty`.

```kotlin
companion object DiffCallback : DiffUtil.ItemCallback<MarsProperty>() {}
```

5. Press `Control+i` to implement the comparator methods for this object, which are `areItemsTheSame()` and `areContentsTheSame()`.

```kotlin
override fun areItemsTheSame(oldItem: MarsProperty, newItem: MarsProperty): Boolean {
   TODO("not implemented") 
}

override fun areContentsTheSame(oldItem: MarsProperty, newItem: MarsProperty): Boolean {
   TODO("not implemented") }
```

6. For the `areItemsTheSame()` method, remove the TODO. Use Kotlin's referential equality operator (`===`), which returns `true` if the object references for `oldItem` and `newItem` are the same.

```kotlin
override fun areItemsTheSame(oldItem: MarsProperty, 
                  newItem: MarsProperty): Boolean {
   return oldItem === newItem
}
```

7. For `areContentsTheSame()`, use the standard equality operator on just the ID of `oldItem` and `newItem`.

```kotlin
override fun areContentsTheSame(oldItem: MarsProperty, 
                  newItem: MarsProperty): Boolean {
   return oldItem.id == newItem.id
}
```

8. Still inside the `PhotoGridAdapter` class, below the companion object, add an inner class definition for `MarsPropertyViewHolder`, which extends `RecyclerView.ViewHolder`. Import `androidx.recyclerview.widget.RecyclerView` and `com.example.android.marsrealestate.databinding.GridViewItemBinding` when requested.
You need the `GridViewItemBinding` variable for binding the `MarsProperty` to the layout, so pass the variable into the `MarsPropertyViewHolder`. Because the base `ViewHolder` class requires a view in its constructor, you pass it the binding root view.

```kotlin
class MarsPropertyViewHolder(private var binding: 
                   GridViewItemBinding):
       RecyclerView.ViewHolder(binding.root) {
}
```

9. In `MarsPropertyViewHolder`, create a `bind()` method that takes a `MarsProperty` object as an argument and sets `binding.property` to that object. Call `executePendingBindings()` after setting the property, which causes the update to execute immediately.

```kotlin
fun bind(marsProperty: MarsProperty) {
   binding.property = marsProperty
   binding.executePendingBindings()
}
```

**Note:** This change may result in data-binding errors in Android Studio. To resolve those errors, you may need to clean and rebuild the app.

10. In `onCreateViewHolder()`, remove the TODO and add the line shown below. Import `android.view.LayoutInflater` when requested.
The `onCreateViewHolder()` method needs to return a new `MarsPropertyViewHolder`, created by inflating the `GridViewItemBinding` and using the `LayoutInflater` from your parent `ViewGroup` context.

```kotlin
return MarsPropertyViewHolder(GridViewItemBinding.inflate(
    LayoutInflater.from(parent.context)))
```

11. In the `onBindViewHolder()` method, remove the TODO and add the lines shown below. Here you call `getItem()` to get the `MarsProperty` object associated with the current `RecyclerView` position, and then pass that property to the `bind()` method in the `MarsPropertyViewHolder`.

```kotlin
val marsProperty = getItem(position)
holder.bind(marsProperty)
```

The full code of the adapter should be:

```kotlin
import android.view.LayoutInflater
import android.view.ViewGroup
import androidx.recyclerview.widget.DiffUtil
import androidx.recyclerview.widget.ListAdapter
import androidx.recyclerview.widget.RecyclerView
import com.example.android.marsrealestate.databinding.GridViewItemBinding
import com.example.android.marsrealestate.network.MarsProperty

/**
 * This class implements a [RecyclerView] [ListAdapter] which uses Data Binding to present [List]
 * data, including computing diffs between lists.
 */
class PhotoGridAdapter : ListAdapter<MarsProperty, PhotoGridAdapter.MarsPropertyViewHolder>(DiffCallback) {

    /**
     * The MarsPropertyViewHolder constructor takes the binding variable from the associated
     * GridViewItem, which nicely gives it access to the full [MarsProperty] information.
     */
    class MarsPropertyViewHolder(private var binding: GridViewItemBinding):
            RecyclerView.ViewHolder(binding.root) {
        fun bind(marsProperty: MarsProperty) {
            binding.property = marsProperty
            // This is important, because it forces the data binding to execute immediately,
            // which allows the RecyclerView to make the correct view size measurements
            binding.executePendingBindings()
        }
    }

    /**
     * Allows the RecyclerView to determine which items have changed when the [List] of [MarsProperty]
     * has been updated.
     */
    companion object DiffCallback : DiffUtil.ItemCallback<MarsProperty>() {
        override fun areItemsTheSame(oldItem: MarsProperty, newItem: MarsProperty): Boolean {
            return oldItem === newItem
        }

        override fun areContentsTheSame(oldItem: MarsProperty, newItem: MarsProperty): Boolean {
            return oldItem.id == newItem.id
        }
    }

    /**
     * Create new [RecyclerView] item views (invoked by the layout manager)
     */
    override fun onCreateViewHolder(parent: ViewGroup,
                                    viewType: Int): MarsPropertyViewHolder {
        return MarsPropertyViewHolder(GridViewItemBinding.inflate(LayoutInflater.from(parent.context)))
    }

    /**
     * Replaces the contents of a view (invoked by the layout manager)
     */
    override fun onBindViewHolder(holder: MarsPropertyViewHolder, position: Int) {
        val marsProperty = getItem(position)
        holder.bind(marsProperty)
    }
}
```

#### Step 4: Add the binding adapter and connect the parts

Finally, use a `BindingAdapter` to initialize the `PhotoGridAdapter` with the list of `MarsProperty` objects. Using a `BindingAdapter` to set the `RecyclerView` data causes data binding to automatically observe the `LiveData` for the list of `MarsProperty` objects. Then the binding adapter is called automatically when the `MarsProperty` list changes.

1. Open `BindingAdapters.kt`. At the end of the file, add a `bindRecyclerView()` method that takes a `RecyclerView` and a list of `MarsProperty` objects as arguments. Annotate that method with a `@BindingAdapter`. Import `androidx.recyclerview.widget.RecyclerView` and `com.example.android.marsrealestate.network.MarsProperty` when requested.

```kotlin
@BindingAdapter("listData")
fun bindRecyclerView(recyclerView: RecyclerView, 
    data: List<MarsProperty>?) {
}
```

2. Inside the `bindRecyclerView()` function, cast `recyclerView.adapter` to `PhotoGridAdapter`, and call `adapter.submitList()` with the data. This tells the `RecyclerView` when a new list is available. Import `com.example.android.marsrealestate.overview.PhotoGridAdapter` when requested.

```kotlin
val adapter = recyclerView.adapter as PhotoGridAdapter
adapter.submitList(data)
```

The `bindRecyclerView` full code should be:

```kotlin
import com.example.android.marsrealestate.network.MarsProperty
import com.example.android.marsrealestate.overview.PhotoGridAdapter

/**
 * When there is no Mars property data (data is null), hide the [RecyclerView], otherwise show it.
 */
@BindingAdapter("listData")
fun bindRecyclerView(recyclerView: RecyclerView, data: List<MarsProperty>?) {
    val adapter = recyclerView.adapter as PhotoGridAdapter
    adapter.submitList(data)
}
```

3. Open `res/layout/fragment_overview.xml`. Add the `app:listData` attribute to the `RecyclerView` element and set it to `viewmodel.properties` using data binding.

```xml
app:listData="@{viewModel.properties}"
```

4. Open `overview/OverviewFragment.kt`. In `onCreateView()`, just before the call to `setHasOptionsMenu()`, initialize the `RecyclerView` adapter in `binding.photosGrid` to a new `PhotoGridAdapter` object.

```kotlin
binding.photosGrid.adapter = PhotoGridAdapter()
```

5. In `fragment_overview`, add an attribute to the `RecyclerView` to set `clipToPadding` to `false` to tell the `RecyclerView` not to clip the inner contents to the padding, which makes it draw the scrolling view in the padded area.

```xml
android:clipToPadding="false"
```

6. Run the app. You should see a grid of `MarsProperty` images. As you scroll to see new images, the app shows the loading-progress icon before displaying the image itself. If you turn on airplane mode, images that have not yet loaded appear as broken-image icons.

***

### Error Handling with RecyclerView

The MarsRealEstate app displays the broken-image icon when an image cannot be fetched. But when there's no network, the app shows a blank screen.

This isn't a great user experience. In this task, you add basic error handling, to give the user a better idea of what's happening. If the internet isn't available, the app will show the connection-error icon. While the app is fetching the `MarsProperty` list, the app will show the loading animation.

#### Step 1: Add status to the view model

To start, you create a `LiveData` in the view model to represent the status of the web request. There are three states to consider—loading, success, and failure. The loading state happens while you're waiting for data in the call to `await()`.

1. Open `overview/OverviewViewModel.kt`. At the top of the file (after the imports, before the class definition), add an `enum` to represent all the available statuses:

```kotlin
enum class MarsApiStatus { LOADING, ERROR, DONE }
```

2. Rename both the internal and external `_response` live data definitions throughout the `OverviewViewModel` class to `_status`. Because you added support for the `_properties` `LiveData` earlier in this codelab, the complete web service response has been unused. You need a `LiveData` here to keep track of the current status, so you can just rename the existing variables. Also, change the types from `String` to `MarsApiStatus.`

```kotlin
private val _status = MutableLiveData<MarsApiStatus>()

val status: LiveData<MarsApiStatus>
get() = _status
```

3. Scroll down to the `getMarsRealEstateProperties()` method and update `_response` to `_status` here as well. Change the `"Success"` string to the `MarsApiStatus.DONE` state, and the `"Failure"` string to `MarsApiStatus.ERROR`.
4. Set the status to `MarsApiStatus.LOADING` before the `try {}` block. This is the initial status while the coroutine is running and you're waiting for data. The complete `try/catch {}` block now looks like this:

```kotlin
_status.value = MarsApiStatus.LOADING
try {
   _properties.value = MarsApi.retrofitService.getProperties()
   _status.value = MarsApiStatus.DONE
} catch (e: Exception) {
   _status.value = MarsApiStatus.ERROR
}
```

5. After the error state in the `catch {}` block, set the `_properties` `LiveData` to an empty list. This clears the `RecyclerView`.

```kotlin
} catch (e: Exception) {
   _status.value = MarsApiStatus.ERROR
   _properties.value = ArrayList()
}
```

#### Step 2: Add a binding adapter for the status `ImageView`

Now you have a status in the view model, but it's just a set of states. How do you make it appear in the app itself? In this step, you use an `ImageView`, connected to data binding, to display icons for the loading and error states. When the app is in the loading state or the error state, the `ImageView` should be visible. When the app is done loading, the `ImageView` should be invisible.

1. Open `BindingAdapters.kt`. Add a new binding adapter called `bindStatus()` that takes an `ImageView` and a `MarsApiStatus` value as arguments. Import `com.example.android.marsrealestate.overview.MarsApiStatus` when requested.

```kotlin
@BindingAdapter("marsApiStatus")
fun bindStatus(statusImageView: ImageView, 
          status: MarsApiStatus?) {
}
```

2. Add a `when {}` inside the `bindStatus()` method to switch between the different statuses.

```kotlin
when (status) {}
```

3. Inside the `when {}`, add a case for the loading state (`MarsApiStatus.LOADING`). For this state, set the `ImageView` to visible, and assign it the loading animation. This is the same animation drawable you used for Glide in the previous task. Import `android.view.View` when requested.

```kotlin
when (status) {
   MarsApiStatus.LOADING -> {
      statusImageView.visibility = View.VISIBLE
      statusImageView.setImageResource(R.drawable.loading_animation)
   }
}
```

4. Add a case for the error state, which is `MarsApiStatus.ERROR`. Similarly to what you did for the `LOADING` state, set the status `ImageView` to visible and reuse the connection-error drawable.

```kotlin
MarsApiStatus.ERROR -> {
   statusImageView.visibility = View.VISIBLE
   statusImageView.setImageResource(R.drawable.ic_connection_error)
}
```

5. Add a case for the done state, which is `MarsApiStatus.DONE`. Here you have a successful response, so turn off the visibility of the status `ImageView` to hide it.

```kotlin
MarsApiStatus.DONE -> {
   statusImageView.visibility = View.GONE
}
```

#### Step 3: Add the status ImageView to the layout

1. Open `res/layout/fragment_overview.xml`. Below the `RecyclerView` element, inside the `ConstraintLayout`, add the `ImageView` shown below.

This `ImageView` has the same constraints as the `RecyclerView`. However, the width and height use `wrap_content` to center the image rather than stretch the image to fill the view. Also notice the `app:marsApiStatus` attribute, which has the view call your `BindingAdapter` when the status property in the view model changes.

```xml
<ImageView
    android:id="@+id/status_image"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintLeft_toLeftOf="parent"
    app:layout_constraintRight_toRightOf="parent"
    app:layout_constraintTop_toTopOf="parent"
    app:marsApiStatus="@{viewModel.status}" />
```

2. Turn on airplane mode in your emulator or device to simulate a missing network connection. Compile and run the app, and notice that the error image appears.

3. Tap the Back button to close the app, and turn off airplane mode. Use the recents screen to return the app. Depending on the speed of your network connection, you may see an extremely brief loading spinner when the app queries the web service before the images begin to load.

***

### Parcel and Parcelables

* In Android, parceling is a way of sharing objects between different processes by flattening an object into a string of data called a parcel.

* A complex object can be stored into the parcel and then recreated from the parcel by implementing the parcelable interface, and they become parcelable objects. Each value in the object is written in sequence to the parcel. The object is recreated by reading data from the parcel in the same order it was written to populate data in a new object.

* Using a parcel to share an object between processes, is functionally similar to using XML or JSON to share data between web services and clients.

* A [bundle is a parcelable object](https://developer.android.com/guide/components/activities/parcelables-and-bundles) that contains a key value store of parcelable objects. We use bundles as the argument property in fragments, primarily because of the way the Android lifecycle works. Activities will be destroyed with a `SaveInstanceState` if the app is killed when running in the background. All of the information in the `SaveInstanceState` has to be from parcelables since the state is used to recreate objects when the app gets restarted and is therefore a new process. When an activity is recreated in this state, the fragment manager needs to be able to recreate all of the fragments.

* Since they are parcelable, bundles can be stored in the `SaveInstanceState`, allowing fragments to preserve their arguments when the process is destroyed and the fragment is recreated.


#### How to make and object [parcelable](https://developer.android.com/reference/android/os/Parcelable)?

First, add the parcelable interface.

```kotlin
data class MarsProperty(
        val id: String,
        // used to map img_src from the JSON to imgSrcUrl in our class
        @Json(name = "img_src") val imgSrcUrl: String,
        val type: String,
        val price: Double) : Parcelable
```

Android Studio can implement a version of this for us. Click on `MarsProperty` and use Alt Enter in Linux or Windows or Option Enter on Mac to show quick fixes and select add parcelable implementation.

```kotlin
constructor(parcel: Parcel) : this(
        parcel.readString()!!,
        parcel.readString()!!,
        parcel.readString()!!,
        parcel.readDouble()) {
}

override fun writeToParcel(parcel: Parcel, flags: Int) {
    parcel.writeString(id)
    parcel.writeString(imgSrcUrl)
    parcel.writeString(type)
    parcel.writeDouble(price)
}

override fun describeContents(): Int {
    return 0
}

companion object CREATOR : Parcelable.Creator<MarsProperty> {
    override fun createFromParcel(parcel: Parcel): MarsProperty {
        return MarsProperty(parcel)
    }

    override fun newArray(size: Int): Array<MarsProperty?> {
        return arrayOfNulls(size)
    }
}
```

* Android Studio has made an implementation of the parcelable.creator object for us in the creator Kotlin companion object.
* This has two methods. One that creates our `MarsProperty` from a [parcel](https://developer.android.com/reference/kotlin/android/os/Parcel), and another that creates a new empty array of `MarsProperty` with a given size.
* To create `MarsProperty` from a parcel, it's added a constructor in `MarsProperty` that takes a parcel and calls its main constructor with the values read in sequence from the parcel. This is then called from the `createFromParcel` call in the creator object.
* The described contents method needs to return zero. It's used by Android to share active file descriptors between processes, and that's not our use case.
* The `writeToParcel` method writes all of the objects properties out to the parcel.
Order is important in these method.
* The order of the parcel operations and `writeToParcel`, and `createFromParcel` in our case,
the constructor must match.
* If you add any properties to the `MarsProperty` class, you have to remember to update both of the methods. If you don't, you'll either create an incorrect object or more likely, crash your application. It's easy to make a mistake with these as your code changes over time.

#### @Parcelize with Kotlin Parcelize Plugin
Fortunately, there's an easier way to get help with parcelable, the Kotlin Parcelize extensions.
In our app Gradle, add the following plugin.

```groovy
apply plugin: 'kotlin-parcelize'
```

Back in `MarsProperty`, we can undo all that stuff Android Studio did for us to fill out the parcelable interface and replace it with the add parcelize annotation.

It's doing exactly the same thing for us. But if we add or remove properties, we don't have to worry about modifying the parcel functions. It will keep the same efficiency of writing parcel functions by hand,
but there is no chance that will mess it up and create incorrect objects in crashes, tooling for the win.

```kotlin
import kotlinx.parcelize.Parcelize

@Parcelize
data class MarsProperty(
        val id: String,
        // used to map img_src from the JSON to imgSrcUrl in our class
        @Json(name = "img_src") val imgSrcUrl: String,
        val type: String,
        val price: Double) : Parcelable
```

**In a nutshell:**
* Parceling: Android's way of turning an object into a stream of data.
* Read/write to/from Parcels: Parcelable objects must do this in the same order.
* `@Parcelize` Turns a Kotlin data object with simple and Parcelable types into a Parcelable object.
* Argument Bundle: Used in recreating Fragments after the app process has been destroyed.

***

### Add the Detail Screen

In this next step, we're going to add a `DetailFragment` to display the details of a specific property.
The `DetailFragment` will show a larger image, the property type, whether it's for rental or sale, and the price.

This fragment is launched when the user taps an image in the overview grid. To accomplish this, you need to add an `onClick` listener to the `RecyclerView` grid items, and then navigate to the new fragment. You navigate by triggering a `LiveData` change in the `ViewModel`, as you've done throughout these lessons. You also use the Navigation component's Safe Args plugin to pass the selected `MarsProperty` information from the overview fragment to the detail fragment.

#### Step 1: Create the detail view model and update detail layout

Similar to the process you used for the overview view model and fragments, you now need to implement the view model and layout files for the detail fragment.

1. Open `detail/DetailViewModel.kt`. Just as network-related Kotlin files are contained in the `network` folder and overview files in `overview`, the `detail` folder contains the files associated with the detail view. Notice that `DetailViewModel` class (empty right now) takes a `marsProperty` as a parameter in the constructor.

```kotlin
class DetailViewModel(marsProperty: MarsProperty,
                     app: Application) : AndroidViewModel(app) {
}
```

2. Inside the class definition, add `LiveData` for the selected Mars property, to expose that information to the detail view. Follow the usual pattern of creating a `MutableLiveData` to hold the `MarsProperty` itself, and then expose an immutable public `LiveData` property. Import `androidx.lifecycle.LiveData` and import `androidx.lifecycle.MutableLiveData` when requested.

```kotlin
private val _selectedProperty = MutableLiveData<MarsProperty>()
val selectedProperty: LiveData<MarsProperty>
   get() = _selectedProperty
```

3. Create an `init {}` block and set the value of the selected Mars property with the `MarsProperty` object from the constructor.

```kotlin
init {
    _selectedProperty.value = marsProperty
}
```

4. Open `res/layout/fragment_detail.xml` and look at it in the design view. This is the layout file for the detail fragment. It contains an `ImageView` for the large photo, a `TextView` for the property type (rental or sale) and a `TextView` for the price. Notice that the constraint layout is wrapped with a `ScrollView` so it will automatically scroll if the view gets too large for the display, for example when the user views it in landscape mode.
5. Go to the **Text** tab for the layout. At the top of the layout, just before the `<ScrollView>` element, add a `<data>` element to associate the detail view model with the layout.

```xml
<data>
   <variable
       name="viewModel"
       type="com.example.android.marsrealestate.detail.DetailViewModel" />
</data>
```

6. Add the `app:imageUrl` attribute to the `ImageView` element. Set it to the `imgSrcUrl` from the view model's selected property. The binding adapter that loads an image using Glide will automatically be used here as well, because that adapter watches all `app:imageUrl` attributes.

```xml
 app:imageUrl="@{viewModel.selectedProperty.imgSrcUrl}"
```

7. Bind the `property_type_text` `TextView` to `viewModel.selectedProperty.type` and the `price_value_text` `TextView` to `viewModel.selectedProperty.price`, converted to a string value:

```xml
 android:text="@{viewModel.selectedProperty.type}"

 android:text="@{String.valueOf(viewModel.selectedProperty.price)}"
```

#### Step 2: Define navigation in the overview view model

When the user taps a photo in the overview model, it should trigger navigation to a fragment that shows details about the clicked item.

1. Open `overview/OverviewViewModel.kt`. Add a `_navigateToSelectedProperty` `MutableLiveData` property and expose it with an immutable `LiveData`.  When this `LiveData` changes to non-null, the navigation is triggered. (Soon you'll add the code to observe this variable and trigger the navigation.)

```kotlin
private val _navigateToSelectedProperty = MutableLiveData<MarsProperty>()
val navigateToSelectedProperty: LiveData<MarsProperty>
   get() = _navigateToSelectedProperty
```

2. At the end of the class, add a `displayPropertyDetails()` method that sets `navigateToSelectedProperty` to the selected Mars property.

```kotlin
fun displayPropertyDetails(marsProperty: MarsProperty) {
   _navigateToSelectedProperty.value = marsProperty
}
```

3. Add a `displayPropertyDetailsComplete()` method that nulls the value of `_navigateToSelectedProperty`. You need this to mark the navigation state to complete, and to avoid the navigation being triggered again when the user returns from the detail view.

```kotlin
fun displayPropertyDetailsComplete() {
   _navigateToSelectedProperty.value = null
}
```

#### Step 3: Set up the click listeners in the grid adapter and fragment

1. Open `overview/PhotoGridAdapter.kt`. At the end of the class, create a custom `OnClickListener` class that takes a lambda with a `marsProperty` parameter. Inside the class, define an `onClick()` function that is set to the lambda parameter.

```kotlin
class OnClickListener(val clickListener: (marsProperty:MarsProperty) -> Unit) {
     fun onClick(marsProperty:MarsProperty) = clickListener(marsProperty)
}
```

2. Scroll up to the class definition for the `PhotoGridAdapter`, and add a private `OnClickListener` property to the constructor.

```kotlin
class PhotoGridAdapter( private val onClickListener: OnClickListener ) :
       ListAdapter<MarsProperty,              
           PhotoGridAdapter.MarsPropertyViewHolder>(DiffCallback) {
```

3. Make a photo clickable by adding the `onClickListener` to the grid item in the `onBindviewHolder()` method. Define the click listener in between the calls to `getItem() and bind()`.

```kotlin
override fun onBindViewHolder(holder: MarsPropertyViewHolder, position: Int) {
   val marsProperty = getItem(position)
   holder.itemView.setOnClickListener {
       onClickListener.onClick(marsProperty)
   }
   holder.bind(marsProperty)
}
```

4. Open `overview/OverviewFragment.kt`. In the `onCreateView()` method, replace the line that initializes the `binding.photosGrid.adapter` property with the line shown below. This code adds the `PhotoGridAdapter.onClickListener` object to the `PhotoGridAdapter` constructor, and calls `viewModel.displayPropertyDetails()` with the passed-in `MarsProperty` object. This triggers the `LiveData` in the view model for the navigation.

```kotlin
binding.photosGrid.adapter = PhotoGridAdapter(PhotoGridAdapter.OnClickListener {
   viewModel.displayPropertyDetails(it)
})
```

#### Step 4: Modify the navigation graph and make `MarsProperty` parcelable

When a user taps a photo in the overview grid, the app should navigate to the detail fragment and pass through the details of the selected Mars property so the detail view can display that information.

![](https://developer.android.com/codelabs/kotlin-android-training-internet-filtering/img/3fbf1636128d623f.png)

Right now you have a click listener from `PhotoGridAdapter` to handle the tap, and a way to trigger the navigation from the view model. But you don't yet have a `MarsProperty` object being passed to the detail fragment. For that you use Safe Args from the navigation component.

1. Open `res/navigation/nav_graph.xml`. Click the **Text** tab to view the XML code for the navigation graph.
2. Inside the `<fragment>` element for the detail fragment, add the `<argument>` element shown below. This argument, called `selectedProperty`, has the type `MarsProperty`.

```xml
<argument
   android:name="selectedProperty"
   app:argType="com.example.android.marsrealestate.network.MarsProperty"
   />
```

3. Compile the app. Navigation gives you an error because the `MarsProperty` isn't _parcelable_. The [`Parcelable`](https://developer.android.com/reference/android/os/Parcelable) interface enables objects to be serialized, so that the objects' data can be passed around between fragments or activities. In this case, for the data inside the `MarsProperty` object to be passed to the detail fragment via Safe Args, `MarsProperty` must implement the `Parcelable` interface. The good news is that Kotlin provides an easy shortcut for implementing that interface.
4. Open `network/MarsProperty.kt`. Add the `@Parcelize` annotation to the class definition. Import `kotlinx.parcelize.Parcelize` when requested. The `@Parcelize` annotation uses the Kotlin Android extensions to automatically implement the methods in the `Parcelable` interface for this class. You don't have to do anything else!

```kotlin
@Parcelize
data class MarsProperty (
```

5. Change the class definition of `MarsProperty` to extend `Parcelable`. Import `android.os.Parcelable` when requested. The `MarsProperty` class definition now looks like this:

```kotlin
@Parcelize
data class MarsProperty (
       val id: String,
       @Json(name = "img_src") val imgSrcUrl: String,
       val type: String,
       val price: Double) : Parcelable {
```

#### Step 5: Connect the fragments

You're still not navigating—the actual navigation happens in the fragments. In this step, you add the last bits for implementing navigation between the overview and detail fragments.

1. Open `overview/OverviewFragment.kt`. In `onCreateView()`, below the lines that initialize the photo grid adapter, add the lines shown below to observe the `navigatedToSelectedProperty` from the overview view model. Import `androidx.lifecycle.Observer` and import `androidx.navigation.fragment.findNavController` when requested.  

The observer tests whether `MarsProperty`—the `it` in the lambda—is not null, and if so, it gets the navigation controller from the fragment with `findNavController()`. Call `displayPropertyDetailsComplete()` to tell the view model to reset the `LiveData` to the null state, so you won't accidentally trigger navigation again when the app returns back to the `OverviewFragment`.

```kotlin
viewModel.navigateToSelectedProperty.observe(this, Observer {
   if ( null != it ) {   
      this.findNavController().navigate(
              OverviewFragmentDirections.actionShowDetail(it))             
      viewModel.displayPropertyDetailsComplete()
   }
})
```

2. Open `detail/DetailFragment.kt`. Add this line just below setting the property `binding.lifecycleOwner` in the `onCreateView()` method. This line gets the selected `MarsProperty` object from the Safe Args. Notice the use of Kotlin's not-null assertion operator (`!!`). If the `selectedProperty` isn't there, something terrible has happened and you actually want the code to throw a null pointer. (In production code, you should handle that error in some way.)

```kotlin
val marsProperty = DetailFragmentArgs.fromBundle(arguments!!).selectedProperty
```

3. Add this line next, to get a new `DetailViewModelFactory`. You'll use the `DetailViewModelFactory` to get an instance of the `DetailViewModel`. The starter app includes an implementation of `DetailViewModelFactory`, so all you have to do here is initialize it.

```kotlin
val viewModelFactory = DetailViewModelFactory(marsProperty, application)
```

4. Finally, add this line to get a `DetailViewModel` from the factory and to connect all the parts.

```kotlin
binding.viewModel = ViewModelProvider(this, viewModelFactory).get(DetailViewModel::class.java)
```

5. Compile and run the app, and tap on any Mars property photo. The detail fragment appears for that property's details. Tap the Back button to return to the overview page.

#### Step 6: Update `MarsProperty` to include the type

The `MarsProperty` class defines the data structure for each property provided by the web service. In a previous codelab, you used the [Moshi](https://github.com/square/moshi) library to parse the raw JSON response from the Mars web service into individual `MarsProperty` data objects.

In this step, you add some logic to the `MarsProperty` class to indicate whether a property is for rent or not (that is, whether the type is the string `"rent"` or `"buy"`). You'll use this logic in more than one place, so it's better to have it here in the data class than to replicate it.

Open `network/MarsProperty.kt`. Add a body to the `MarsProperty` class definition, and add a custom getter for `isRental` that returns `true` if the object is of type `"rent"`.

```kotlin
data class MarsProperty(
       val id: String,
       @Json(name = "img_src") val imgSrcUrl: String,
       val type: String,
       val price: Double)  {
   val isRental
       get() = type == "rent"
}
```

#### Step 7: Create a more useful detail page

Right now the detail page shows only the same Mars photo you're used to seeing on the overview page. The `MarsProperty` class also has a property type (rent or buy) and a property price. The detail screen should include both these values, and it would be helpful if the rental properties indicated that the price was a per-month value. You use `LiveData` transformations in the view model to implement both those things.

1. Open `res/values/strings.xml`. The starter code includes string resources, shown below, to help you build the strings for the detail view. For the price, you'll use either the `display_price_monthly_rental` resource or the `display_price` resource, depending on the property type.

```xml
<string name="type_rent">Rent</string>
<string name="type_sale">Sale</string>
<string name="display_type">For %s</string>
<string name="display_price_monthly_rental">$%,.0f/month</string>
<string name="display_price">$%,.0f</string>
```

2. Open `detail/DetailViewModel.kt`. At the bottom of the class, add the code shown below.  Import `androidx.lifecycle.Transformations` if requested. This transformation tests whether the selected property is a rental, using the same test from the first task. If the property is a rental, the transformation chooses the appropriate string from the resources with a Kotlin `when {}` switch. Both of these strings need a number at the end, so you concatenate the `property.price` afterwards.

```kotlin
val displayPropertyPrice = Transformations.map(selectedProperty) {
   app.applicationContext.getString(
           when (it.isRental) {
               true -> R.string.display_price_monthly_rental
               false -> R.string.display_price
           }, it.price)
}
```

3. Import the generated `R` class to gain access to the string resources in the project.

```kotlin
import com.example.android.marsrealestate.R
```

4. After the `displayPropertyPrice` transformation, add the code shown below. This transformation concatenates multiple string resources, based on whether the property type is a rental.

```kotlin
val displayPropertyType = Transformations.map(selectedProperty) {
   app.applicationContext.getString(R.string.display_type,
           app.applicationContext.getString(
                   when (it.isRental) {
                       true -> R.string.type_rent
                       false -> R.string.type_sale
                   }))
}
```

5. Open `res/layout/fragment_detail.xml`. There's just one more thing to do, and that is to bind the new strings (which you created with the `LiveData` transformations) to the detail view. To do that, you set the value of the text field for the property type text to `viewModel.displayPropertyType`, and the text field for the price value text to `viewModel.displayPropertyPrice`.

```xml
<TextView
   android:id="@+id/property_type_text"
...
android:text="@{viewModel.displayPropertyType}"
...
   tools:text="To Rent" />

<TextView
   android:id="@+id/price_value_text"
...
android:text="@{viewModel.displayPropertyPrice}"
...
   tools:text="$100,000" />
```

6. Compile and run the app. Now all the property data appears on the detail page, nicely formatted.

***

### Add a Filter
Currently your app displays _all_ the Mars properties in the overview grid. If a user were shopping for a rental property on Mars, having the icons to indicate which of the available properties are for sale would be useful, but there are still a lot of properties to scroll through on the page. In this task, you add an options menu to the overview fragment that enables the user to show only rentals, only for-sale properties, or show all.

One way you could accomplish this task is to test the type for each `MarsProperty` in the overview grid and only display the matching properties. The actual Mars web service, however, has a query parameter or option (called `filter`) that enables you to get only properties of either type `rent` or type `buy`. You could use this filter query with the `realestate` web service URL in a browser like this:

```
https://android-kotlin-fun-mars-server.appspot.com/realestate?filter=buy
```

In this task, you modify the `MarsApiService` class to add a query option to the web service request with Retrofit. Then you hook up the options menu to re-download all the Mars property data using that query option. Because the response you get from the web service only contains the properties you're interested in, you don't need to change the view display logic for the overview grid at all.

#### Step 1: Update the Mars API service

To change the request, you need to revisit the `MarsApiService` class that you implemented in the first task in this lesson. You modify the class to provide a filtering API.

1. Open `network/MarsApiService.kt`. Just below the imports, create an `enum` called `MarsApiFilter` to define constants that match the query values the web service expects.

```kotlin
enum class MarsApiFilter(val value: String) {
   SHOW_RENT("rent"),
   SHOW_BUY("buy"),
   SHOW_ALL("all") }
```

2. Modify the `getProperties()` method to take string input for the filter query, and annotate that input with `@Query("filter")`, as shown below. Import `retrofit2.http.Query` when prompted. The `@Query` annotation tells the `getProperties()` method (and thus Retrofit) to make the web service request with the filter option. Each time `getProperties()` is called, the request URL includes the `?filter=type` portion, which directs the web service to respond with results that match that query.

```kotlin
suspend fun getProperties(@Query("filter") type: String): List<MarsProperty>  
```

#### Step 2: Update the overview view model

You request data from the `MarsApiService` in the `getMarsRealEstateProperties()` method in `OverviewViewModel`. Now you need to update that request to take the filter argument.

1. Open `overview/OverviewViewModel.kt`. You will see errors in Android Studio due to the changes you made in the previous step. Add `MarsApiFilter` (the enum of possible filter values) as a parameter to the `getMarsRealEstateProperties()` call.  Import `com.example.android.marsrealestate.network.MarsApiFilter` when requested.

```kotlin
private fun getMarsRealEstateProperties(filter: MarsApiFilter) {
```

2. Modify the call to `getProperties()` in the Retrofit service to pass along that filter query as a string.

```kotlin
 _properties.value = MarsApi.retrofitService.getProperties(filter.value)
```

3. In the `init {}` block, pass `MarsApiFilter.SHOW_ALL` as an argument to `getMarsRealEstateProperties()`, to show all properties when the app first loads.

```kotlin
/**
* Call getMarsRealEstateProperties() on init so we can display status immediately.
*/
init {
   getMarsRealEstateProperties(MarsApiFilter.SHOW_ALL)
}
```

4. At the end of the class, add an `updateFilter()` method that takes a `MarsApiFilter` argument and calls `getMarsRealEstateProperties()` with that argument.

```kotlin
/**
 * Updates the data set filter for the web services by querying the data with the new filter
 * by calling [getMarsRealEstateProperties]
 * @param filter the [MarsApiFilter] that is sent as part of the web server request
 */
fun updateFilter(filter: MarsApiFilter) {
   getMarsRealEstateProperties(filter)
}
```

## Step 3: Connect the fragment to the options menu

The last step is to hook up the overflow menu to the fragment to call `updateFilter()` on the view model when the user picks a menu option.

1. Open `res/menu/overflow_menu.xml`. The MarsRealEstate app has an existing overflow menu that provides the three available options: showing all properties, showing just rentals, and showing just for-sale properties.

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android">
   <item
       android:id="@+id/show_all_menu"
       android:title="@string/show_all" />
   <item
       android:id="@+id/show_rent_menu"
       android:title="@string/show_rent" />
   <item
       android:id="@+id/show_buy_menu"
       android:title="@string/show_buy" />
</menu>
```

2. Open `overview/OverviewFragment.kt`. At the end of the class, implement the `onOptionsItemSelected()` method to handle menu item selections.

```kotlin
override fun onOptionsItemSelected(item: MenuItem): Boolean {
}
```

3. In `onOptionsItemSelected()`, call the `updateFilter()` method on the view model with the appropriate filter. Use a Kotlin `when {}` block to switch between the options. Use `MarsApiFilter.SHOW_ALL` for the default filter value. Return `true`, because you've handled the menu item. Import `MarsApiFilter` (`com.example.android.marsrealestate.network.MarsApiFilter`) when requested. The complete `onOptionsItemSelected()` method is shown below.

```kotlin
/**
 * Updates the filter in the [OverviewViewModel] when the menu items are selected from the
 * overflow menu.
 */
override fun onOptionsItemSelected(item: MenuItem): Boolean {
   viewModel.updateFilter(
           when (item.itemId) {
               R.id.show_rent_menu -> MarsApiFilter.SHOW_RENT
               R.id.show_buy_menu -> MarsApiFilter.SHOW_BUY
               else -> MarsApiFilter.SHOW_ALL
           }
   )
   return true
}
```

4. Compile and run the app. The app launches the first overview grid with all property types and the for-sale properties marked with the dollar icon.
5. Choose **Rent** from the options menu. The properties reload and none of them appear with the dollar icon. (Only rental properties are shown.) You might have to wait a few moments for the display to refresh to show only the filtered properties.
6. Choose **Buy** from the options menu. The properties reload again, and all of them appear with the dollar icon. (Only for-sale properties are shown.)

***

### Extra: Add "for sale" images to the overview

Up until now, the only part of the Mars property data you've used is the URL for the property image. But the property data—which you defined in the `MarsProperty` class—also includes an ID, a price, and a type (rental or for sale). To refresh your memory, here's a snippet of the JSON data you get from the web service:

```json
{
   "price":8000000,
   "id":"424908",
   "type":"rent",
   "img_src": "http://mars.jpl.nasa.gov/msl-raw-images/msss/01000/mcam/1000ML0044631290305226E03_DXXX.jpg"
},
```

In this task, you start working with the Mars property type to add a dollar-sign image to the properties on the overview page that are for sale.


#### Step 1: Update the grid item layout

Now you update the item layout for the grid of images to show a dollar-sign drawable only on those property images that are for sale:

![](https://developer.android.com/codelabs/kotlin-android-training-internet-filtering/img/670b5be64a12ddc.png)

With data binding expressions you can do this test entirely in the XML layout for the grid items.

1. Open `res/layout/grid_view_item.xml`. This is the layout file for each individual cell in the grid layout for the `RecyclerView`. Currently the file contains only the `<ImageView>` element for the property image.
2. Inside the `<data>` element, add an `<import>` element for the [`View`](https://developer.android.com/reference/android/view/View) class. You use imports when you want to use components of a class inside a data binding expression in a layout file. In this case, you are going to use the `View.GONE` and `View.VISIBLE` constants, so you need access to the `View` class.

```xml
<import type="android.view.View"/>
```

3. Surround the entire image view with a `FrameLayout`, to allow the dollar-sign drawable to be stacked on top of the property image.

```xml
<FrameLayout
   android:layout_width="match_parent"
   android:layout_height="170dp">
             <ImageView 
                    android:id="@+id/mars_image"
            ...
</FrameLayout>
```

4. For the `ImageView`, change the `android:layout_height` attribute to `match_parent`, to fill the new parent `FrameLayout`.

```xml
android:layout_height="match_parent"
```

5. Add a second `<ImageView>` element just below the first one, inside the `FrameLayout`. Use the definition shown below. This image appears in the lower right corner of the grid item, on top of the Mars image, and uses the drawable defined in `res/drawable/ic_for_sale_outline.xml` for the dollar-sign icon.

```xml
<ImageView
   android:id="@+id/mars_property_type"
   android:layout_width="wrap_content"
   android:layout_height="45dp"
   android:layout_gravity="bottom|end"
   android:adjustViewBounds="true"
   android:padding="5dp"
   android:scaleType="fitCenter"
   android:src="@drawable/ic_for_sale_outline"
   tools:src="@drawable/ic_for_sale_outline"/>
```

6. Add the `android:visibility` attribute to the `mars_property_type` image view. Use a binding expression to test for the property type, and assign the visibility either to `View.GONE` (for a rental) or `View.VISIBLE` (for a purchase).

```kotlin
 android:visibility="@{property.rental ? View.GONE : View.VISIBLE}"
```

Until now you have only seen binding expressions in layouts that use individual variables defined in the `<data>` element. Binding expressions are extremely powerful and enable you to do operations such as tests and math calculations entirely within your XML layout. In this case, you use the ternary operator (`?:`) to perform a test (is this object a rental?). You provide one result for true (hide the dollar-sign icon with `View.GONE`) and another for false (show that icon with `View.VISIBLE`).

The new complete `grid_view_item.xml` file is shown below:

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
       xmlns:app="http://schemas.android.com/apk/res-auto"
       xmlns:tools="http://schemas.android.com/tools">
   <data>
       <import type="android.view.View"/>
       <variable
           name="property"
           type="com.example.android.marsrealestate.network.MarsProperty" />
   </data>
   <FrameLayout
       android:layout_width="match_parent"
       android:layout_height="170dp">

       <ImageView
           android:id="@+id/mars_image"
           android:layout_width="match_parent"
           android:layout_height="match_parent"
           android:scaleType="centerCrop"
           android:adjustViewBounds="true"
           android:padding="2dp"
           app:imageUrl="@{property.imgSrcUrl}"
           tools:src="@tools:sample/backgrounds/scenic"/>

       <ImageView
           android:id="@+id/mars_property_type"
           android:layout_width="wrap_content"
           android:layout_height="45dp"
           android:layout_gravity="bottom|end"
           android:adjustViewBounds="true"
           android:padding="5dp"
           android:scaleType="fitCenter"
           android:src="@drawable/ic_for_sale_outline"
           android:visibility="@{property.rental ? View.GONE : View.VISIBLE}"
           tools:src="@drawable/ic_for_sale_outline"/>
   </FrameLayout>
</layout>
```

7. Compile and run the app, and note that properties that are not rentals have the dollar-sign icon.

***

### Filtering and detail views with internet data Summary

**Binding expressions**
* Use [binding expressions](https://developer.android.com/topic/libraries/data-binding/expressions) in XML layout files to perform simple programmatic operations, such as math or conditional tests, on bound data.
* To reference classes inside your layout file, use the `<import>` tag inside the `<data>` tag.

**Web service query options**
* Requests to web services can include optional parameters.
* To specify query parameters in the request, use the `@Query` annotation in [Retrofit](https://square.github.io/retrofit/).

***

### Extra: Differences between object and companion object

Objects can implement interfaces. Inside a class, defining a simple object that doesn't implement any interfaces has no benefit in most cases. However, defining multiple objects that implement various interfaces (e.g. `Comparator`) can be very useful.

In terms of lifecycle, there is no difference between a companion object and a named object declared in a class.

There are two different types of `object` uses, **expression** and **declaration**.

**Object Expression**
An object expression can be used when a class needs slight modification, but it's not necessary to create an entirely new subclass for it. Anonymous inner classes are a good example of this.

```kotlin
button.setOnClickListener(object: View.OnClickListener() {
    override fun onClick(view: View) {
        // click event
    }
})
```

One thing to watch out for is that anonymous inner classes can access variables from the enclosing scope, and these variables do not have to be `final`. This means that a variable used inside an anonymous inner class that is not considered `final` can change value unexpectedly before it is accessed.

**Object Declaration**
An object declaration is similar to a variable declaration and therefore cannot be used on the right side of an assignment statement. Object declarations are very useful for implementing the Singleton pattern.

```kotlin
object MySingletonObject {
    fun getInstance(): MySingletonObject {
        // return single instance of object
    }
}
```

And the `getInstance` method can then be invoked like this. `MySingletonObject.getInstance()`.

**Companion Object**
Companion objects are essentially the same as a standard _object_ definition, only with a couple of additional features to make development easier.

A companion object is always declared inside of another class. **Whilst it can have a name, it doesn't need to have one**, in which case it automatically has the name _Companion_:

```kotlin
class OuterClass {
    companion object { // Equivalent to "companion object Companion"
    }
}
```

Companion objects allow their members to be accessed from inside the companion class without specifying the name.

```kotlin
class OuterClass {
    companion object {
        private val secret = "You can't see me"
        val public = "You can see me"
    }

    fun getSecretValue() = secret
}
```

[Source 1](https://stackoverflow.com/questions/43814616/kotlin-difference-between-object-and-companion-object-in-a-class) - [Source 2](https://www.baeldung.com/kotlin-objects).

**[Semantic difference between object expressions and declarations](https://kotlinlang.org/docs/reference/object-declarations.html#semantic-difference-between-object-expressions-and-declarations)**
There is one important semantic difference between object expressions and object declarations:
* object expressions are executed (and initialized) **immediately**, where they are used;
* object declarations are initialized **lazily**, when accessed for the first time;
* a companion object is initialized when the corresponding class is loaded (resolved), matching the semantics of a Java static initializer.

Given the above explanation, the use-case completely depends on the problem we are trying to solve. If we need to provide the `Singleton` behavior, then we are better off with `Objects`, else if we just want to add some `static essence` to our classes, we can use `Companion objects`.

[Source 3](https://medium.com/mindorks/kotlin-object-vs-companion-object-a1907c76a2af)

***

### Summary

#### Getting data from the internet

**REST web services**

* A _web service_ is a service on the internet that enables your app to make requests and get data back.
* Common web services use a [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) architecture. Web services that offer REST architecture are known as _RESTful_ services. RESTful web services are built using standard web components and protocols.
* You make a request to a REST web service in a standardized way, via URIs.
* To use a web service, an app must establish a network connection and communicate with the service. Then the app must receive and parse response data into a format the app can use.
* The [Retrofit](https://square.github.io/retrofit/) library is a client library that enables your app to make requests to a REST web service.
* Use converters to tell Retrofit what do with data it sends to the web service and gets back from the web service. For example, the `ScalarsConverter` converter treats the web service data as a `String` or other primitive.
* To enable your app to make connections to the internet, add the `"android.permission.INTERNET"` permission in the Android manifest.

**JSON parsing**

* The response from a web service is often formatted in [JSON](https://www.json.org/), a common interchange format for representing structured data.
* A JSON object is a collection of key-value pairs. This collection is sometimes called a _dictionary_, a _hash map_, or an _associative array_.
* A collection of JSON objects is a JSON array. You get a JSON array as a response from a web service.
* The keys in a key-value pair are surrounded by quotes. The values can be numbers or strings. Strings are also surrounded by quotes.
* The [Moshi](https://github.com/square/moshi) library is an Android JSON parser that converts a JSON string into Kotlin objects. Retrofit has a converter that works with Moshi.
* Moshi matches the keys in a JSON response with properties in a data object that have the same name.
* To use a different property name for a key, annotate that property with the `@Json` annotation and the JSON key name.

#### Loading and displaying images from the internet

* To simplify the process of managing images, use the [Glide library](https://github.com/bumptech/glide) to download, buffer, decode, and cache images in your app.
* Glide needs two things to load an image from the internet: the URL of an image, and an `ImageView` object to put the image in. To specify these options, use the `load()` and `into()` methods with Glide.
* [Binding adapters](https://developer.android.com/topic/libraries/data-binding/binding-adapters) are extension methods that sit between a view and that view's bound data. Binding adapters provide custom behavior when the data changes, for example, to call Glide to load an image from a URL into an `ImageView`.
* Binding adapters are extension methods annotated with the `@BindingAdapter` annotation.
* To add options to the Glide request, use the `apply()` method. For example, use `apply()` with `placeholder()` to specify a loading drawable, and use `apply()` with `error()` to specify an error drawable.
* To produce a grid of images, use a `RecyclerView` with a `GridLayoutManager`.
* To update the list of properties when it changes, use a binding adapter between the `RecyclerView` and the layout.

#### Filtering and detail views with internet data

**Binding expressions**

* Use [binding expressions](https://developer.android.com/topic/libraries/data-binding/expressions) in XML layout files to perform simple programmatic operations, such as math or conditional tests, on bound data.
* To reference classes inside your layout file, use the `<import>` tag inside the `<data>` tag.

**Web service query options**

* Requests to web services can include optional parameters.
* To specify query parameters in the request, use the `@Query` annotation in [Retrofit](https://square.github.io/retrofit/).

