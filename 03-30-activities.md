# 03-30 Activities

### Admin [5min]
- Day 2!
- Check in on homework: looks like pretty much everyone has access to the GitHub organization?
  - My assumption is you're comfortable with git to fork/clone and submit the homework. IF NOT, please let us know and we're happy to go over it outside of class.
- Has everyone worked out kinks with getting Android Studio running on your machines?
  - I know we don't have a lot of room, but you should bring your laptop and devices (with a cable to connect them!) to class. We'll be doing a lot of live coding, and the best way to learn from this is to follow along.
- Plan for today: Do a bit more pure Java to talk about _GUIs_ and _event-driven programming_. Then we'll jump into talking about `Activities` and how to write Java code for Android!
- Fork and clone the repo today for starter code! (This course assuming you are okay with doing this; if not swing by office hours and we can review it).


## GUI Patterns (Swing) [30mins]
Android applications are user-driven graphical applications. So to look at some of the _patterns_ involved in this kind of software (without the overhead of the Android framework), let's consider how to build simple graphical applications in Java using the [Swing library](https://docs.oracle.com/javase/tutorial/uiswing/start/)
- We're going to be doing a little bit of Java programming; easiest to just do this in Sublime Text or whatever light-weight IDE you have floating around.

The **Swing** library is a set of Java classes used to specify graphical user interfaces (GUIs). These classes can be found in the [`javax.swing`](https://docs.oracle.com/javase/8/docs/api/javax/swing/package-summary.html) package. They also rely on the [`java.awt`](https://docs.oracle.com/javase/8/docs/api/java/awt/package-summary.html) package (the "Advanced Windowing Toolkit"), which is an older GUI library that Swing builds on top of.
- Fun fact: Swing library is named after the dance style: the developers wanted to name it after something hip and cool and popular. In the mid-90s.

Let's look at an incredibly basic GUI class: `MyGUI` found in the `src/main/java/` folder. Things to note about this class:
- The class subclasses [`JFrame`](https://docs.oracle.com/javase/8/docs/api/javax/swing/JFrame.html). `JFrame` represents a "window" in your operating system, and does all the work of making that window show up and interact with the operating system in a normal way. By subclassing `JFrame`, we get that functionality for free! this is how we build all GUI applications.
  - We call the parent constructor (passing in the title for the window), and then specify what happens when we hit the "close" button.
- We then instantiate a [`JButton`](https://docs.oracle.com/javase/8/docs/api/javax/swing/JButton.html), which is a Java button
  - Note that `JButton` is the Swing version of a button, building off of the older `java.awt.Button` class.
- We then `.add()` this button to the `JFrame`. This puts the button inside the Window. A little but like using jQuery to add some HTML to the page.
- Finally, we call `.pack()` to tell the Frame to resize itself to fit the contents, and then `.setVisible()` to make it actually appear.
- We run this program from `main` by just instantiating our specialized `JFrame`, which will contain the button.

We can compile and run it with `./gradlew -q run`. And voila, we have a basic button!

#### Events
If we click the button... nothing happens. Let's make it print out a message when clicked. We can do this through **event-based programming** (if you remember `onClick` events from JavaScript, this is the same idea).

The computer sees interactions with its GUI as a series of **events**. The event of clicking a button. The event of moving the mouse. The event of closing a window. Etc. Each thing you interact with _generates_ and _emits_ these events. So when you click on a button, it creates and emits an "I was clicked!" event.
- You can think of this like the button shouting "Hey hey! I was pressed!" We can then respond to this shouting to have our program do something when the button is clicked.

Events, like everything else in Java, are Objects (of the [`EventObject`](https://docs.oracle.com/javase/8/docs/api/java/util/EventObject.html) type) that are created by the emitter. A `JButton` in particular emits [`ActionEvents`](https://docs.oracle.com/javase/8/docs/api/java/awt/event/ActionEvent.html) when pressed (the "action" being that it was pressed).
- When buttons are pressed, they shout out `ActionEvents`

In order to respond to this shouting, we need to "listen" for these events. Then whenever we hear that there is an event happening, we can react to it.
- This is like a person manning a submarine radar. Or hooking up a baby monitor. Or following someone on Twitter, in a way.

But this is Java, so we need an object to listen for these events: a "listener" if you will. Luckily, Java provides a type that can listen for `ActionEvents`: [`ActionListener`](https://docs.oracle.com/javase/8/docs/api/java/awt/event/ActionListener.html). This type has an `actionPerformed()` method that can be called in response to an event.
- We use the [Observer Pattern](https://sourcemaking.com/design_patterns/observer) to connect this listener object to the button (`button.addActionListener(listener)`). This _registers_ the listener, so that the Button knows who to shout at when something happens. Again, like following someone on Twitter. When the button is pressed, it will go to any listeners registered with it and call their `actionPerformed()` methods, passing in the `ActionEvent` it generated.

Look again: is `ActionListener` a class? No, it's an interface! Which means if we want to make an `ActionListener` object, we need to create a class that implements this interface (and provides the `actionPerformed()` method that can be called when the event occurs). There are a few ways we can do this:

1. Well we already have a class we're developing: `MyGUI`! So let's just make _that_ class `implement ActionListener`. We'll fill in the provided method, and then specify that `this` object is the listener, and voila.
  - This is my favorite way to create listeners in Java (since it keeps everything self-contained: the `JFrame` handles the events its buttons produce).
  - We'll utilize a variant of this pattern in Android: we'll make classes implement listeners, and then "register" that listener somewhere else in the code (often in a nested class).

2. But what if we want to reuse our listener across different classes, but don't want to have to create a new `MyGUI` object to listen for a button to be clicked? We can instead use an **inner** or **nested** class. For example, a nested class `MyActionListener` that implements the interface, and then just instantiate one of these to register with the button.
  - This could be a `static` nested class, but then we can't have it access instance variables (because it belongs to the _class_, not the _object_). So might make it an inner class. Of course then we can't re-use it elsewhere without making the `MyGUI`... but at least we've organized the functionality a bit.

3. It seems sort of silly to create a whole new `MyActionListener` class that has one method and is just going to be instantiated once. So what if instead of giving it a name, we just made it an [**anonymous class**](https://docs.oracle.com/javase/tutorial/java/javaOO/anonymousclasses.html)?
  - Like how we've made _anonymous variables_ by instantiating objects without assigning them to named variables... we can do the same thing with a class that just implements an interface. The syntax looks like:
    ```java
    button.addActionListener(new ActionListener() {
      //class declaration goes here!

      public void actionPerformed(ActionEvent) { /*...*/}
    });
    ```
  - This is how buttons are often used in Android: we'll create an anonymous listener object to respond to the event that occurs when they are pressed.

Questions on this kind of event-driven programming?

#### Layouts and Composites [end by 11:10]
One more piece if we have time: what if we want to add a second button? If we try to just `.add()` another button... it replaces the one we just had! This is because Java doesn't know _where_ to put the second button. Below? Above? Left? Right?

In order to have the `JFrame` contain multiple components, we need to specify a [**layout**](https://docs.oracle.com/javase/8/docs/api/java/awt/LayoutManager.html), which knows how to organize items that are added to the Frame. We do this with the `.setLayout()` method. For example, we can give the frame a `BoxLayout()` with a `PAGE_AXIS` orientation to have it lay out the buttons in a vertical row.
- Java has different `LayoutManagers` that each have their own way of organizing components. We'll see this same idea in Android (and talk about Android's layout system in _a lot_ more detail next week)!

What if we want to do more complex layouts? Well we can use a different `LayoutManager`... but we can actually achieve a lot of flexibility simply by using _multiple containers_.

For example, we can make a `JPanel` object, which is basically an "empty" component. We can then add multiple buttons to this this panel, and add that panel to the `JFrame`. Because `JPanel` _is a_ `Component` (just like `JButton` is), we can use the `JPanel` exactly as we used the `JButton`---this panel just happens to have multiple buttons.

And since we can put any `Component` in a `JPanel`, and `JPanel` is itself a component... we can create nest these components together into a tree in an example of the [Composite Pattern](https://sourcemaking.com/design_patterns/composite). This allows us to create very complex user interfaces with just a simple `BoxLayout`!
- We'll see this kind of "nested" composite pattern in Android next week!

## Activities
Now that we've gotten a feel for the pattern/coding style of creating graphical applications, let's see how they play out in Android!
- You'll need to create a new `Android` application to play with, with a single **Empty** Activity (e.g., `MainActivity`). In the future I'll have starter code for you to work from, but since we're starting from scratch it makes sense to start with something empty. Plus: it's good practice!
  - We can also take a **2-minute** stretch break while you get everything up and running.

### What is an Activity? [10min]
What is an [Activity](http://developer.android.com/guide/components/activities.html)? According to Google:

> An Activity is an application component that provides a screen with which users can interact in order to do something

You can think of an Activity as a single Screen in your app. The equivalent of a "Window" in a GUI system (or a `JFrame` in our Swing app).
- Note that Activities don't __need__ to be full screens: they can also be floating modal windows, embedded inside other Activities (like half a screen), etc. But we'll start by thinking of them as full screens.
- In many ways, an Activity is a "bookkeeping mechanism": a place to hold _state_ and _data_, and tell to Android what to show on the display.
  - It functions much like a Controller (in Model-View-Controller sense) in that regard!
- We can have lots of Activities (screens) in an Application, and they are loosely connected so we can easily move between them.

Also to note from the documentation:

> An activity is a single, focused thing that the user can do.

which implies a design suggestion: Activities (screens) break up your App into "tasks": each one can represent what a user is doing at once! If the user does something else, that should be a different Activity and so probably a different screen.

#### Making Activities
How do we use them? We create our own activities by subclassing the provided [`Activity`](http://developer.android.com/reference/android/app/Activity.html) class.
- This is **inheritance**: we're making a specialized type of `Activity`, similar to extending `JFrame` in Swing apps
- All the methods that control how the OS interacts with Activities are provided for us.

If you look at the default Empty activity, it actually subclasses [`AppCompatActivity`](http://developer.android.com/reference/android/support/v7/app/AppCompatActivity.html), which is a already specialized kind of Activity that provides an [`ActionBar`](http://developer.android.com/reference/android/support/v7/app/ActionBar.html) (the toolbar at the top). If we change the class to just extend `Activity`, that bar disappears.
  - You'd need to import the `Activity` class! The keyboard shortcut inAndroid Studio is `alt+return`, or you can do it by hand (look up the package)!

There are a pile of other built-in Activity subclasses that we could subclass instead. We'll mention them as they become relevant.
- Many on the books have been deprecated in favor of **Fragments**, which are sort of like "subactivities" that get nested in larger Activities. We'll talk about Fragments more in a couple weeks, once we've gotten the basics down.

Other important point to note: does this activity have a **constructor** that we call? (No!)
- We never write code that **instantiates** our Activity. There is no `main` method. Activities are created and managed by the Android operating system!

### The Activity Lifecycle [10min]
So if we never call the constructor or main, how do we start doing stuff?

Activities have an _incredibly_ well-defined [lifecycle](http://developer.android.com/guide/components/activities.html#Lifecycle)&mdash;that is, a series of **events** that occur during usage (e.g., when created, stopped, etc).
- When each of these events occur, Android executes a **callback method**, similar to how you called `actionPerformed` to react to a "button press" event. We can override these methods in order to do special actions (read: our own code) when these events occur.

What is the lifecycle?

![lifecycle state chart](http://developer.android.com/images/activity_lifecycle.png)

(Or an alternative, simplified diagram [here](http://developer.android.com/images/training/basics/basic-lifecycle.png)).

There are 7 "events" that occur in the Activity Lifecycle, which are designated by the callback function that they execute:
- `onCreate()`: when **first** created/instantiated. This is where you initialize UI stuff (e.g., specify the layout to use)
- `onStart()`: called just before the Activity becomes **visible** to the user.
  - Whats the difference between this and `onCreate`? `onStart` can be called more than once (e.g., if you leave the Activity and come back).
- `onResume()`: just before **user interaction** starts--activity is ready to go! A little bit like when that Activity "has focus"
  - difference between `onStart()`? When start is called we're visible, but resume is called when interaction is ready.
  - We can be visible but not interacting (like if there is a modal in front of us!)
- `onPause()`: when the system is about to start another activity (so about to lose focus). This is the "mirror" of `onResume()`. _The activity stays visible_.
  - This is where we tend to store (temporary) unsaved changes (like email drafts), stop animations, etc, since the activity might be on its way out.
- `onStop()`: the activity is no longer visible. (e.g., another activity took over, but also because might be destroyed). A mirror of `onStart()``
  - This is where we would want to persist any state information (e.g., save their game).
- `onRestart()`: called when the app is coming back from a "stopped" state
- `onDestroy()`: activity is about to be closed. This can happen because the user ended the application, **or** (and this is important!) because the OS is trying to save memory and so kills your app (if it hasn't been use in a while).
  - can do more final cleanup, but better to have cleanup in `onPause()` and `onStop()`

Note that apps may not need to use all of these methods: for example, if there is no difference between starting from scratch and resuming from stop, then you don't need an onRestart (since onStart goes in the middle). Similarly onStart may not be needed if you just use onCreate and onResume; but these lifecycles allow for more granularity and the ability to avoid duplicate code.

### Overriding the methods
Let's look at overriding these methods to see them in action!

We already have `onCreate()` overridden for us, since that's where the layout is specified (we'll cover how soon, I promise!)
- Notice it takes a `Bundle` as a parameter. A [`Bundle`](http://developer.android.com/reference/android/os/Bundle.html) is an object that stores key-value pairs, like a super-simple `HashMap` (or an Object in JavaScript). Bundles can only hold basic types (numbers, Strings) and so are used for temporarily "bunding" _small_ amounts of information.
  - This `Bundle` in particular in stored information about the Activity's current state (e.g., what text they may have typed into a search box), so that if the App gets killed it can be restarted in the same state and the user won't notice that it was ever lost!
  - It stores current layout information in it by default (if Views have ids), and since its a Map (key-value pairs) you can store other data in it as well.
    - calls `onSaveInstanceState()` for each View, and those tend to save important bits already
    - see [Saving Activity State](http://developer.android.com/guide/components/activities.html#SavingActivityState) for details.
- Also note that we call `super.onCreate()`. **ALWAYS CALL UP THE INHERITANCE CHAIN**, so that you can do system-level stuff!

So we can add the others: how about `onStart()`? (see [Implementing Lifecycle Callbacks](http://developer.android.com/guide/components/activities.html#ImplementingLifecycleCallbacks) for template).


## Logging & ADB [10min]
But how can we know if the lifecycle events are getting called?
- `System.out.println()`? we don't actually have a terminal! More specifically, our device (which is where the application is running) doesn't have access to `stdout`
  - Aside: we can get it with `adb shell stop; adb shell setprop log.redirect-stdio true; adb shell start`
- Instead, Android provides a [Logging](http://developer.android.com/tools/debugging/debugging-log.html) system that we can use to write out debugging information, and which is automatically accessible over the `adb` (Android Debugging Bridge).
  - Logging can be filtered, categorized, sorted, etc.
  - Can be disabled in production builds, but often isn't :p
- To perform logging, we'll use the [`android.util.Log`](http://developer.android.com/reference/android/util/Log.html) class. This class includes a number of `static` methods, which all basically wrap around `println` to print to the device's log file, which is then accessible through the `adb`.
  - The device's log file is stored... sort of. It's a 16k file, but is shared across the _entire_ system. So fills up fast. Hence filtering/searching becomes important, and you tend to watch it/debug in real time!
  - Remember to import the [`Log`](http://developer.android.com/reference/android/util/Log.html)] class!

### Log Methods
Methods correspond to different level of priority (importance) of the messages. From low to high:
- `Log.v()`: VERBOSE output. The most detail. Everyday stuff. Often our go-to level.
  - Ideally, you should only compile into an application during development!
- `Log.d()`: DEBUG output. lower-level, but a bit less detail (e.g., code-level)
  - Debug logs _can_ be compiled but stripped at runtime using the [`BuildConfig`] generated class, which can be customized through Gradle
- `Log.i()`: INFO output. High level info (e.g., user-level)
- `Log.w()`: WARN output. Warnings
- `Log.e()`: ERROR output. Errors
- Also look at the API... `Log.wtf()`!

These are used to help "filter out the noise". So you can look just at errors, at errors and warnings, at err warn info... all the way down to _everything_ with verbose.
- A HUGE amount of information is logged, so filtering really helps!

Each `Log` method takes two `Strings` as parameters. The second is the message to print. The first is a "tag"--a String that's prepended to the output which you can search and filter on.
- This is is usually the App or Class name (e.g., "AndroidDemo", "MainActivity")
- A common practice is to declare a `TAG` constant you can use throughout the class:
```java
private static final String TAG = "MainActivity";
```

### Logcat
You can view the logs via `adb` (the debugging bridge) and a service called `Logcat` (from "log" and "conCATenation", since it concats the logs).
- The easiest way to check Logcat is to use Android Studio. The Logcat browser panel is usually found at the bottom of the screen after you launch an application. It "tails" the log, showing the latest output as it appears
  - You can use the dropdown box to filter by priority, and the search box to search (e.g., by tag if you want).
  - Android Studio also lets you filter to only show the current application, which is hugely awesome.
  - Note that you may see a lot of Logs that you didn't produce, including possibly Warnings (e.g., I see a lot of stuff about how OpenGL connects to the graphics card). _This is normal_!
- It is also possible to view Logcat through the command-line using `adb`, and includes complex filtering arguments. See [Reading and Writing Logs](http://developer.android.com/tools/debugging/debugging-log.html) for more details.

### Demo!
Let's log out some of the lifecycle!
- implement `onResume()`. Note the wonders of tab completion! Have it log out at INFO level. Hit `menu` to send to background.
- `onStop()` and switch out of the app.
- `onDestroy()` can easily be called if you set phone to "Don't Keep Activities" (at bottom of developer settings)
- Something else to test: Cause the app to throw a runtime `Exception` in one of the handlers. For example, you could make a new local array and try to access an item out of bounds. Or just `throw new RuntimeException()` (which is slightly less interesting).
  - _Can you see the **Stack Trace** in the logs?_

#### BREAK 5mins, resume 11:45 latest

## Basic Events [10min]
Now that can "output" some text (via log), let's add some "input" via an interface element: a Button we can click.
- In **`res/layouts/activity_main.xml`** (the original Activity's layout), add the following code:
```xml
<Button
    android:id="@+id/my_button"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Start Activity"
    />
```
This should go inside the `<RelativeLayout>` element, **replace** the `<TextView>` element.
  - This defines a (anyone?) Button. The `android:text` attribute gives the text that is on the button.
  - We'll talk in a lot more detail about how exactly this XML works (and what's the deal with the id, layout_width/height) next week, but you should be able to make a pretty good educated guess based on the names.
  - Actually, to make the button not overlap, change the `<RelativeLayout>` to a `<LinearLayout>`. More on layout tomorrow!
  - Defining this in XML is basically the same process as creating the `JButton` and adding it to the `JFrame` in Java!

Now we have a button, but we want to click on it. So we need to register a "listener" for it (in Java).
```java
Button button = (Button)findViewById(R.id.my_button);
button.setOnClickListener(new View.OnClickListener() {
    public void onClick(View v) {
        // Perform action on click
    }
});
```

- First we need to get access to a variable that represents that Button defined in the XML. The `findViewById` method "finds" the appropriate XML element with the given `id`. We'l talk about why we wrote the id as `R.id.my_button` tomorrow. Note that this method returns a `View`, so we want to cast into the more specific `Button`.
- Now we can assign a listener to that button by registering through the `.setOnClickListener()`, passing in an **anonymous class**
  - Again, tab-completion is our friend!
  - This is _just like_ what we did with Swing!
- Finally, we can fill in the method to have it log out something when clicked.

This is an example of an [Input Control](http://developer.android.com/guide/topics/ui/controls.html). We'll talk about these in more detail next week.


## Multiple Activities [15mins]
The whole point of considering the Activity Lifecycle is because Android applications can have multiple activities and interact with multiple other applications. So let's talk briefly about how we could have an app use multiple Activities (and so get a sense for how this lifecycle may affect us)

We can go ahead and create a New Activity through Android Student by using `File > New > Activity`. We could also just add a new .java file with the Activity class in it
  - Make a new Empty activity `SecondActivity`
  - Note that this Activity also gets a resource XML layout
    - You should edit the `<TextView>` element so this one can have a message as well.
- _ALSO NOTE_: For every activity we make, it gets added to the **Manifest** file. This is sort of like the "Table of contents" for our application, telling the operating system information about what our app looks like so it can interact with it.
  - another `<activity>` element in the `<application>` element. We can also see the first Activity; we'll talk about its child elements later (a lot of that)
  - We can also add `android:label` attributes to these `<activity>` elements to give them nicer names.

In Android, we don't start new Activities by instantiating them (remember, ___we never instantiate Activities___!). Instead, we send the operating system a message requesting that the Activity do something (i.e., start). These messages are called [**Intents**](http://developer.android.com/guide/components/intents-filters.html).
  - Intents are **messages** used to communicate between app components like Activities. This allows them to communicate, even though they don't have references to each other (so we can't just call a method on them).
  - I don't have a good justification for the name, other than it is an "intention" to do something that you announce to the OS
  - You can think of Intents as like envelopes: they are addressed to a particular target (e.g., another Activity--or more properly a `Context`), and contain a brief message about what to do.

[`Intents`](http://developer.android.com/reference/android/content/Intent.html) are something we _can_ instantiate, so let's do that in our event handler! There are lots of different constructors, but the one we'll start with is:
```java
Intent intent = new Intent(MainActivity.this, SecondActivity.class);
```
- The first parameter refers to the current [**Context**](http://developer.android.com/reference/android/content/Context.html), which is a superclass of `Activity`. Context is an **abstract class** that acts as a reference for information about the current running environment; it represents environmental data (stuff like "what OS is running? Is there a keyboard plugged in?"). You can _almost_ think of it as representing the "Application", though it's broader than that (since `Application` is actually a subclass of `Context`!)
  - The context is _used_ to do "application-level" actions: mostly working with resources (accessing/loading), but also communicating between Activities like we're doing now. Effectively, it lets us refer to the state in which we are running: the "context" for our code (e.g., "where is this occurring?"). It's a kind of _reflection_ or meta-programming, in a way.
  - There are a couple of different kinds of Contexts we might use:
    - The Application context (e.g., the `Application`) references the state of the entire application. It's basically the Java object that is built out of the Manifest (and so contains that level of information)
    - The Activity context (e.g., the `Activity`) that references the state of that activity. Again, this would be the `<activity>` tags from the Manifest.

    Each of these `Context` objects exist for the life of their respective component: that is, an `Activity` Context is around as long as the Activity exists (disappearing after `onDestroy`), where as `Application` Contexts survive as long as the application does. We'll almost always use the `Activity` context, as it's safer and less likely to cause memory leaks.

- The second parameter is the class we want to send the Intent to (the `.class` property fetches a reference to the class type; this is metaprogramming!)

And now that we built the intent, we can use it to start an activity using the [`startActivity`](http://developer.android.com/reference/android/app/Activity.html#startActivity(android.content.Intent)) method (inherited from `Activity`), passing it the `Intent`!
- Voila! we can now start a second activity, and see how that impacts our Lifecycle calls (e.g., with visibility, etc).
- And we can use the **back** button to go backwards!

There are actually a couple of different kinds of `Intents` (this is an **Explicit Intent**, because it is explicit about what Activity it's sent to), and a lot more we can do with them. We'll dive into Intents in more detail later; for now we're going to focus on mostly Single Activities.
- e.g., if you look back at the Manifest, you can see that the MainActivity has an `<intent-filter>` that allows it to receive particular kinds of Intents--including ones to use it when launching the App!

## Back & Tasks [5-10min]
So we can have lots of Activities (even across multiple apps!) running and move between them. How exactly is that "Back" button keeping track of where to go to?
- Do you know what kind of data structure is associated with "back" or "undo"? A **stack**!
- Every time you start a new Activity, Android creates it and puts it on the top of a stack. Then when you hit the back button, that activity is popped off the stack and you're taken to the new head.
![activity stack example](http://developer.android.com/images/fundamentals/diagram_backstack.png)

However, you might have different "sequences" of actions you're working on: maybe you start writing an email, and then go to check your Twitter timeline through a different set of Activities. Android breaks up these sequences into groups called [**Tasks**](http://developer.android.com/guide/components/tasks-and-back-stack.html). A task is a collection of activities arranged in a Stack; and there can be multiple tasks in the background.
- Tasks usually start from the Home Screen. E.g., when you launch an Application, that starts a new Task.
- When you go back to home screen, that Task is moved to the background, so the "back" button won't let you navigate that Stack.
- Thinking of them like different tabs/browsers and webpages is a pretty good analogy

Important caveat: Tasks are distinct from one another, so you can have different copies of the same Activity on multiple stacks (e.g., the Camera activity could be part of both Facebook and Twitter app Tasks if you are on a selfie binge)
  - Though it is possible to modify this, see [Managing Tasks](http://developer.android.com/guide/components/tasks-and-back-stack.html#ManagingTasks)

Demo: switch to another app, then back to ours

### Bonus: Up Navigation
We can make this "back" navigation a little more intuitive for users by providing explicit [up navigation](http://developer.android.com/design/patterns/navigation.html)), rather than just forcing them to go back through Activities in the order they viewed them (e.g., if you're swiping through emails and want to go back to the home list). We just need a little bit of configuration for our Activities:
- In the Java code, we want to add more functionality to the `ActionBar`. *Think*: what event handler should it be put in?
  ```java
  getSupportActionBar().setHomeButtonEnabled(true);
  ```
- Then In the **Manifest**, add a `android:parentActivityName` attribute to the `SecondActivity`, with a value to set the full class name (including package **and** appname!) of your Main Activity. This will let you be able to use the "back" visual elements (e.g., of the ActionBar) to move back to the "parent" activity. See [Up Navigation](http://developer.android.com/design/patterns/navigation.html) for details.
    - This is only supported for API 16+; since our min SDK is 15, we can include backwards support with the following child XML element:
    ```xml
    <meta-data
         android:name="android.support.PARENT_ACTIVITY"
         android:value="{parent.activity.package.goes.here}" />    
    ```

## Bonus: Toasts




## Extra: Toasts [5min]
Logging is fantastic and one of the the best techniques we have for debugging, both in how Activities are being used or for any kind of bug (also RuntimeExceptions)
- It harkens back to printline debugging, which is totes legit. Android Studio does have a [debugger](http://developer.android.com/tools/debugging/debugging-studio.html) if you're comfortable with those (can be handy!)

However, sometimes you want to check some output/interaction without Logging it. You just want to see some feedback while the app is running! Or you want to give a quick message to the user. Android provides a number of different classes for doing visual notifications, including alert-style and customizable [Dialogs](http://developer.android.com/guide/topics/ui/dialogs.html), which we'll talk about in a few weeks.

But a simple, quick way of giving some short visual feedback is to use what is called a [Toast](http://developer.android.com/guide/topics/ui/notifiers/toasts.html). This is a tiny little text box that pops up at the bottom of the screen for a moment.
- Toast because it pops up :p

Toasts a pretty simple to implement, as with the following example (from the docs):
```java
Context context = this; //getApplicationContext(); //for not disappearing if app closed quickly.
String text = "Hello toast!";
int duration = Toast.LENGTH_SHORT;

Toast toast = Toast.makeText(context, text, duration);
toast.show();
```
- But since `this` Activity _is_ a context, and we can just use the Toast anonymous, we can shorten this to a one-liner:
```java
Toast.makeText(this, "Hello world", Toast.LENGTH_SHORT).show();
```
  - Boom, a quick visual alert method we can use for proof-of-concept stuff!
- Note that this uses a static `makeToast()` method, rather than a constructor. This is an example of a Factory method--a design pattern we'll see a lot.
- Toasts are intended to be ways to interact with the user (e.g., giving them quick feedback), but can possibly be useful for testing too! Though in the end, Logcat is going to be your best bet for debugging.


## Action Items
- Finish up the warmup! We've covered all you need to finish it. If there are questions, ask early!
- Take a look at the next assignment (SunSpotter) if you have a chance---though most of it is resource work, which we'll talk about on Monday.
- Review the documentation as needed; lots of details out there!


#### Lecture References
- [Activity Guide](http://developer.android.com/guide/components/activities.html)
- [`Activity`](http://developer.android.com/reference/android/app/Activity.html)
- [lifecycle state chart](http://developer.android.com/images/activity_lifecycle.png)
- [`android.util.Log`](http://developer.android.com/reference/android/util/Log.html)
- [Input Control](http://developer.android.com/guide/topics/ui/controls.html)
- [Context](http://developer.android.com/reference/android/content/Context.html)
- [activity stack example](http://developer.android.com/images/fundamentals/diagram_backstack.png)
