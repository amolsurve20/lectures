## Services

### Admin
- Homework check-in: hope you've gotten started / at least thought through ideas. Any questions about requirements/etc?
- Project check-in: everyone is in groups, and your first deliverable (the project proposal) is due **Wed afternoon**.
  - I recommend you get it in early; I'll try to keep an eye on the submissions so I can give you early feedback (you can then adjust and revise before it is finally "graded" if needed).
  - If there are any questions about project ideas, check in with me!
- Fork & clone repo for today, as always. Has some really basic starter code; we'll be adding in a lot more

### Services
This week we're going to cover some aspects of Android development that are _not_ immediately visible to users---more "back-end" components that are used to support your apps. Today, we'll look at the most general of these: [**Services**](http://developer.android.com/guide/components/services.html).

Google's definition:
> A Service is an application component that can perform long-running operations in the background and does not provide a user interface. Another application component can start a service and it will continue to run in the background even if the user switches to another application.... For example, a service might handle network transactions, play music, perform file I/O, or interact with a content provider, all from the background.

You can think of Services as sort of like "Activities" (components/modules) that don't have a user interface or user interaction directly tied to them. Services are launched by Activities, but then do their own thing in the background _even after the Activity is closed_ (this is distinct from `ASyncTask`, which had its lifecycle tied to that of its containing Activity). Once a Service is started, it keeps running until it is explicitly stopped (though it can be destroyed by the OS to save memory, similar to an Activity---though Services have higher "priority" and so aren't killed as readily).

Common uses for a Service:
- We want to download some data from the network, but want to let the user interact with our system while that is happening
- OR we want to _upload_ some data to the network, but don't want to lose that upload if the user leaves the app!
- OR we want to save to the database but don't want that to crash if the user leaves the app
- OR we want to run some other kind of "background" task that is long-running even if the app gets closed (music playing is a classic, and one we'll look at today).

*Important* things to note:
1. A Service is **not** a separate process; it runs in the same process as the app that starts it (unless otherwise specified)
2. A Service is **not** a Thread, and in fact doesn't need to run outside the UI thread! However, we quite often _do_ want to run the service outside of the Main Thread, and so often have it spawn and run a new Runnable Thread (similar to what we did with the `SurfaceView` last week, and what we've done in Java).
  - When we say "in the background" we mean from the user's perspective, not "on a background thread" per se.


### IntentService
To create a service, we're going to subclass [`Service`](http://developer.android.com/reference/android/app/Service.html), just like we've done with `Activity`, `Fragment`, `BroadcastReceiver`, and everything else, and then override some _lifecycle callbacks_ (again, just like we did with `Activity`).
- Services really are put together like Activities that run without a user interface ("in the background").
- Again, when we subclass `Service` it's important that we make our own background thread so that we don't block the Main thread!

Because making a Service that does some (single) task in a background thread is so common, Android actually includes a Service subclass we can use to do exactly this work. This class is called [`IntentService`](http://developer.android.com/reference/android/app/IntentService.html)&mdash;a service that responds to `Intents` and does some work in response to them.
- We're going to start with `IntentService` because it's simpler, and then move into the more generic, complex version.
- This does similar work to `ASyncTask`, with the advantage that it will keep doing that work even if the Activity is destroyed!
- `IntentService` will listen to any incoming "requests" (`Intents`) and "queue" them up, handling each one at a time. Once it is out of tasks to do, the service will shut itself down to save memory (it will restart if more `Intents` are sent to it). Basically, it handles a lot of the setup and cleanup on its own!


We start off by making a new class (e.g., `NumberService`) that subclasses `IntentService`
- We need to provide a default _constructor_ that can call `super(String debugName)`
- We need to provide the `onHandleIntent(Intent)` method. Incoming `Intents` wait their turn in line, and then each is delivered to this method in turn. Note that all this work (delivery and execution) will happen in a **background thread** supplied by `IntentService`
  - For example, we can log out some information and then have the service pause for a few seconds:
  ```java
  for(int count=0; count<=10; count++){
      Log.v(TAG, "Count: "+count);
      try {
          Thread.sleep(5000); //sleep for 5 seconds
      } catch (InterruptedException e) {
          Thread.currentThread().interrupt();
      }
  }
  ```

Just like with Activities, we also need to declare the `<service>` in the Manifest:
```xml
<service android:name=".CountingService" />
```

Finally, we can send an `Intent` to the Service by using the `startService()` method (similar to `startActivity()`, but for services!)
- We can put an Extra in the Intent if we want to keep track of them.

And now when our service starts, we can see it start counting (but without blocking). We can also kill the Activity and see that it keeps running!

(Note, if we want to get `Toasts` to work, we need to make sure they run on the UI Thread. We can do this using a [`Handler`](http://developer.android.com/reference/android/os/Handler.html), which is an object used to pass messages between Threads.
```java
mHandler.post(new Runnable() {
    @Override
    public void run() {
        Toast.makeText(MyService.this, "Count: " + count, Toast.LENGTH_SHORT).show();
        Log.v(TAG, "" + count);
    }
});
```

### Service Lifecycle
Now that we have a basic example, let's talk about what is going on under the hood: specifically, let's start with the [Service lifecycle](http://developer.android.com/guide/components/services.html#LifecycleCallbacks). Note that there are two "models" of Services: unbound (like what we just did) and "bound" services (in the sense that "client" Activities are "bound" to the service to interact with it; we'll do this more in a bit).
- Services have an `onCreate()` method that works just like Activities. We don't have a UI to set up, so we don't often do a lot in here. The `IntentService` constructor achieves a lot of the same results.
- The big important callback is called `onStartCommand()`. This is the method called when the service is **sent a command** by another application. It's a bit like the `doInBackground()` method of `ASyncTask`.
  - This is not when the service is started necessarily, but when it receives a command to start (even if its already running!)
  - Usually gets called when we use `startService()`
  - In our `IntentService`, a received intent is then used to call the _helper_ `onHandleIntent()` method. The handle method is not itself part of the lifecycle; that's a special piece of `IntentService`.
- (`onBind()` and `onUnbind` are for bound services, more in a moment)
- And we have `onDestroy()` which again is like an Activity's method
  - Note that in general Services have to be manually told to **stop**. We do this by calling `stopService(Intent)` to send it a "stop" intent, or by calling `stopSelf()` inside the Service to have the service stop itself.
  - _Important_: `IntentService` will finish up handling any Intents that are currently "running", but anything else that is queued will be removed.

We can override these if we want to see them in action, though `IntentService` already took care of them for us:
- `onCreate()` it sets up the "message queue" so that it can queue up Intents to run one at a time
- `onStartCommand()` queues up the incoming intent, and then send it along to the `onHandleIntent()` method when available
  - We can adjust the count to 3 and send multiple items to the queue: watch how they are handled (or can be stopped by us!)
- Once there are no more tasks to do, `IntentService` calls `stopSelf()` (thereby executing `onDestroy()`).
- (It also has a default `onBind()` that returns `null`)

We can fill in some of these callbacks to see how they are getting executed:
```java
public int onStartCommand(Intent intent, int flags, int startId) {
  Toast.makeText(this, "Intent received", Toast.LENGTH_SHORT).show();
  return super.onStartCommand(intent,flags,startId);
}
```

To finish up the lifecycle, let's talk about that `int` that is returned by `onStartCommand`. This is a flag that indicates how the Service should behave when it is "restarted" after having been destroyed:
- [`START_NOT_STICKY`](http://developer.android.com/reference/android/app/Service.html#START_NOT_STICKY): "If the system kills the service after `onStartCommand()` returns, do not recreate the service, unless there are pending intents to deliver. This is the safest option to avoid running your service when not necessary and when your application can simply restart any unfinished jobs."
- [`START_STICKY`](http://developer.android.com/reference/android/app/Service.html#START_STICKY): "If the system kills the service after `onStartCommand()` returns, recreate the service and call onStartCommand(), but do not redeliver the last intent. Instead, the system calls `onStartCommand()` with a null intent, unless there were pending intents to start the service, in which case, those intents are delivered. This is suitable for media players (or similar services) that are not executing commands, but running indefinitely and waiting for a job."
- [`START_REDELIVER_INTENT`](http://developer.android.com/reference/android/app/Service.html#START_REDELIVER_INTENT): "If the system kills the service after onStartCommand() returns, recreate the service and call onStartCommand() with the last intent that was delivered to the service. Any pending intents are delivered in turn. This is suitable for services that are actively performing a job that should be immediately resumed, such as downloading a file."

In other words: services may get killed, but we can specify how they get resurrected!
- And of course, we could return different values for different started commands.

#### Break?

### Media Player!
One of the classic uses for a background service is to play music. Since audio is relevant to this week's homework, let's use that as an example (even though background audio isn't sufficient for your app). But before we get into making a music service, we need to do a quick aside about playing music with [`MediaPlayer`](http://developer.android.com/guide/topics/media/mediaplayer.html)
1. Last week we played sound using the `SoundPool` API. Android actually has three (3) audio APIs! `SoundPool` is great for managing short sound effects that play simultaneously, though you need to do extra work to load them ahead of time.
2. [`AudioTrack`](http://developer.android.com/reference/android/media/AudioTrack.html) allows you to play audio at a very low level (e.g., by "pushing" bytes to a stream). This is useful for generated Audio (like MIDI music) or if you're trying to do some other kind of low-level stuff.
3. The third API really should be the first, in that it's the most common and the one recommended by the docs. It's called `MediaPlayer`, and is the main API for playing sound and video (e.g., if you want to play `.mp3` files).

`MediaPlayer` is actually ridiculously easy to use, particularly if we're playing a locally defined resource (e.g., something in `res/raw/`). You simply use a factory to make a new `MediaPlayer` object and then call `.play()` on it:
```java
MediaPlayer mediaPlayer = MediaPlayer.create(context, R.raw.sound_file);
mediaPlayer.start(); // no need to call prepare(); create() does that for you
```

We can also call `.pause()` to pause the music, `.seekTo()` to jump to a particular millisecond, and `.stop()` to stop the music.
- Note that when we `stop()`, we also need to release any resources used by the `MediaPlayer` (to free up memory):
```java
mediaPlayer.release();
mediaPlayer = null;
```

Note that we can also use `MediaPlayer` to play files from a `ContentProvider` or even off the Internet!
```java
String url = "http://........"; // your URL here
MediaPlayer mediaPlayer = new MediaPlayer();
mediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);
mediaPlayer.setDataSource(url);
mediaPlayer.prepare(); // might take long! (for buffering, etc);
//use .prepareAsync() and implement OnPreparedListener() to handle when finished
//mediaPlayer.setOnPreparedListener(this);
mediaPlayer.start();
```

- We can also implement and register `MediaPlayer.OnCompletionListener` so that our Service stops when the song is done.


### A Music Service
So this works all good and well, and in fact will even keep running as long as the Activity is alive. But remember that Activities are fragile, and can be destroyed at any moment (including by us!). So in order to keep our music playing even as we go about other tasks, we should [use a Service](http://developer.android.com/guide/topics/media/mediaplayer.html#mpandservices)
- Services have higher priority so don't get destroyed as readily!

For this we're going to subclass the big guy himself, `Service`, and set up all the pieces (there aren't really many).
- `onCreate()` we can include, though it doesn't need to do anything (can log that service starts)
- `onStartCommand()` should create and start our MediaPlayer. We can return `NOT_STICKY` so the music doesn't start up again randomly.
  - note we can create a new background thread to hold the MediaPlayer to avoid any blocking issues, though this isn't covered in the [guide](http://developer.android.com/guide/topics/media/mediaplayer.html), and StackOverflow seems to disagree.
  - We'll put this in a separate helper method, so we can access it later
- `onDestroy()` should clean up and release the MediaPlayer
- (If we want to handle pausing, we can specify that in the intent we send to the service, like as a custom ACTION, or we can use service binding as we'll demo in a moment).

Don't forget to include it in the `Manifest`!!

We can now have our Activity `startService()` and `stopService()` in order to control our media in the "background", even if we leave the Activity!

#### Foreground Services
Services are normally "background" tasks, that run without any UI and that the user isn't aware of (e.g., for downloading or uploading data, etc). But music playing is definitely something that the user is aware of, and in fact may want to interact with it! But we'd still like to keep it divorced from the Activity so that it can run without the Activity being active.

To do this, we use what is called a [**Foreground Service**](http://developer.android.com/guide/components/services.html#Foreground). Foreground services represent services that are divorced from Activities (they _are_ Services after all), but the user is aware of and so have much higher survival priority if the OS gets low on memory.
- Foreground services require a **Notification** in the status bar, similar to what we've used before. This will effectively act as the "user interface" for the service, and let the user see and be aware of its execution.

We create this notification inside the Service's `onStartCommand()` method, then pass it to `startForeground()` in order to put our service in the foreground:

```java
String songName = "The Entertaininer";

PendingIntent pendingIntent = PendingIntent.getActivity(getApplicationContext(), 0,
        new Intent(getApplicationContext(), MainActivity.class), PendingIntent.FLAG_UPDATE_CURRENT);

Notification notification = new NotificationCompat.Builder(this)
        .setSmallIcon(android.R.drawable.ic_media_play)
        .setContentTitle("Music Player")
        .setContentText("Now playing: "+songName)
        .setContentIntent(pendingIntent)
        .setOngoing(true) //cannot be dismissed by the user
        .build();
startForeground(NOTIFICATION_ID, notification); //make this a foreground service!
```
- Add icon, content, text as before.
- We give the notification a `PendingIntent` to run when selected, which can just open up our `MainActivity` so we can control the player.
- Set the notification to be ongoing!
- **Note** that once we are done doing foreground work, we should call `stopForeground(true)` to get rid of the foreground service (e.g., in `onDestroy()`)

There are a couple of other details that you should handle if you're making a full-blown music player app, including: keeping the phone from going to sleep, playing other audio at the same time (e.g., notification sounds; ringtones), switching to external speakers, etc. See the [guide](http://developer.android.com/guide/topics/media/mediaplayer.html) for more details (we'll not go over them here since pretty straightforward and not immediately relevant to the topics at hand).


### Bound Service
I mentioned before that there are two "types" of Services: **started services** (like what we've done _both_ with `IntentService` and `Service`, in that we launched it with `startService()`), and **bound services** (that used that `onBind()` callback). A [Bound Services](http://developer.android.com/guide/components/bound-services.html) is a service that acts as the "server" in a client-server setup: it allows for Activities to "connect" to it (___bind it___) and then send/receive messages to it (e.g., call methods on it). These messages can even be _across processes_, allowing for [interprocess communication](https://en.wikipedia.org/wiki/Inter-process_communication)! This is useful when you want interact with the service from an Activity in some way (e.g., if we want to `pause()` our music), or if we want to make the service's capabilities available to other applications.

We make a Bound Service by having that service implement the `onBind()` callback. This method returns an `IBinder` object (the `I` indicates that it's an _Interface_; enterprise-level Java convention). When Activities connect to this service (using the `bindService()` method), this `IBinder` object is given to them, and they can use it to get access to the Service process to call methods on.
- Effectively, the binding process produces an _object_ that represents that service, so we can call methods on it without needing to send it Intents.

Let's add the ability to "pause" the music as a bound service.

(Following the example in the guide, let's make our service also provide random numbers. Like a [Lava Lamp](https://en.wikipedia.org/wiki/Lavarand)! We'll then hook up the media player).
- Make a new `Random() object`, and fill in a `getRandomNumber()` method.

The first thing we need to do is have our Service implement the `onBind()` method... so we're going to need an `IBinder` to return. For "local services" (e.g., services that are run _in the same process_), the easiest way to get an `IBinder` is to extend the `Binder` class. We can provide a `getService()` method to return a copy of the service:
```java
private final IBinder mBinder = new LocalBinder(); //singleton instance variable
public class LocalBinder extends Binder {

}
```
- Then we just need to return this object from `onBind()`.

This `IBinder` class can do a couple of different things: We need to make _our own class_ to do this, which do a couple of different things:
1. It can have public methods in it that we call to talk to the service--really acts as a "public interface"
  ```java
  public String getSongName() {
      return songName;
  }
  ```
2. It can give us access to the service itself (return the `Service` object) that we can call methods on:
  ```java
    public MusicService getService() {
        // Return this instance of this Service so clients can call public methods on it!
        return MusicService.this;
    }
  ```
3. It can give us some other object "owned" by the service that we can call methods on (good for if we want to expose only part of the service).

In the Activity, we need to do a bit more work:
- We bind a service using <a href="http://developer.android.com/reference/android/content/Context.html#bindService(android.content.Intent, android.content.ServiceConnection, int)">`bindService(Intent, ServiceConnection, flag)`</a>. We can do this whenever we want the service to be available (`onStart()` is clean, so we can fetch the song name).
  - Intent points at what service we want to bind
  - (last param) flag can be `Context.BIND_AUTO_CREATE` as a good default; this will create the Service object as needed.
    - Note this **doesn't** call `onStartCommand()`, but creates and gives us access to the service
    - Bound services are not considered "started", and are kept around as long as they are bound. But if we "start" a service, we also need to remember to stop it!
- The second parameter is an object that implements `ServiceConnection` to be able to handle the connection to our Service (e.g., the callback). We can implement this ourselves, or make a separate anonymous class:
  ```java
  /** Defines callbacks for service binding, passed to bindService() */
  private ServiceConnection mConnection = new ServiceConnection() {
      public void onServiceConnected(ComponentName className, IBinder service) {
          // We've bound to LocalService, cast the IBinder and get LocalService instance
          LocalBinder binder = (LocalBinder) service;
          mService = binder.getService();
          mBound = true;
      }
      public void onServiceDisconnected(ComponentName arg0) {
          mBound = false;
      }
  };
  ```
  - the `onServiceConnected()` callback handles the actual connection, and is where we get access to that `IBinder`. We can use it to fetch the Service object to call methods on, saving that as an instance variable to do stuff to it!
- Finally, if we're bound, we can call methods on the service (which must exist!)--such as in the "pause" button method.

- Note that we could also `unbindService()` when the activity stops:
```java
protected void onStop() {
    if (mBound) {
        unbindService(this);
    }
    super.onStop();
}
```

Note that this is still just a _local service_. If we want to make the service be available to other processes, we'd need to do more work. We use a `Messenger` and `Handler` object to pass messages between those processes (like what we did for notifying between threads). For more details, see the example [in the guide](http://developer.android.com/guide/components/bound-services.html#Messenger), as well as the following complete sample classes ([`MessengerService`](https://github.com/android/platform_development/blob/master/samples/ApiDemos/src/com/example/android/apis/app/MessengerService.java), [`MessengerServiceActivities`](https://github.com/android/platform_development/blob/master/samples/ApiDemos/src/com/example/android/apis/app/MessengerServiceActivities.java)).
- We can look over these together if time

That about covers basic services, which you may want to include as part of your final project (e.g., if you want to do data upload/download, or have something else run in the "background" while your Activities are closed).


#### Lecture references
- [Services](http://developer.android.com/guide/components/services.html)
- [Service Lifecycle](http://developer.android.com/images/service_lifecycle.png)
- [`AudioTrack`](http://developer.android.com/reference/android/media/AudioTrack.html)
- <a href="http://developer.android.com/reference/android/content/Context.html#bindService(android.content.Intent, android.content.ServiceConnection, int)">`bindService(Intent, ServiceConnection, flag)`</a>
