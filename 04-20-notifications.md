## Notifications & Settings

### Admin [5-10min]
Homework check-in
- Todoer got turned in (or getting turned in with a late day?)
  - That was in many respects "the big one" for assignments, in that it combined probably the most new concepts of any of the apps we'll make. But it's a real app!
- This week's is an SMS app (or pieces of one). It's another pretty broad assignment that integrates a lot of different concepts (including things we've started working with like Fragments and ContentProviders).
  - But you can do this one in pairs! About making a messaging client, so nice to be able to send messages to another person :)
  - But after that, we'll become slightly more focused. as we get into the final project
- Note that project details are online if you want to check them out. You might start brainstorming about the project you want to do, but don't have to decide for another week or two.

Fork and clone the repo for today; working off of what we had finished yesterday!
  - We've added a couple menu buttons though: let's review how those are implemented!


### PendingIntents
Let's pick up where we left off from yesterday. We had implemented the `sendTextMessage()` method to send an SMS message. Looking at the <a href="http://developer.android.com/reference/android/telephony/SmsManager.html#sendTextMessage(java.lang.String, java.lang.String, java.lang.String, android.app.PendingIntent, android.app.PendingIntent)">documentation for this method</a>, you can see that this works by looking at the inbox in the Messages app... but there is another way as well. Those last two parameters are for [`PendingIntents`](http://developer.android.com/reference/android/app/PendingIntent.html): one for when messages are sent and one for when messages are delivered.
- What's a `PendingIntent`? The details are not _super_ readable... It's basically a wrapper around an `Intent` that we give to **another** class. Then when that class receives our `PendingIntent` and reacts to it, it can run the `Intent` (command) we sent it with as if that `Activity` was us (whew).
  - Basically we're saying "when I call you, you can come pick me up using my car" kind of thing.
  - Or like if you gave a stamped envelope to someone to put your letter or recommendation inside (do this!)
- So the idea is we specify what `Intent` should be delivered when the message is finished being sent (that `Intent` becomes "pending"). Effectively, this lets us send Intents in response to some other kind of event.

Let's go ahead and set one up:
```java
public static final String ACTION_SMS_STATUS = "edu.uw.intentdemo.ACTION_SMS_STATUS";
...
Intent intent = new Intent(ACTION_SMS_STATUS);
PendingIntent pendingIntent = PendingIntent.getBroadcast(MainActivity.this, 0, intent, 0);

smsManager.sendTextMessage("5554", null, "This is a test message!", pendingIntent, null);
```
We're doing a couple of steps here:
- We're defining out own custom Action. It's just a `String`, but name-spaced to avoid conflicts
- We then create an **implicit intent** for this action
- And then create a `PendingIntent`. We're using the `getBroadcast()` method to specify that the intent should be sent via a Broadcast (c.f. `getActivity()` for `startActivity()`).
  - First param is `content` that should send the intent, then a request code (e.g., for result callbacks if we wanted), then the `Intent`, and finally any extra flags (none for now).

We can then have our `BroadcastReceiver` respond to this `Intent` just like any other one!
```java
if(intent.getAction() == MainActivity.ACTION_SMS_STATUS) {
    if (getResultCode() == Activity.RESULT_OK) {
        Toast.makeText(context, "Message sent!", Toast.LENGTH_SHORT).show();
    }
    else {
        Toast.makeText(context, "Error sending message", Toast.LENGTH_SHORT).show();
    }
}
```
- **Don't forget** to add our custom intent to the `<intent-filter>`!

We'll see more with `PendingIntents` in a moment.


### Notifications
We can let the user know what's going on by popping up a toast or even an `AlertDialog`, but often we want to notify the user of something outside of the normal UI (e.g., when the app isn't running, or without getting in the way of stuff). To do this, we can use [Notifications](http://developer.android.com/guide/topics/ui/notifiers/notifications.html). These show up in the **notification area** (the icons at the top) and in the system's **notification drawer**, which the user can get to at any point (even when outside the app) by swiping down.

Android's documentation for UI components is overall pretty good (because Google wants to make sure people can build effective apps so the platform is worthwhile!) And because there are so many different UI elements and they change all the time, in order to do real-world Android development you need to be able to read, synthesize, and apply this documentation.
  - So for this morning, we're going to practice doing that! Everyone open up the **Notifications** page (google search "android notifications" and you'll find it). Let's walk through the process using Google's documentation, to get a sense for how we can follow their explanation.
  - OUR GOAL: when we hit the click menu button, we want to make a notification that says we did so, and how many times we've hit it!

Okay, looking at the documentation we see an overview. Also a link to the **Design Guide**, which is a good place to go to figure out how to design _effective_ notifications.
  - This is not a design class, but this is really good advice for Android development!

There is a lot of text about how to make a Notification... I personally prefer to work off of sample code, modifying until I have something that does what I want, so I'm going to scroll down **slowly** until I find an example I can copy/paste in, or at least reference.
- Then we can scroll back up to see how this woks!
- Hey look "Creating a Simple Notification". Sounds great!

The first part of this Notification is using `NotificationCompat.Builder` (use `v4`). Where else have we seen Builders? The `AlertBuilder`! Same idea here
  - I don't have a drawable resource that makes me want to comment out the icon... but scrolling back up I can see that it is _required_. So we can generate an Image Asset just like we did with icons (cause hey, notifications are a thing that happens!)

The next line makes an Intent. We've done that... but why are we creating an Intent? Well if we scroll up and look where where `Intent` is referenced, we can find out about Notification Actions (e.g., what happens when someone clicks the Notification). And since `Intents` are messages to open Activities, it makes sense that clicking a Notification might send an `Intent`, right?
- Notice that the `Intent` will actually be wrapped in a `PendingIntent`. So what we're going to do is send the Notification a `PendingIntent`, which contains an "RSVP" that it can send back to us when someone clicks on it.
- The idea being that the Notification "component" (which is not our Activity!) will be able to run this intent.. which wakes up our Activity! The Intent is "pending" delivery/activation by the other service.

The notification is using the a [`TaskStackBuilder`](http://developer.android.com/reference/android/support/v4/app/TaskStackBuilder.html) to [construct an "artificial" backstack](http://developer.android.com/guide/topics/ui/notifiers/notifications.html#NotificationResponse). The idea here is that if we click on the notification and jump to a "detail" view (say), we'd like to still be able to hit the back button and return to the "master" view.
- Or how when we launched the phone dialer, hitting back returned to the list of contacts.
- We build this backstack not just with methods, but by integrating with the "parent-child" relationship we've otherwise set up between Activities
  - In the `Manifest`, we had specified that `SecondActivity's` parent is `MainActivity`. This is what gave us the nice back button in the ActionBar. These sequence of `parentActivityName` attributes forms a hierarchy that will be the "back navigation hierarchy."
- We add the "end" of the hierarchy to the builder using `addParentStack(ResultActivity.class)`, and then finally put the `Intent` we actually want to use "on top" of the stack with `addNextIntent(resultIntent)`. The `resultIntent` is _not_ the `PendingIntent`... yet

Instead, we can have this `TaskStackBuilder` build us an appropriate `PendingIntent` (using the `getPendingIntent()` method).
- Pass it an _ID_ to refer to that request (like we've done before), and a flag `PendingIntent.FLAG_CURRENT_UPDATE` so that if we re-issue the pending intent it update instead of replacing.
- We can then assign that `Intent` to the _notification_ builder (with `setContentIntent`).

Finally, we can use the `NotificationManager` (similar to the `FragmentManager`, `SmsManager`, `BluetoothManager`, etc.) to fetch the _notification service_ (the "application" that handles all the notifications for the OS).
- We'll then tell this manager to actually issue the `Notification`
  - Other parameter: an `ID` number to refer to a particular notification. This is similar to the ID number we used for `Intent` results.

And Boom we have notifications :)
- We can click on that notification and get taken to our app! Which has a back stack!

We can also **update** this notification later, and it's really straightforward: we simply re-issue a notification with the same **id** number you assigned, and it will "replace" the previous one!
- Thus we can have our text be based on some instance variable, and have the Notification track the number of clicks!

You may notice that this notification doesn't ["pop up"](http://developer.android.com/guide/topics/ui/notifiers/notifications.html#Heads-up) in a way we might expect. This is because it's [priority](http://developer.android.com/guide/topics/ui/notifiers/notifications.html#Priority) isn't high enough (needs to be `NotificationCompat.PRIORITY_HIGH` or higher) **and** because it doesn't use either sound or vibration! (It needs to be _really important_ to get a heads-up pop).
- We can make it vibrate but using the <a href="http://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html#setVibrate(long[])">`setVibrate()`</a> method, passing it an array of times (in milliseconds) which to turn vibration on and off.
  - Pattern is `[delay, vibrate, sleep, vibrate, sleep, ...]`
  - May need permission? `<uses-permission android:name="android.permission.VIBRATE" />`
- We can also assign a default sound with `builder.setSound(Settings.System.DEFAULT_NOTIFICATION_URI);`
- See the [design guide](http://developer.android.com/design/patterns/notifications.html#correctly_set_and_manage_notification_priority) for best practices on priority.

As always, there are a number of other pieces/details we can specify, but I leave those to you to look up in the documentation :)


#### Design
Again, you may have noticed that what we did **not** talking about is the UI Design guidelines: e.g., what kind of text should I put in my notification? _When_ should I choose to use a notification? Android has lots of guidance on these questions in their "design" documentation, and further HCI and Mobile Design topics apply here just as well (has anyone taken the Mobile Design class? Does it cover this stuff?)
- Leaving the UI design up to you. But major guidelines apply (e.g., make actions obvious, give feedback, avoid irreversible actions, etc.). There are some notes about good interface design guidelines in the _final project_ write-up, since you'll need to have a good UI for that.

### Break 5mins

### SharedPreferences
Last piece I want to talk about today: what if we wanted to let the user decide whether the clicking should create notifications or not? Maybe sometimes they just want to see Toasts! The cleanest way to do this is to create some [Settings](http://developer.android.com/guide/topics/ui/settings.html) using `Preferences`.

[Shared Preferences](http://developer.android.com/guide/topics/data/data-storage.html#pref) are another way that we can **persist** data in application (besides doing something like putting it into a database via a ContentProvider).
- Shared Preferences stores _key-value pairs_ of primitives (Strings, ints, etc), similar to what we've been putting in Bundles. This data will be stored across application sessions: if I save some data to the Preferences and close the app, it will be there when I come back.
- Preferences are stored in an **XML File** (not a resource though!). So basically we put in lists of key-value paris in XML.
  - This is not great for structured data (since only stores key-value pairs), hence why we used `ContentProviders` and databases for more complex data. But for simple values? It's great.
- **Important note**: even though they are _called_ "Preferences", they not just for "user preferences". We can persist any small bits of primitive data in a Preferences file.

#### Accessing SharedPreferences
We can get access to this SharedPreferences file using the `.getSharedPreferences(String, int)` method.
- The String is going to be the name of the Preference File we want to access (so we can have multiple XML files). Just use `getPreferences()` to use a single default.
- The `int` is a flag about whether other apps should have access to that file. `MODE_PRIVATE` (0) is the default, `MODE_WORLD_READABLE` and `MODE_WORLD_WRITEABLE` are the other options.

We can edit this XML file by calling `.edit()` on the `SharedPreferences` object to get a `SharedPreferences.Editor`, which is basically our bundle-esque hashmap we can `put` values into.
- We need to call `.commit()` on the editor to save our changes!

Finally, we can just call `get` methods on the `SharedPreferences` object in order to fetch data out of it!
- Second param is a default value if preference doesn't exist! Yay!

[[Quick demo in `onStop()`/`notify` and `onCreate()`]]

### Preference Settings
While this is a generic data store, it's called `SharedPreferences` because it's most commonly used for "user preferences": think the "Settings" for an app.

The "Preference Menu" is a user-facing element, so we'll want to define it as an [XML resource](http://developer.android.com/guide/topics/ui/settings.html#DefiningPrefs). But we're not going to try and create our own layout: instead we're just going to define the list of [`Preferences`](http://developer.android.com/reference/android/preference/Preference.html) themselves as a resource!
- We can create a new resource using Android Studio. The "type" for this is actually just `XML` (generic), though our "root element" will be a `PreferenceScreen` (thanks intelligent defaults!)
- By convention, the preferences resource is named `preferences.xml`

Inside the `PreferenceScreen`, we add more elements: one to represent each "line" of the screen settings window (or preference we want to let the user adjust).
- We can define different types of `Preference` objects, such as `<CheckBoxPreference>`, `<EditTextPreference>`, `<SwitchPreference>`, or `<ListPreference>` (for a dialog of radio buttons). There are a couple of other options as well, see the [`Preference`](http://developer.android.com/reference/android/preference/Preference.html) base class.
- These elements should include (among others) the following XML attributes:
  - `android:key` the key to store the preference in the SharedPreferences file
  - `android:title` a user-visible name
  - `android:defaultvalue` guess :p (true or false for checked).
  - More options cam be found in the the [Preference](http://developer.android.com/reference/android/preference/Preference.html#lattrs) documentation.

We can also further divide these Preferences to organize them: we can place them inside a `PreferenceCategory` tag (with its own `title` and `key`) in order to group them together.

Finally we can specify that our Preferences have multiple screens by nesting `PreferenceScreen` elements. This produces "subscreens" (like submenus): when we click on the item it will take us to the next screen.
- Note that a cleaner (but more work) way to do this if you have _lots_ of settings is to use [`preference-headers`](http://developer.android.com/guide/topics/ui/settings.html#PreferenceHeaders) which allows for better multi-pane layouts... but since we're not making any apps with that many settings I'll leave it as exercise for the reader.

So we've got our Preferences all defined in XML: we just need to show them in our application! To do this, we're going to use the `PreferenceFragment` class (a specialized Fragment for showing lists of `Preference` objects).
- We don't need to specify an `onCreateView()` method, instead we're just going to load that `Preference` resource in the `onCreate()` method using `addPreferencesFromResource(R.xml.preferences)`. This will cause the `PreferenceFragment` to create the appropriate layout!

We'll put this Fragment inside a plain `Activity`, which just loads that Fragment via a Transaction
```java
getFragmentManager().beginTransaction()
                .replace(android.R.id.content, new SettingsFragment())
                .commit();
```
- And the Activity doesn't even need to load a layout: just specify a transaction!
- Note that `android.R.id.content` refers to the "root element" of the current View--basically what `setContentView()` is normally inflating into.
  - But if we want to include other stuff (e.g., an ActionBar), we'd need to structure the Activity and it's layout in more detail (particularly for multiple screens)
- Also note that there is a `PreferenceActivity` class as well, but **do not use it**.  Many of it's methods are deprecated, and since we're targeting later than Honeycomb we should use the Fragment.


Finally, how do we interact with these settings? Here's the trick: a `preferences` XML resource is **automatically** associated with a `SharedPreferences` file. And in fact, every time we adjust a setting in the `PreferenceFragment`, the values in that file are edited as well!
- So we never need to write to the file, just read from it!

The `preference` XML corresponds to the "default" `SharedPreferences` file, which we'll access via
```java
SharedPreferences sharedPref = PreferenceManager.getDefaultSharedPreferences(this);
```
- And then we have this object we can fetch data from with `getString()`, `getBoolean()`, etc.
- So we can check the preferences before we show a notification!

That's the basics of using Settings. For more details see the [documentation](http://developer.android.com/guide/topics/ui/settings.html), as well as the [design guide](http://developer.android.com/design/patterns/settings.html) for best practices on how to organize your settings.


### ShareActionProvider (if time?)
But wait there's more! One of the other things we can add to menus are [Action Views](http://developer.android.com/training/appbar/action-views.html) that are expandable widgets in the action bar (e.g., search example). Or, to play around with Intents more, we can add an [Action Provider](http://developer.android.com/training/appbar/action-views.html#action-provider) (like [`ShareActionProvider`](http://developer.android.com/training/sharing/shareaction.html)), which gives us a bunch of interaction built into the menu! This is the "quick share with these social media sites" button that we see commonly.
- We'd want to look at [class documentation](http://developer.android.com/reference/android/support/v7/widget/ShareActionProvider.html) for how to set this up (it's much clearer than the training docs).

How to use it:
- We're going to add another item to our menu's XML. This will look like most items, except it will have an extra field `app:actionProviderClass`
  ```xml
  <item
      android:id="@+id/menu_item_share"
      android:title="Share"
      app:showAsAction="ifRoom"
      app:actionProviderClass="android.support.v7.widget.ShareActionProvider"
      />
  ```
- We'll then add the item to our menu in `onCreateOptionsMenu()`
  ```java
  MenuItem item = menu.findItem(R.id.menu_item_share);
  mShareActionProvider = (ShareActionProvider) MenuItemCompat.getActionProvider(item);

  Intent intent = new Intent(Intent.ACTION_DIAL);
  intent.setData(Uri.parse("tel:206-685-1622"));

  mShareActionProvider.setShareIntent(intent);
  ```
  - We get access to the item using `findItem()`, and then cast it to a `ShareActonProvider` (make sure you're using the support version!)
  - We can then specify an _implicit Intent_ that we want that "Share Button" to be able to perform. This would commonly use the `ACTION_SEND` action (like for sharing a picture or text), but we'll use the `DIAL` action because we have a couple of dialers but don't actually have many `SEND` responders on the emulator.

The Menu item will then list a dropdown with all of the different Activities that `resolve` to handling that implicit intent!


### Review (if time?)
For the remainder of our time (if any), I think I'd like to open things up to work/review/questions. What aspects of things we've covered so far need review/practice? Are there any pieces of the homework you'd like to make sure and go over? What can I best do to help at the moment?
- And if there is nothing, happy to let you work on your own or go early :)

Next week: we're going to start doing location-based stuff, and dig into the Maps API! Because location is what mobile is all about :)


### Lecture References
- <a href="http://developer.android.com/reference/android/telephony/SmsManager.html#sendTextMessage(java.lang.String, java.lang.String, java.lang.String, android.app.PendingIntent, android.app.PendingIntent)">`sendTextMessage`</a>
- [Notifications](http://developer.android.com/guide/topics/ui/notifiers/notifications.html)
- [Priorities](http://developer.android.com/design/patterns/notifications.html#correctly_set_and_manage_notification_priority)
- [Shared Preferences](http://developer.android.com/guide/topics/data/data-storage.html#pref)
- [`Preference`](http://developer.android.com/reference/android/preference/Preference.html)
- [`ShareActionProvider`](http://developer.android.com/reference/android/support/v7/widget/ShareActionProvider.html)
