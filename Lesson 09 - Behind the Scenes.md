## Lesson 09: Behind the Scenes

> Not everything is displayed on the screen. In Android, there's so much happening in the background and you'll get to build your own background services and tasks in this lesson.

### Introduction

Your users waiting for your app to launch, not so much. Let's take a look at two apps. Both apps rely on data from the network.

* The first app fetches the network data every single time the app launches. This means a loading spinner and another loading spinner and another loading spinner. Even though we're going to show the exact same data as the last time we launched the app. Your users will grow tired of this experience.

* Second app is snappy. When it launches, it immediately shows the data fetched from the last time the app opened. This is enabled by offline caching. Users expect your app to show data as soon as possible after it opens.

Offline caching lets you cache or save the data locally from the last time the app opened, so you have something to show right away. By implementing offline caching, users can still use your app when offline.

In this lesson, you'll learn how to implement offline caching by building [an app](https://github.com/udacity/andfun-kotlin-dev-bytes) that lets users watch DevByte videos. DevBytes are short videos made by the android developer relations team to introduce new developer features on android. They're a great way to stay up to date with new features as they come out, as well as tips and tricks and best practices. As you follow the steps of this lesson, you'll take an online only app and transform it to work by adding offline caching.

This starter app comes with a lot of code, in particular all the networking and user interface modules, so that you can focus on the repository module of the app.

***

### Caching

Caching means storing something for future use. In computer science when we say cache we often mean to copy data closer to the place it will be used, so it can be accessed faster.
Your browser, for example, has a cache, it stores a copy of every web page you visit on the local file system, so we can try to load the resources locally the next time you visit the page.

When writing an Android app, where should you cache data? On the file system. Every Android app has access to a folder it can use to store files and data.

You can implement a cache the way your web browser does in your Android app using Retrofit. That's the networking library you used in the previous lesson. Every time you get a network result, keep a copy on disk, then the next time you query at the same place, you load the stored copy from the disk. You can configure Retrofit to do this caching for you, and it's a really good idea to turn that on for production apps.

However, most apps need to implement a more sophisticated type of caching. When you cache data or make a copy of it, you have to think about cache invalidation.

Cache invalidation means knowing when the data in a cache is not correct anymore. It may have changed on the server, or perhaps it's just been in the cache so long it started to grow new features. Either way, the data is too old to use from the cache, so you have to throw it away.

Network requests over HTTP have a bunch of features to help with this, and when you enable network caching and Retrofit, you'll use all of them. But as your complexity grows, you'll need more structure for your cache, for example, your data may be a combination of several network requests or perhaps your backend is written in a way that you can't use network caching from Retrofit.

In these cases, Retrofit caching may still help save your users bandwidth, but it's not going to create a great cache for your app.

The best way to store structured data in an offline cache and give you control over how data is stored and invalidated is to use a SQL database. Instead of hosting the database on a server somewhere, you keep it locally on the device file system.

After an app fetches data from the network, the app can [cache](https://searchstorage.techtarget.com/definition/cache) the data by storing the data in a device's storage. You cache data so that you can access it later when the device is offline, or if you want to access the same data again.

The following table shows several ways to implement network caching in Android. In this codelab, you use `Room`, because it's the recommended way to store structured data on a device file system.

|  |  |
| --- | --- |
| **Caching technique** | **Uses** |
| [`Retrofit`](http://square.github.io/retrofit/) is a networking library used to implement a type-safe REST client for Android. You can configure Retrofit to store a copy of every network result locally. | Good solution for simple requests and responses, infrequent network calls, or small datasets. |
| You can use [`SharedPreferences`](https://developer.android.com/training/data-storage/shared-preferences.html) to store key-value pairs. | Good solution for a small number of keys and simple values. You can't use this technique to store large amounts of structured data. |
| You can access the app's internal storage directory and save data files in it. Your app's package name specifies the app's internal storage directory, which is in a special location in the Android file system. This directory is private to your app, and it is cleared when your app is uninstalled. | Good solution if you have specific needs that a file system can solve—for example, if you need to save media files or data files and you have to manage the files yourself. You can't use this technique to store complex and structured data. |
| You can cache data using [`Room`](https://developer.android.com/topic/libraries/architecture/room), which is an SQLite object-mapping library that provides an abstraction layer over SQLite. | Recommended solution for complex and structured data, because the best way to store structured data on a device's file system is in a local SQLite database. |

***

### How to store data

**Which statements are true about using a database for an offline cache?**

* Storing the latest result in the database means the offline cache is always up to date.
* Always displaying from the database exposes issues stale cache data during development.
* When updating values, make sure the server responds successfully before saving them in the offline cache.

The DevBytes starter app fetches a list of video URLs from the network using the [Retrofit](https://square.github.io/retrofit/) library and displays the list using a `RecyclerView`. The app uses `ViewModel` and `LiveData` to hold the data and update the UI. The app's architecture is similar to apps you developed previously in this course.

The starter app is online-only, so the user needs a network connection to use it. In this codelab, you implement offline caching to display results from the local database, instead of from the network. Your users will be able to use the app while their device is offline, or if they have a slow network connection.

To implement the offline cache, you use a [`Room`](https://developer.android.com/topic/libraries/architecture/room) database to make fetched data persistent in the device's local storage. You access and manage the `Room` database using a _repository pattern_, which is a design pattern that isolates data sources from the rest of the app. This technique provides a clean API for the rest of the app to use for accessing the data.

**Key concept:** Don't retrieve data from the network every time the app is launched. Instead, display data that you fetch from the database. This technique decreases app-loading time.

When the app fetches data from the network, store the data in the database instead of displaying the data immediately.

When a new network result is received, update the local database and display the new content on the screen from the local database. This technique ensures that the offline cache is always up-to-date. Also, if the device is offline, your app can still load locally cached data.

Even better, by following this pattern, it's really easy to spot bugs in your offline cache during development not when you get a report of strange data from a user.
What happens when you need to change a value that's stored in the database. Say for example, you have a user profile and the user changes their name. You need to update the server, and you also need to update the offline cache somehow since it's the single source of truth.

You could do this by writing the new name to your local database, then trying to write it to the server. This has a major problem. Say you update the cache, then try to tell the server about the change. The server may end up out of sync. It may never get the updated data because the network is not available or the server may experience an error while saving. If that happens, your app will show incorrect data from the offline cache.

To avoid the data getting out of sync, the easiest way to update values is to delay updating the offline cache until after the server lets you know the data is saved. Once you get the result from the server, you can write it to the offline cache. This way, you can be sure your offline cache always holds the correct data.

***

### Decorating a Room

Now that you're familiar with patterns for offline caching, it's time to implement one. Our app will show a list of DevByte videos which come from the server. We've created a server for you, and the starter app will use Retrofit to fetch that list. You've already learned how to use Room, and as a review, here's the features you'll need to build an offline cache.

DevByte videos will be represented by a `DatabaseVideo` entity. They have a `PrimaryKey` of the video URL. The server design ensures that the video URL will always be unique. Then, for the `DAO`, or data access object,
you'll need two methods.

* The first is `getVideos`. `GetVideos` returns a list of videos.Taking a look at the UI, this is all we need to display the home screen, a list of all the videos in the database. To implement this in Room, you'll use the query annotation with select all from `databasevideo`.

* The other thing your cache needs is a way to save values into the database. When new data comes in from the network, you'll call this to save them in the cache. We use `insertAll` because we want to insert all new or changed videos in one call. When you call `insertAll`, by default, it throws an exception if there's already an entity with the same `PrimaryKey` in the database. To solve this problem, we'll specify a conflict strategy.

A conflict strategy tells the database what to do when it experiences a conflict, that is, when you try to insert the same `PrimaryKey` that's already in the database.

When saving data from the network in an offline cache, we often just want to overwrite anything that used to be in the database, since the new network result is more up to date. The replace strategy does exactly this. Whenever there is a conflict, the old content is replaced with new values. This type of insert is called an `upsert`, a combination of update and insert. You may hear this called `upsert` or `OnConflict` update by some developers.

That's really all there is to a cache, a way to save data and a way to load it.

***

### Building a Room


#### Explore the code

1. In Android Studio, expand all the packages.
2. Explore the `domain` package. This package contains `data` classes for representing the app's data. For example, the `DevByteVideo` data class in `domain/Models.kt` class represents a single DevByte video.
3. Explore the `network` package.

The `network/DataTransferObjects.kt` class contains the data class for a [data transfer object](https://en.wikipedia.org/wiki/Data_transfer_object) called `NetworkVideo`. The data transfer object is used to parse the network result. This file also contains a convenience method, `asDomainModel()`, to convert network results to a list of domain objects. The data transfer objects are different from the domain objects, because they contain extra logic for parsing network results.

```kotlin
fun NetworkVideoContainer.asDomainModel(): List<Video> {
 return videos.map {
 Video (
 title = it.title,
 description = it.description,
 url = it.url,
 updated = it.updated,
 thumbnail = it.thumbnail)
 }
}
```

The code above defines an extension function on `NetworkViewHolder` called `asDomainModel` that returns a `List<Video>`. In the body, you can see it just converts `NetworkVideo` into `Video` using the `map` function from the Kotlin standard library.

**Tip:** It's a best practice to separate the network, domain, and database objects. This strategy follows the [separation of concerns](https://en.wikipedia.org/wiki/Separation_of_concerns) principle. If the network response or the database schema changes, you want to be able to change and manage app components without updating the entire app's code.

4. Try exploring the rest of starter code on your own.

The rest of the apps' architecture is similar to the other apps used in the previous lessons:

* The Retrofit service, `network/Service.kt`, fetches the `devbytes` playlist from the network.
* The `DevByteViewModel` holds the app data as `LiveData` objects.
* The UI controller, `DevByteFragment`, contains a `RecyclerView` to display the video list and the observers for the `LiveData` objects.

***

### Add a DatabaseVideo Entity

## Step 1: Add the Room dependency

1. Open the `build.gradle (Module:app)` file and add the `Room` dependency to the project.

```kotlin
// Room dependency
def room_version = "2.1.0-alpha06"
implementation "androidx.room:room-runtime:$room_version"
kapt "androidx.room:room-compiler:$room_version"
```

#### Step 2: Adding database object

In this step, you create a database entity named `DatabaseVideo` to represent database objects. You also implement convenience methods to convert `DatabaseVideo` objects into domain objects, and to convert network objects into `DatabaseVideo` objects.

1. Open `database/DatabaseEntities.kt` and create a `Room` entity called `DatabaseVideo`. Set `url` as the primary key. The DevBytes server design ensures that the video URL is always unique.

```kotlin
/**
* DatabaseVideo represents a video entity in the database.
*/
@Entity
data class DatabaseVideo constructor(
       @PrimaryKey
       val url: String,
       val updated: String,
       val title: String,
       val description: String,
       val thumbnail: String)
```

2. In `database/DatabaseEntities.kt`, create an extension function called `asDomainModel()`. Use the function to convert `DatabaseVideo` database objects into domain objects.

```kotlin
/**
* Map DatabaseVideos to domain entities
*/
fun List<DatabaseVideo>.asDomainModel(): List<DevByteVideo> {
   return map {
       DevByteVideo(
               url = it.url,
               title = it.title,
               description = it.description,
               updated = it.updated,
               thumbnail = it.thumbnail)
   }
}
```

In this sample app, the conversion is simple, and some of this code isn't necessary. But in a real-world app, the structure of the domain, database, and network objects will be different. You'll need conversion logic, which can get complicated.

3. Open `network/DataTransferObjects.kt` and create an extension function called `asDatabaseModel()`. Use the function to convert network objects into `DatabaseVideo` database objects.

```kotlin
/**
* Convert Network results to database objects
*/
fun NetworkVideoContainer.asDatabaseModel(): List<DatabaseVideo> {
   return videos.map {
       DatabaseVideo(
               title = it.title,
               description = it.description,
               url = it.url,
               updated = it.updated,
               thumbnail = it.thumbnail)
   }
}
```

***

### Adding the VideoDao

In this step, you implement `VideoDao` and define two helper methods to access the database. One helper method gets videos from the database, and the other method inserts videos into the database.

1. In `database/Room.kt`, define a `VideoDao` interface and annotate is with `@Dao`.

```kotlin
@Dao
interface VideoDao { 
}
```

2. Inside the `VideoDao` interface, create a method called `getVideos()` to fetch all the videos from the database. Change the return type of this method to `LiveData`, so that the data displayed in the UI is refreshed whenever the data in the database is changed.

```kotlin
@Query("select * from databasevideo")
fun getVideos(): LiveData<List<DatabaseVideo>>
```

If an `Unresolved reference` error appears in Android Studio, import `androidx.room.Query`.

3. Inside the `VideoDao` interface, define another `insertAll()` method to insert a list of videos fetched from the network into the database. For simplicity, overwrite the database entry if the video entry is already present in the database. To do this, use the `onConflict` argument to set the conflict strategy to `REPLACE`.

```kotlin
@Insert(onConflict = OnConflictStrategy.REPLACE)
fun insertAll( videos: List<DatabaseVideo>)
```

The full code of the `DAO` should be:

```kotlin
package com.example.android.devbyteviewer.database

import androidx.lifecycle.LiveData
import androidx.room.Dao
import androidx.room.Insert
import androidx.room.OnConflictStrategy
import androidx.room.Query

@Dao
interface VideoDao {
  @Query("select * from databasevideo")
  fun getVideos(): LiveData<List<DatabaseVideo>>

  @Insert(onConflict = OnConflictStrategy.REPLACE)
  fun insertAll(vararg videos: DatabaseVideo)
}
```

**When should you prefer `LiveData` in Room `@Queries`?**

* You want to watch for changes to the database and update the UI dynamically.
* You want to be able to call the query from the UI thread.
* You want Room to run the query to run on a background thread.

***

### Adding the `VideosDatabase`

In this step, you add the database for your offline cache by implementing [`RoomDatabase`](https://developer.android.com/reference/android/arch/persistence/room/RoomDatabase).

1. In `database/Room.kt`, after the `VideoDao` interface, create an `abstract` class called `VideosDatabase`. Extend `VideosDatabase` from `RoomDatabase`.
2. Use the `@Database` annotation to mark the `VideosDatabase` class as a `Room` database. Declare the `DatabaseVideo` entity that belongs in this database, and set the version number to `1`.
3. Inside `VideosDatabase`, define a variable of the type `VideoDao` to access the `Dao` methods.

```kotlin
@Database(entities = [DatabaseVideo::class], version = 1)
abstract class VideosDatabase: RoomDatabase() {
   abstract val videoDao: VideoDao
}
```

4. Create a `private` `lateinit` variable called `INSTANCE` outside the classes, to hold the singleton object. The `VideosDatabase` should be [singleton](https://en.wikipedia.org/wiki/Singleton_pattern) to prevent having multiple instances of the database opened at the same time.
5. Create and define a `getDatabase()` method outside the classes. In `getDatabase()`, initialize and return the `INSTANCE` variable inside the `synchronized` block.

```kotlin
private lateinit var INSTANCE: VideosDatabase

fun getDatabase(context: Context): VideosDatabase {
    synchronized(VideosDatabase::class.java) {
        if (!::INSTANCE.isInitialized) {
            INSTANCE = Room.databaseBuilder(context.applicationContext,
                    VideosDatabase::class.java,
                    "videos").build()
        }
    }
    return INSTANCE
}
```

**Tip**: The `.isInitialized` Kotlin property returns `true` if the `lateinit` property (`INSTANCE` in this example) has been assigned a value, and `false` otherwise.

***

### Repositories

#### The [repository pattern](https://developer.android.com/jetpack/docs/guide#fetch-data)

The _repository pattern_ is a design pattern that isolates data sources from the rest of the app.

A _repository_ mediates between data sources (such as persistent models, web services, and caches) and the rest of the app. The diagram below shows how app components such as activities that use `LiveData` might interact with data sources by way of a repository.

![Repositories.png](images/Repositories.png)

To implement a repository, you use a _repository class_, such as the `VideosRepository` class that you create in the next task. The repository class isolates the data sources from the rest of the app and provides a clean API for data access to the rest of the app. Using a repository class is a recommended best practice for code separation and architecture.

#### Advantages of using a repository

A repository module handles data operations and allows you to use multiple backends. In a typical real-world app, the repository implements the logic for deciding whether to fetch data from a network or use results that are cached in a local database. This helps make your code modular and testable. You can easily mock up the repository and test the rest of the code.

***

### Building a Repository

Your Room database has been working out nicely. It’s saving all the Videos, and loading them. Saving and loading. It’s a pretty good database. Nice work.

Now we need you to implement a repository. You were just briefed on how the repository pattern can be used to implement offline caching - time to put it into practice.

A `Repository` is just a regular class that has one (or more) methods that load data without specifying the data source as part of the main API. Because it's just a regular class, there's no need for an annotation to define a repository. The repository hides the complexity of managing the interactions between the database and the networking code.

You can define any API that makes sense for your data. Make your repo take a `VideosDatabase` constructor parameter, this way you won’t have any dependencies on Context in your repository (so called dependency injection).

#### Step 1: Adding a repository

1. In `repository/VideosRepository.kt`, create a `VideosRepository` class. Pass in a `VideosDatabase` object as the class's constructor parameter to access the `Dao` methods.

```kotlin
/**
* Repository for fetching devbyte videos from the network and storing them on disk
*/
class VideosRepository(private val database: VideosDatabase) {
}
```

2. Inside the `VideosRepository` class, add a `refreshVideos()` method that has no arguments and returns nothing. This method will be the API used to refresh the offline cache.
3. Make `refreshVideos()` a suspend function. Because `refreshVideos()` performs a database operation, it must be called from a coroutine.

**Note**: Databases on Android are stored on the file system, or disk, and in order to save they must perform a disk I/O. The disk I/O, or reading and writing to disk, is slow and always blocks the current thread until the operation is complete. Because of this, you have to run the disk I/O in the [I/O dispatcher](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-i-o.html). This dispatcher is designed to offload blocking I/O tasks to a shared pool of threads using [`withContext`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-context.html)`(Dispatchers.IO) { ... }`.

4. Inside the `refreshVideos()` method, switch the coroutine context to `Dispatchers.IO` to perform network and database operations.

```kotlin
/**
* Refresh the videos stored in the offline cache.
*
* This function uses the IO dispatcher to ensure the database insert database operation
* happens on the IO dispatcher. By switching to the IO dispatcher using `withContext` this
* function is now safe to call from any thread including the Main thread.
*
*/
suspend fun refreshVideos() {
   withContext(Dispatchers.IO) {
   }
}
```

5. Inside the `withContext` block, fetch the DevByte video playlist from the network using the Retrofit service instance, `DevByteNetwork`.

```kotlin
val playlist = DevByteNetwork.devbytes.getPlaylist()
```

6. Inside the `refreshVideos()` method, after fetching the playlist from the network, store the playlist in the `Room` database.

To store the playlist, use the `VideosDatabase` object, `database`. Call the `insertAll` DAO method, passing in the `playlist` retrieved from the network. Use the `asDatabaseModel()` extension function to map the `playlist` to the database object.

```kotlin
database.videoDao.insertAll(playlist.asDatabaseModel())
```

7. Here is the complete `refreshVideos` method with a log statement for tracking when it gets called:

```kotlin
suspend fun refreshVideos() {
   withContext(Dispatchers.IO) {
       Timber.d("refresh videos is called");
       val playlist = DevByteNetwork.devbytes.getPlaylist()
       database.videoDao.insertAll(playlist.asDatabaseModel())
   }
}
```

#### Step 2: Retrieve data from the database

In this step, you create a `LiveData` object to read the video playlist from the database. This `LiveData` object is automatically updated when the database is updated. The attached fragment, or the activity, is refreshed with new values.

1. In the `VideosRepository` class, declare a `LiveData` object called `videos` to hold a list of `DevByteVideo` objects.
2. Initialize the `videos` object, using `database.videoDao`. Call the `getVideos()` DAO method. Because the `getVideos()` method returns a list of database objects, not a list of `DevByteVideo` objects, Android Studio throws a "type mismatch" error.

```kotlin
val videos: LiveData<List<DevByteVideo>> = database.videoDao.getVideos()
```

3. To fix the error, use `Transformations.map` to convert the list of database objects to a list of domain objects. Use the `asDomainModel()` conversion function.

**Refresher**: The [`Transformations.map`](https://developer.android.com/reference/android/arch/lifecycle/Transformations.html#map%28android.arch.lifecycle.LiveData%3CX%3E,%20android.arch.core.util.Function%3CX,%20Y%3E%29) method uses a conversion function to convert one `LiveData` object into another `LiveData` object. The transformations are only calculated when an active activity or a fragment is observing the returned `LiveData` property.

```kotlin
val videos: LiveData<List<DevByteVideo>> = Transformations.map(database.videoDao.getVideos()) {
   it.asDomainModel()
}
```

The complete code now should be like:

```kotlin
class VideosRepository(private val database: VideosDatabase) {

    /**
     * A playlist of videos that can be shown on the screen.
     */
    val videos: LiveData<List<Video>> =
            Transformations.map(database.videoDao.getVideos()) {
        it.asDomainModel()
    }

    /**
     * Refresh the videos stored in the offline cache.
     *
     * This function uses the IO dispatcher to ensure the database insert database operation
     * happens on the IO dispatcher. By switching to the IO dispatcher using `withContext` this
     * function is now safe to call from any thread including the Main thread.
     *
     * To actually load the videos for use, observe [videos]
     */
    suspend fun refreshVideos() {
        withContext(Dispatchers.IO) {
            val playlist = DevByteNetwork.devbytes.getPlaylist()
            database.videoDao.insertAll(playlist.asDatabaseModel())
        }
    }
}
```

***

### Using the Repository

In this task, you integrate your repository with the `ViewModel` using a simple refresh strategy. You display the video playlist from the `Room` database, not directly fetching from the network.

A _database refresh_ is a process of updating or refreshing the local database to keep it in sync with data from the network. For this sample app, you use a very simple refresh strategy, where the module that requests data from the repository is responsible for refreshing the local data.

In a real-world app, your strategy might be more complex. For example, your code might automatically refresh the data in the background (taking bandwidth into account), or cache the data that the user is most likely to use next.

1. In `viewmodels/DevByteViewModel.kt`, inside `DevByteViewModel` class, create a `private` member variable called `videosRepository` of the type `VideosRepository`. Instantiate the variable by passing in the singleton `VideosDatabase` object.

```kotlin
/**
* The data source this ViewModel will fetch results from.
*/
private val videosRepository = VideosRepository(getDatabase(application))
```

2. In the `DevByteViewModel` class, replace the `refreshDataFromNetwork()` method with the `refreshDataFromRepository()` method.

The old method, `refreshDataFromNetwork()`, fetched the video playlist from the network using the Retrofit library. The new method loads the video playlist from the repository.

```kotlin
/**
* Refresh data from the repository. Use a coroutine launch to run in a
* background thread.
*/
private fun refreshDataFromRepository() {
   viewModelScope.launch {
       try {
           videosRepository.refreshVideos()
           _eventNetworkError.value = false
           _isNetworkErrorShown.value = false

       } catch (networkError: IOException) {
           // Show a Toast error message and hide the progress bar.
           if(playlist.value.isNullOrEmpty())
               _eventNetworkError.value = true
       }
   }
}
```

3. In the `DevByteViewModel` class, inside the `init` block, change the function call from `refreshDataFromNetwork()` to `refreshDataFromRepository()`. This code fetches the video playlist from the repository, not directly from the network.

```kotlin
init {
   refreshDataFromRepository()
}
```

4. In the `DevByteViewModel` class, delete the `_playlist` property and its backing property, `playlist`.

Code to delete:

```kotlin
private val _playlist = MutableLiveData<List<Video>>()
...
val playlist: LiveData<List<Video>>
   get() = _playlist
```

5. In the `DevByteViewModel` class, after instantiating the `videosRepository` object, add a new `val` called `playlist` for holding a `LiveData` list of videos from the repository.

```kotlin
/**
* A playlist of videos displayed on the screen.
*/
val playlist = videosRepository.videos
```

6. Run your app. The app runs as before, but now the DevBytes playlist is fetched from the network and saved in the `Room` database. The playlist is displayed on the screen from the `Room` database, not directly from the network.
7. To notice the difference, enable airplane mode on the emulator or device.
8. Run the app once again. Notice that the "Network Error" toast message is not displayed, instead the playlist is fetched from the offline cache and displayed.
9. Turn off airplane mode in the emulator or device.
10. Close and re-open the app. The app loads the playlist from the offline cache, while the network request runs in the background.

If new data came in from the network, the screen would automatically update to show the new data. However, the DevBytes server does not refresh its contents, so you do not see the data updating.

**Tip:** The easiest way to remove the cache for testing is to uninstall the app.

***

### [WorkManager](https://developer.android.com/topic/libraries/architecture/workmanager) for the background

[`WorkManager`](https://developer.android.com/arch/work) is one of the [Android Architecture Components](https://developer.android.com/topic/libraries/architecture/) and part of [Android Jetpack](http://d.android.com/jetpack). `WorkManager` is for background work that's deferrable and requires guaranteed execution:

* _Deferrable_ means that the work is not required to run immediately. For example, sending analytical data to the server or syncing the database in the background is work that can be deferred.
* _Guaranteed execution_ means that the task will run even if the app exits or the device restarts.

While `WorkManager` runs background work, it takes care of compatibility issues and best practices for battery and system health. `WorkManager` offers compatibility back to API level 14. `WorkManager` chooses an appropriate way to schedule a background task, depending on the device API level. It might use [`JobScheduler`](https://developer.android.com/reference/android/app/job/JobScheduler) (on API 23 and higher) or a combination of [`AlarmManager`](https://developer.android.com/reference/android/app/AlarmManager) and [`BroadcastReceiver`](https://developer.android.com/reference/android/app/BroadcastReceiver).

![](https://developer.android.com/codelabs/kotlin-android-training-work-manager/img/e04f53ac665e07c9.png)

`WorkManager` also lets you set criteria on when the background task runs. For example, you might want the task to run only when the battery status, network status, or charge state meet certain criteria. You learn how to set constraints later in this lesson.

**Note:**
* `WorkManager` is not intended for in-process background work that can be terminated safely if the app process is killed.
* `WorkManager` is not intended for tasks that require immediate execution.

**How can you avoid using too much battery in the background?**
* Use WorkManager.
* Specify as many reasonable constraints as possible for each job, such as device idle, charging, and connected to wifi.
* Be sparing about scheduling background work - if the work can wait until next launch then do it when the app is in the foreground.
* When scheduling work that happens periodically, use the largest delay possible between each run.
* Make sure your background jobs are efficient and don't use excessive network, CPU, or disk access.

***

### Creating a Worker

You're going to upgrade DevByteViewer to pre-fetch data when the app is in the background. You should use the `WorkManager` library to accomplish this.

After you’ve completed this exercise and the next one, your users will get the freshest data every time they open the app. WorkManager will make sure to schedule the work so it has the lowest impact on battery life possible.

Before you add code to the project, familiarize yourself with the following classes in `WorkManager` library:

* [`Worker`](https://developer.android.com/reference/androidx/work/Worker.html)  
    This class is where you define the actual work (the task) to run in the background. You extend this class and override the [`doWork()`](https://developer.android.com/reference/androidx/work/Worker.html#doWork%28%29) method. The `doWork()` method is where you put code to be performed in the background, such as syncing data with the server or processing images. You implement the `Worker` in this task.
* [`WorkRequest`](https://developer.android.com/reference/androidx/work/WorkRequest.html)  
    This class represents a request to run the worker in background. Use `WorkRequest` to configure how and when to run the worker task, with the help of [`Constraints`](https://developer.android.com/reference/androidx/work/Constraints.html) such as device plugged in or Wi-Fi connected. You implement the `WorkRequest` in a later task.
* [`WorkManager`](https://developer.android.com/reference/androidx/work/WorkManager.html)  
    This class schedules and runs your `WorkRequest`. `WorkManager` schedules work requests in a way that spreads out the load on system resources, while honoring the constraints that you specify. You implement the `WorkManager` in a later task.

#### Add the `WorkManager` dependency

1. Open the `build.gradle (Module:app)` file and add the `WorkManager` dependency to the project.  
      
    If you use the [latest version](https://developer.android.com/jetpack/androidx/releases/work) of the library, the solution app should compile as expected. If it doesn't, try resolving the issue, or revert to the library version shown below.

```groovy
// WorkManager dependency
def work_version = "1.0.1"
implementation "android.arch.work:work-runtime-ktx:$work_version"
```

2. Sync your project and make sure there are no compilation errors.

#### Step 1: Create a worker

In this task, you add a [`Worker`](https://developer.android.com/reference/androidx/work/Worker) to pre-fetch the DevBytes video playlist in the background.

1. Inside the `devbyteviewer` package, create a new package called `work`.
2. Inside the `work` package, create a new Kotlin class called `RefreshDataWorker`.
3. Extend the `RefreshDataWorker` class from the `CoroutineWorker` class. Pass in the `context` and [`WorkerParameters`](https://developer.android.com/reference/androidx/work/WorkerParameters.html) as constructor parameters.

```kotlin
class RefreshDataWorker(appContext: Context, params: WorkerParameters) :
       CoroutineWorker(appContext, params) {
}
```

4. To resolve the abstract class error, override the `doWork()` method inside the `RefreshDataWorker` class.

```kotlin
override suspend fun doWork(): Result {
  return Result.success()
}
```

A _suspending function_ is a function that can be paused and resumed later. A suspending function can execute a long running operation and wait for it to complete without blocking the main thread.

#### Step 2: Implementing `doWork()`

The `doWork()` method inside the `Worker` class is called on a background thread. The method performs work synchronously, and should return a [`ListenableWorker.Result`](https://developer.android.com/reference/androidx/work/ListenableWorker.Result.html) object. The Android system gives a `Worker` a maximum of 10 minutes to finish its execution and return a `ListenableWorker.Result` object. After this time has expired, the system forcefully stops the `Worker`.

To create a `ListenableWorker.Result` object, call one of the following static methods to indicate the completion status of the background work:

* [`Result.success()`](https://developer.android.com/reference/androidx/work/ListenableWorker.Result.html#success%28%29)—work completed successfully.
* [`Result.failure()`](https://developer.android.com/reference/androidx/work/ListenableWorker.Result.html#failure%28%29)—work completed with a permanent failure.
* [`Result.retry()`](https://developer.android.com/reference/androidx/work/ListenableWorker.Result.html#retry%28%29)—work encountered a transient failure and should be retried.

In this task, you implement the `doWork()` method to fetch the DevBytes video playlist from the network. You can reuse the existing methods in the `VideosRepository` class to retrieve the data from the network.

1. In the `RefreshDataWorker` class, inside `doWork()`, create and instantiate a `VideosDatabase` object and a `VideosRepository` object.

```kotlin
override suspend fun doWork(): Result {
   val database = getDatabase(applicationContext)
   val repository = VideosRepository(database)

   return Result.success()
}
```

2. In the `RefreshDataWorker` class, inside `doWork()`, above the `return` statement, call the `refreshVideos()` method inside a `try` block. Add a log to track when the worker is run.

```kotlin
try {
   repository.refreshVideos( )
   Timber.d("Work request for sync is run")
   } catch (e: HttpException) {
   return Result.retry()
}
```

To resolve the "Unresolved reference" error, import `retrofit2.HttpException`.

3. Here is the complete `RefreshDataWorker` class for your reference:

```kotlin
class RefreshDataWorker(appContext: Context, params: WorkerParameters):
        CoroutineWorker(appContext, params) {
    /**
     * A coroutine-friendly method to do your work.
     * Note: In recent work version upgrade, 1.0.0-alpha12 and onwards have a breaking change.
     * The doWork() function now returns Result instead of Payload because they have combined Payload into Result.
     * Read more here - https://developer.android.com/jetpack/androidx/releases/work#1.0.0-alpha12
     */
    override suspend fun doWork(): Result {
        val database = getDatabase(applicationContext)
        val repository = VideosRepository(database)
        try {
           repository.refreshVideos()
       } catch (e: HttpException) {
           return Result.retry()
       }
       return Result.success()
    }
}
```

***

### Schedule Background Work

**Define a periodic WorkRequest**

A `Worker` defines a unit of work, and the [`WorkRequest`](https://developer.android.com/reference/androidx/work/WorkRequest) defines how and when work should be run. There are two concrete implementations of the `WorkRequest` class:

* The [`OneTimeWorkRequest`](https://developer.android.com/reference/androidx/work/OneTimeWorkRequest.html) class is for one-off tasks. (A _one-off_ task happens only once.)
* The [`PeriodicWorkRequest`](https://developer.android.com/reference/androidx/work/PeriodicWorkRequest.html) class is for periodic work, work that repeats at intervals.

Tasks can be one-off or periodic, so choose the class accordingly. For more information on scheduling recurring work, see the [recurring work documentation](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/recurring-work).

**Note**: The minimum interval for periodic work is 15 minutes. Periodic work can't have an initial delay as one of its constraints.

In this task, you define and schedule a `WorkRequest` to run the worker that you created in the previous task.

#### Step 1: Set up recurring work

Within an Android app, the `Application` class is the base class that contains all other components, such as activities and services. When the process for your application or package is created, the `Application` class (or any subclass of `Application`) is instantiated before any other class.

In this sample app, the `DevByteApplication` class is a subclass of the `Application` class. The `DevByteApplication` class is a good place to schedule the `WorkManager`.

1. In the `DevByteApplication` class, create a method called `setupRecurringWork()` to set up the recurring background work.

```kotlin
/**
* Setup WorkManager background job to 'fetch' new network data daily.
*/
private fun setupRecurringWork() {
}
```

2. Inside the `setupRecurringWork()` method, create and initialize a periodic work request to run once a day, using the [`PeriodicWorkRequestBuilder()`](https://developer.android.com/reference/androidx/work/PeriodicWorkRequest.Builder) method. Pass in the `RefreshDataWorker` class that you created in the previous task. Pass in a repeat interval of `1` with a time unit of `TimeUnit.`**`DAYS`**.

```kotlin
val repeatingRequest = PeriodicWorkRequestBuilder<RefreshDataWorker>(1, TimeUnit.DAYS).build()
```

To resolve the error, import `java.util.concurrent.TimeUnit`.

#### Step 2: Schedule a WorkRequest with WorkManager

After you define your `WorkRequest`, you can schedule it with `WorkManager`, using the [`enqueueUniquePeriodicWork()`](https://developer.android.com/reference/androidx/work/WorkManager.html#enqueueUniquePeriodicWork%28java.lang.String,%20androidx.work.ExistingPeriodicWorkPolicy,%20androidx.work.PeriodicWorkRequest%29) method. This method allows you to add a uniquely named [`PeriodicWorkRequest`](https://developer.android.com/reference/androidx/work/PeriodicWorkRequest.html) to the queue, where only one `PeriodicWorkRequest` of a particular name can be active at a time.

For example, you might only want one sync operation to be active. If one sync operation is pending, you can choose to let it run or replace it with your new work, using an [ExistingPeriodicWorkPolicy](https://developer.android.com/reference/androidx/work/ExistingPeriodicWorkPolicy.html).

To learn more about ways to schedule a `WorkRequest`, see the [`WorkManager`](https://developer.android.com/reference/androidx/work/WorkManager.html#public-methods_1) documentation.

1. In the `RefreshDataWorker` class, at the beginning of the class, add a companion object. Define a work name to uniquely identify this worker.

```kotlin
companion object {
   const val WORK_NAME = "com.example.android.devbyteviewer.work.RefreshDataWorker"
}
```

2. In the `DevByteApplication` class, at the end of the `setupRecurringWork()` method, schedule the work using the `enqueueUniquePeriodicWork()` method. Pass in the `KEEP` `enum` for the `ExistingPeriodicWorkPolicy`. Pass in `repeatingRequest` as the `PeriodicWorkRequest` parameter.

```kotlin
WorkManager.getInstance().enqueueUniquePeriodicWork(
       RefreshDataWorker.WORK_NAME,
       ExistingPeriodicWorkPolicy.KEEP,
       repeatingRequest)
```

If pending (uncompleted) work exists with the same name, the `ExistingPeriodicWorkPolicy.KEEP` parameter makes the `WorkManager` keep the previous periodic work and discard the new work request.

**Best Practice**: The `onCreate()` method runs in the main thread. Performing a long-running operation in `onCreate()` might block the UI thread and cause a delay in loading the app. To avoid this problem, run tasks such as initializing Timber and scheduling `WorkManager` off the main thread, inside a coroutine.

3. At the beginning of the `DevByteApplication` class, create a [`CoroutineScope`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/coroutine-scope.html) object. Pass in [`Dispatchers.Default`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-default.html) as the constructor parameter.

```kotlin
private val applicationScope = CoroutineScope(Dispatchers.Default)
```

4. In the `DevByteApplication` class, add a new method called `delayedInit()` to start a coroutine.

```kotlin
private fun delayedInit() {
   applicationScope.launch {
   }
}
```

5. Inside the `delayedInit()` method, call `setupRecurringWork()`.
6. Move the Timber initialization from the `onCreate()` method to the `delayedInit()` method.

```kotlin
private fun delayedInit() {
   applicationScope.launch {
       Timber.plant(Timber.DebugTree())
       setupRecurringWork()
   }
}
```

7. In the `DevByteApplication` class, at the end of the `onCreate()` method, add a call to the `delayedInit()` method.

```kotlin
override fun onCreate() {
   super.onCreate()
   delayedInit()
}
```

8. Open the **Logcat** pane at the bottom of the Android Studio window. Filter on `RefreshDataWorker`.
9. Run the app. The `WorkManager` schedules your recurring work immediately.  
      
    In the **Logcat** pane, notice the log statements that show that the work request is scheduled, then runs successfully.

```
D/RefreshDataWorker: Work request for sync is run
I/WM-WorkerWrapper: Worker result SUCCESS for Work [...]
```

The `WM-WorkerWrapper` log is displayed from the `WorkManager` library, so you can't change this log message.

#### Step 3: (Optional) Schedule the WorkRequest for a minimum interval

In this step, you decrease the time interval from 1 day to 15 minutes. You do this so you can see the logs for a periodic work request in action.

1. In the `DevByteApplication` class, inside the `setupRecurringWork()` method, comment out the current **`repeatingRequest`** definition. Add a new work request with a periodic repeat interval of `15` minutes.

```kotlin
// val repeatingRequest = PeriodicWorkRequestBuilder<RefreshDataWorker>(1, TimeUnit.DAYS).build()
val repeatingRequest = PeriodicWorkRequestBuilder<RefreshDataWorker>(15, TimeUnit.MINUTES).build()
```

2. Open the **Logcat** pane in Android Studio and filter on `RefreshDataWorker`. To clear the previous logs, click the **Clear logcat** icon ![](https://developer.android.com/codelabs/kotlin-android-training-work-manager/img/73fa94afd290964.png) .
3. Run the app, and the `WorkManager` schedules your recurring work immediately. In the **Logcat** pane, notice the logs—the work request is run once every 15 minutes. Wait 15 minutes to see another set of work request logs. You can leave the app running or close it; the work manager should still run.  
      
    Notice that the interval is sometimes less than 15 minutes, and sometimes more than 15 minutes. (The exact timing is subject to OS battery optimizations.)

```
12:44:40 D/RefreshDataWorker: Work request for sync is run
12:44:40 I/WM-WorkerWrapper: Worker result SUCCESS for Work 
12:59:24 D/RefreshDataWorker: Work request for sync is run
12:59:24 I/WM-WorkerWrapper: Worker result SUCCESS for Work 
13:15:03 D/RefreshDataWorker: Work request for sync is run
13:15:03 I/WM-WorkerWrapper: Worker result SUCCESS for Work 
13:29:22 D/RefreshDataWorker: Work request for sync is run
13:29:22 I/WM-WorkerWrapper: Worker result SUCCESS for Work 
13:44:26 D/RefreshDataWorker: Work request for sync is run
13:44:26 I/WM-WorkerWrapper: Worker result SUCCESS for Work
```

Congratulations! You created a worker and scheduled the work request with `WorkManager`. But there's a problem: you did not specify any constraints. `WorkManager` will schedule the work once a day, even if the device is low on battery, sleeping, or has no network connection. This will affect the device battery and performance and could result in a poor user experience.

***

### Adding constraints

In the previous task, you used `WorkManager` to schedule a work request. In this task, you add criteria for when to execute the work.

When defining the `WorkRequest`, you can specify constraints for when the `Worker` should run. For example, you might want to specify that the work should only run when the device is idle, or only when the device is plugged in and connected to Wi-Fi. You can also specify a backoff policy for retrying work. The [supported constraints](https://developer.android.com/reference/androidx/work/Constraints.Builder#public-methods_1) are the set methods in `Constraints.Builder`. To learn more, see [Defining your Work Requests](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/define-work).

**PeriodicWorkRequest and constraints**

A `WorkRequest` for repeating work, for example `PeriodicWorkRequest`, executes multiple times until it is cancelled. The first execution happens immediately, or as soon as the given constraints are met.

The next execution happens during the next period interval. Note that execution might be delayed, because `WorkManager` is subject to OS battery optimizations, for example when the device is in [Doze mode](https://developer.android.com/training/monitoring-device-state/doze-standby).

#### Step 1: Add a Constraints object and set one constraint

In this step, you create a `Constraints` object and set one constraint on the object, a network-type constraint. (It's easier to notice the logs with only one constraint. In a later step, you add other constraints.)

1. In the `DevByteApplication` class, at the beginning of `setupRecurringWork()`, define a `val` of the type `Constraints`. Use the `Constraints.Builder()` method.

```kotlin
val constraints = Constraints.Builder()
```

To resolve the error, import `androidx.work.Constraints`.

2. Use the [`setRequiredNetworkType()`](https://developer.android.com/reference/androidx/work/Constraints.Builder#setRequiredNetworkType%28androidx.work.NetworkType%29) method to add a network-type constraint to the `constraints` object. Use the `UNMETERED` enum so that the work request will only run when the device is on an unmetered network.

```kotlin
.setRequiredNetworkType(NetworkType.UNMETERED)
```

3. Use the `build()` method to generate the constraints from the builder.

```kotlin
val constraints = Constraints.Builder()
       .setRequiredNetworkType(NetworkType.UNMETERED)
       .build()
```

Now you need to set the newly created `Constraints` object to the work request.

4. In the `DevByteApplication` class, inside the `setupRecurringWork()` method, set the `Constraints` object to the periodic work request, `repeatingRequest`. To set the constraints, add the [`setConstraints()`](https://developer.android.com/reference/androidx/work/WorkRequest.Builder.html#setConstraints%28androidx.work.Constraints%29) method above the `build()` method call.

```kotlin
val repeatingRequest = PeriodicWorkRequestBuilder<RefreshDataWorker>(15, TimeUnit.MINUTES)
        .setConstraints(constraints)
        .build()
```

#### Step 2: Run the app and notice the logs

In this step, you run the app and notice the constrained work request being run in the background at intervals.

1. Uninstall the app from the device or emulator to cancel any previously scheduled tasks.
2. Open the **Logcat** pane in Android Studio. In the **Logcat** pane, clear the previous logs by clicking the **Clear logcat** icon on the left. Filter on `work`.
3. Turn off the Wi-Fi in the device or emulator, so you can see how constraints work. The current code sets only one constraint, indicating that the request should only run on an unmetered network. Because Wi-Fi is off, the device isn't connected to the network, metered or unmetered. Therefore, this constraint will not be met.
4. Run the app and notice the **Logcat** pane. The `WorkManager` schedules the background task immediately. Because the network constraint is not met, the task is not run.

```
11:31:44 D/DevByteApplication: Periodic Work request for sync is scheduled
```

5. Turn on the Wi-Fi in the device or emulator and watch the **Logcat** pane. Now the scheduled background task is run approximately every 15 minutes, as long as the network constraint is met.

```
11:31:44 D/DevByteApplication: Periodic Work request for sync is scheduled
11:31:47 D/RefreshDataWorker: Work request for sync is run
11:31:47 I/WM-WorkerWrapper: Worker result SUCCESS for Work \[...\]
11:46:45 D/RefreshDataWorker: Work request for sync is run
11:46:45 I/WM-WorkerWrapper: Worker result SUCCESS for Work \[...\] 
12:03:05 D/RefreshDataWorker: Work request for sync is run
12:03:05 I/WM-WorkerWrapper: Worker result SUCCESS for Work \[...\] 
12:16:45 D/RefreshDataWorker: Work request for sync is run
12:16:45 I/WM-WorkerWrapper: Worker result SUCCESS for Work \[...\] 
12:31:45 D/RefreshDataWorker: Work request for sync is run
12:31:45 I/WM-WorkerWrapper: Worker result SUCCESS for Work \[...\] 
12:47:05 D/RefreshDataWorker: Work request for sync is run
12:47:05 I/WM-WorkerWrapper: Worker result SUCCESS for Work \[...\] 
13:01:45 D/RefreshDataWorker: Work request for sync is run
13:01:45 I/WM-WorkerWrapper: Worker result SUCCESS for Work \[...\]
```

#### Step 3: Add more constraints

In this step, you add the following constraints to the `PeriodicWorkRequest`:

* Battery not low.
* Device charging.
* Device idle; available only in API level 23 (Android M) and higher.

Implement the following in the `DevByteApplication` class.

1. In the `DevByteApplication` class, inside the `setupRecurringWork()` method, indicate that the work request should run only if the battery is not low. Add the constraint before the `build()` method call, and use the [`setRequiresBatteryNotLow()`](https://developer.android.com/reference/androidx/work/Constraints.Builder#setRequiresBatteryNotLow%28boolean%29) method.

```kotlin
.setRequiresBatteryNotLow(true)
```

2. Update the work request so it runs only when the device is charging. Add the constraint before the `build()` method call, and use the [`setRequiresCharging()`](https://developer.android.com/reference/androidx/work/Constraints.Builder#setRequiresCharging%28boolean%29) method.

```kotlin
.setRequiresCharging(true)
```

3. Update the work request so it runs only when the device is idle. Add the constraint before the `build()` method call, and use [`setRequiresDeviceIdle()`](https://developer.android.com/reference/androidx/work/Constraints.Builder#setRequiresDeviceIdle%28boolean%29) method. This constraint runs the work request only when the user isn't actively using the device. This feature is only available in Android 6.0 (Marshmallow) and higher, so add a condition for SDK version `M` and higher.

```kotlin
.apply {
   if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
       setRequiresDeviceIdle(true)
   }
}
```

Here is the complete definition of the `constraints` object.

```kotlin
val constraints = Constraints.Builder()
       .setRequiredNetworkType(NetworkType.UNMETERED)
       .setRequiresBatteryNotLow(true)
       .setRequiresCharging(true)
       .apply {
           if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
               setRequiresDeviceIdle(true)
           }
       }
       .build()
```

4. Inside the `setupRecurringWork()` method, change the request interval back to once a day.

```kotlin
val repeatingRequest = PeriodicWorkRequestBuilder<RefreshDataWorker>(1, TimeUnit.DAYS)
       .setConstraints(constraints)
       .build()
```

Here is the complete implementation of the `setupRecurringWork()` method, with a log so you can track when the periodic work request is scheduled.

```kotlin
private fun setupRecurringWork() {

       val constraints = Constraints.Builder()
               .setRequiredNetworkType(NetworkType.UNMETERED)
               .setRequiresBatteryNotLow(true)
               .setRequiresCharging(true)
               .apply {
                   if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                       setRequiresDeviceIdle(true)
                   }
               }
               .build()
       val repeatingRequest = PeriodicWorkRequestBuilder<RefreshDataWorker>(1, TimeUnit.DAYS)
               .setConstraints(constraints)
               .build()
       
       Timber.d("Periodic Work request for sync is scheduled")
       WorkManager.getInstance().enqueueUniquePeriodicWork(
               RefreshDataWorker.WORK_NAME,
               ExistingPeriodicWorkPolicy.KEEP,
               repeatingRequest)
   }
```

5. To remove the previously scheduled work request, uninstall the DevBytes app from your device or emulator.
6. Run the app, and the `WorkManager` immediately schedules the work request. The work request runs once a day, when all the constraints are met.
7. This work request will run in the background as long as the app is installed, even if the app is not running. For that reason, you should uninstall the app from the phone.

Great Job! You implemented and scheduled a battery-friendly work request for the daily pre-fetch of videos in the DevBytes app. `WorkManager` will schedule and run the work, optimizing the system resources. Your users and their batteries will be very happy.

***

### Summary

#### Offline Caching

* [Caching](https://searchstorage.techtarget.com/definition/cache) is a process of storing data fetched from a network on a device's storage. Caching lets your app access data when the device is offline, or if your app needs to access the same data again.
* The best way for your app to store structured data on a device's file system is to use a local SQLite database. `Room` is an SQLite object-mapping library, meaning that it provides an abstraction layer over SQLite. Using `Room` is the recommended best practice for implementing offline caching.
* A repository class isolates data sources, such as `Room` database and web services, from the rest of the app. The repository class provides a clean API for data access to the rest of the app.
* Using repositories is a recommended best practice for code separation and architecture.
* When you design an offline cache, it's a best practice to separate the app's network, domain, and database objects. This strategy is an example of [separation of concerns](https://en.wikipedia.org/wiki/Separation_of_concerns).
* To add offline-support to an app, add a local database using `Room`. Implement a repository to manage and access the `Room` database. In the `ViewModel`, fetch and display the data directly from the repository instead of fetching the data from the network.
* Use a database refresh strategy to insert and update the data in the local database. In a database refresh, the local database is updated or refreshed to keep it in sync with data from the network.
* To update your app's UI data automatically when the data in the database changes, use observable queries with a return value of type [`LiveData`](https://developer.android.com/reference/androidx/lifecycle/LiveData.html) in the DAO. When the `Room` database is updated, `Room` runs the query in background to update the associated `LiveData`.


#### WorkManager

* The [`WorkManager`](https://developer.android.com/topic/libraries/architecture/workmanager.html) API makes it easy to schedule deferrable, asynchronous tasks that must be run reliably.
* Most real-world apps need to perform long-running background tasks. To schedule a background task in an optimized and efficient way, use `WorkManager`.
* The main classes in the `WorkManager` library are [`Worker`](https://developer.android.com/reference/androidx/work/Worker.html), [`WorkRequest`](https://developer.android.com/reference/androidx/work/WorkRequest.html), and [`WorkManager`](https://developer.android.com/reference/androidx/work/WorkManager.html).
* The `Worker` class represents a unit of work. To implement the background task, extend the `Worker` class and override the [`doWork()`](https://developer.android.com/reference/androidx/work/Worker.html#doWork%28%29) method.
* The `WorkRequest` class represents a request to perform a unit of work. `WorkRequest` is the base class for specifying parameters for work that you schedule in `WorkManager`.
* There are two concrete implementations of the `WorkRequest` class: [`OneTimeWorkRequest`](https://developer.android.com/reference/androidx/work/OneTimeWorkRequest.html) for one-off tasks, and [`PeriodicWorkRequest`](https://developer.android.com/reference/androidx/work/PeriodicWorkRequest.html) for periodic work requests.
* When defining the `WorkRequest`, you can specify [`Constraints`](https://developer.android.com/reference/androidx/work/Constraints.html) indicating when the `Worker` should run. Constraints include things like whether the device is plugged in, whether the device is idle, or whether Wi-Fi is connected.
* To add constraints to the `WorkRequest`, use the set methods listed in [the `Constraints.Builder` documentation](https://developer.android.com/reference/androidx/work/Constraints.Builder#public-methods_1). For example, to indicate that the `WorkRequest` should not run if the device battery is low, use the [`setRequiresBatteryNotLow()`](https://developer.android.com/reference/androidx/work/Constraints.Builder#setRequiresBatteryNotLow%28boolean%29) set method.
* After you define the `WorkRequest`, hand off the task to the Android system. To do this, schedule the task using one of the `WorkManager` [`enqueue` methods](https://developer.android.com/reference/androidx/work/WorkManager.html#enqueue%28androidx.work.WorkRequest%29).
* The exact time that the `Worker` is executed depends on the constraints that are used in the `WorkRequest`, and on system optimizations. `WorkManager` is designed to give the best possible behavior, given these restrictions.
