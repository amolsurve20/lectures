## 04-06 Lists & Data

### Admin [5min]
- Homework check-in: manage to get the Java assignment turned in? Taken a look at next one SunSpotter?
  - Due week from today--I hope you've already started it!!
  - This assignment will feel bigger--it's a full app! (more or less), but we should cover all the pieces you need by end of the day.

## Inputs [20min]
Yesterday we talked about **Views**. (_What's a View?_). We discussed `ViewGroups` (Layouts), and we've used some basic `Views` like `TextView`, `ImageView`, and `Button`. That last one is an example of an [Input Control](http://developer.android.com/guide/topics/ui/controls.html)
- These are "simple" (read: single purpose) widgets that allow for use input.
- We've already seen Button, but there are others! Mostly in the [`android.view`](http://developer.android.com/reference/android/view/package-summary.html) package

[[Show off a demo activity with them all!]]
- [Button](http://developer.android.com/guide/topics/ui/controls/button.html) (afford clicking). Can have text, images, or both
- [EditText](http://developer.android.com/guide/topics/ui/controls/text.html) (text entry)
  - can control `android:inputType` (just like inputs in HTML!)
- [Checkbox](http://developer.android.com/guide/topics/ui/controls/checkbox.html) (on-off state)
- [RadioButton](http://developer.android.com/guide/topics/ui/controls/radiobutton.html) (select from a set of choices)
  - put these into a `RadioGroup` to make them mutually exclusive
- [ToggleButton](http://developer.android.com/guide/topics/ui/controls/togglebutton.html) (on-off state)
- [Switch](http://developer.android.com/reference/android/widget/Switch.html) (on-off state; ToggleButton with a slider UI). Introduced in API 14.
- [Spinner](http://developer.android.com/guide/topics/ui/controls/spinner.html) (pick from an array of choices)
  - define choices in resources (e.g., `strings.xml`)!
- [Pickers](http://developer.android.com/guide/topics/ui/controls/pickers.html): compound controls around some specific input (dates, times, etc)
  - typically put in pop-up dialogs, which we'll talk about next week
- ...and more! (see [`android.widget`](http://developer.android.com/reference/android/widget/package-summary.html))

These all mostly work the same way: you define them in the layout resource, and then can access them in Java in order to interact with them.

There are two ways of interacting with controls (and Views in general) from Java: (1) calling methods on them and (2) listening for events from them
- Can think of methods as "outside to in" (wrt the control)
- Can think of events as "inside to out"

We've already done the event-driven approach to respond to buttons.
- We register a **Listener** and specify a **callback method** to get "called back to" when the event occurs.
  - Find the button, set a listener, etc.
- Note that we can also specify a callback method in the resource using the `android:onClick` attribute. The value is the _name_ of the method.
    ```xml
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:onClick="handleButtonClick" />
    ```
  We declare the callback method in Java as taking in a `View` parameter and returning `void`:
    ```java
    public void handleButtonClick(View view) { }
    ```
  The `View` passed to the method will be whatever View caused the event to occur.

- We'll actually use a mix of both of these strategies in this class.
  - _Opinion_: arguable about which is better--the resource is easier/faster, but starts mixing the "behavior" and the logic (though since buttons are made to be pressed, it's not unreasonable to give an "name" to what they should do; this can always just be a "launcher" method that calls something else)

We can use listeners like this to respond to all kinds of controls.
- Checkboxes use `onClick`, ToggleButtons use `onCheckedChanged`, etc.
- Other common events in the [View documentation](http://developer.android.com/reference/android/view/View.html#NestedClasses)
  - Include `OnDragListener` (for drags), `OnHoverListener` ("hover" events), `OnKeyListener` (when user types), `OnLayoutChangeListener` (when layout changes display), etc.

In addition to listening for events, we can call methods directly on these Views to access their state. Methods such as `isVisible`, `hasFocus`, etc. give us information about state, but we can also inquire directly about the inputs
- For example, the `isChecked()` method lets us look up if a checkbox is ticked whenever we want to know

This is also a good way of getting inputs: for example, we can call `.getText()` on an `EditText` view to fetch the contents of that view
- [[demo: log contents when button pressed]]

Between listening for events and querying for state, we can interact with input controls however we want. Check the documentation for more details on how to use specific widgets!


## ListView and Adapters [30min]
((Switch to `list_layout` Layout))!

Now that we've covered basic controls, let's look at some more advanced Views. In particular, the [ListView](http://developer.android.com/guide/topics/ui/layout/listview.html).
- It's is a _ViewGroup_ that displays a scrollable list of items!
- This is basically a `LinearLayout` inside a `ScrollView` (a `ViewGroup` that can be scrolled).
  - Each item in the linear layout is another `View` (usually a layout) for that particular item
  - However, The `ListView` does extra work: it keeps track of what items are already laid on the screen, inflating only the visible items plus a few extra on the top and bottom as buffers. Then as the user scrolls, it takes the disappearing views and recycles them (altering the content) to reuse for the new Views that are appearing. This lets it save on memory and performance and work smoothly. See [here](https://github.com/codepath/android_guides/wiki/Using-an-ArrayAdapter-with-ListView#row-view-recycling) for some diagrams.
  - A more advanced and flexible version of this behavior is offered by the [RecyclerView](http://developer.android.com/reference/android/support/v7/widget/RecyclerView.html) class. See [this guide](http://developer.android.com/training/material/lists-cards.html).

The `ListView` control uses a **Model-View-Controller** architecture. What's that?
- A design pattern common in UI systems
- **Model** is the data, **View** is the representation/display of that data, and **Controller** hooks them together!
  - Originally developed as part of the [Smalltalk](http://heim.ifi.uio.no/~trygver/themes/mvc/mvc-index.html) language
- You'll actually find this all over Android. The resources are models and views (separately), with the Java Activities act as controllers
- So with a `ListView`, we'll have some data to be displayed, the Views (layouts) to be shown, with the `ListView` control itself acting as a controller

Specifically, the `ListView` is a subclass of [`AdapterView`](http://developer.android.com/reference/android/widget/AdapterView.html), which is a view backed by a data source--the `AdapterView` hooks the two together (the controller)!
- There are other `AdapterViews` as well; [`GridView`](http://developer.android.com/guide/topics/ui/layout/gridview.html) works exactly the same way, but lays out items in a scrollable grid rather than a scrollable list.

Okay, so let's get our pieces in place
1. First we need the **model**. Some raw data. Let's use a String[] (and we can fill it with whatever data):
    ```java
    String[] data = new String[99];
    for(int i=99; i>0; i--){
      data[99-i] = i+ " bottles of beer on the wall";
    }
    ```

2. Next we want the **view**. A `View` to show. Let's make a layout resource for that (`list_item` is a good name and a common idiom)
  - Put a basic `TextView` in there; no layout needed! (width: `match_parent`, height: `wrap_content`)
  - can give `android:minHeight=?android:attr/listPreferredItemHeight` (framework's preferred height), and some `center_vertical` gravity
  - `android:lines` if we need more space
  - **don't forget an `id`!**

3. Finally, we want the **controller**. The `ListView` itself; let's add that element to our Activity's layout. How big should it be?


To fill in the controller `ListView`, we need to provide it an [`Adapter`](http://developer.android.com/guide/topics/ui/declaring-layout.html#AdapterViews) to use to connect the model to the view. The Adapter does the "translation" work of converting models to views (and getting the model for particular views to show)
- We're going to use an [`ArrayAdapter`](http://developer.android.com/reference/android/widget/ArrayAdapter.html), because we have an array and because it's the simplest to use.
  - An `ArrayAdapter` creates views by calling `.toString()` on each item in the array and putting that String inside a `TextView`!
  - Check the parameters of the constructor (and note the generics!)

  ```java
  ArrayAdapter<String> adapter = new ArrayAdapter<String>(this,
    R.layout.list_item_layout, R.layout.list_item_txtView, myStringArray);
  ```
- We then get a reference to the `ListView` with `findViewById`, and call `.setAdapter()` to attach the adapter to that controller.
```java
ListView listView = (ListView) findViewById(R.id.listview);
listView.setAdapter(adapter);
```

And voila! We have a scrollable list of data! What else can we do with it?
- Each item in this list is selectable (can have an onClick), so we can click on them e.g., to go to a "detail" Activity
  - use `AdapterView#..setOnItemClickListener(ItemClickListener)` to register!
  - The `position` is where in the list they clicked, use `(Type)parent.getItemAtPostion(position)` to fetch the value.
- Each item does have an individual layout, so we can customize these appearances (e.g., if our layout also wanted to include pictures)
  - See [this guide](https://github.com/codepath/android_guides/wiki/Using-an-ArrayAdapter-with-ListView#row-view-recycling) for an example on making a custom adapter to fill in multiple `Views` with data from a list! You will _not_ need to do this for your homework, though you can if you want.

And remember, a `GridView` is basically the same thing (in fact, we can just change over that and have everything work, if we use _polymorphism_!)

### **break**

## Network Connections and ASyncTask [40min]
We have our lovely `ListView` hooked up to an adapter so it can show a list of Strings. But during lab we were using some code that fetched data from the Internet and gave us a list of Strings... can we combine them?!
- You betchya!

Let's go ahead and add in the downloader method (copy and paste into this class, or can just access it in the other), and then we can call it to fill our `String[]` (instead of doing it by hand)
- Does it work? What went wrong?! (look at the logs!) `NetworkOnMainThreadException`

Android apps run by default on the ___Main Thread___ (also called the ___UI Thread___). This thread is in charge of all user interactions--handling button presses, scrolls, drags, etc.--but also ui _output_ like drawing/displaying text! See [Android Threads](http://developer.android.com/guide/components/processes-and-threads.html#Threads) for more details.
- _What's a thread?_ (You played with these in lab!) A thread is a piece of a program that is independently scheduled by the processor. Computers do exactly one thing at a time, but make it look like you're doing lots of stuff by switching between them (between processes) really fast. Threads are a way that we can break up a single application or process into little "sub-process" that can be run simultaneously---by switching back and forth periodically so everyone has a chance to work
- Within a single thread, all method calls are **synchronous**---that is, one has to finish before the next occurs. We can't get to step 4 without going past step 3. With an event-driven system like Android, each method call is fast enough that this isn't a problem.
- But for long, drawn-out processes like network access (or processing bitmaps, or accessing a database), this could cause all of the other stuff to have to wait. It's like a traffic jam!
  - Things like network access are **blocking** method calls, which stop the Thread from continuing.
  - A blocked Main Thread will lead to the infamous **"Application not responding" (ANR)** error

We need to move the network code _off_ the Main Thread, onto a **background thread**. To do this, we're going to use a class called [`ASyncTask`](http://developer.android.com/reference/android/os/AsyncTask.html) to let us perform a task (like network connecting) asynchronously (without waiting for other people).
- How did we find this class? Look at the [Processes and Threads Guide](http://developer.android.com/guide/components/processes-and-threads.html)!
- Note that this background thread will be _tied to the lifecycle of the Activity_---if I close the Activity, the network connection will die as well.
  - A better, if _much_ more complex solution would be to use a `Service`, which we will cover later in the course. But since we're just fetching a small amount of data, we don't really care if the network connection gets dropped.
- This class can be pretty complicated to use, but let's look at the documentation and walk through it.

The first thing we should notice (if the API was a little more readable) is that `ASyncTask` is **`abstract`** What does that mean? (We need to subclass it to use it!)
- We can also notice that it's a generic class with three (3)! parameters: the type of the Params to the task, the type of the Progress measurement, and the type of the Result. We can start these as `Void` if we don't know them, but we should be able to guess what kind of params/returns we want from our method (take in a `String`, return a `String[]`)
- So we can make an _inner_ class (non-static, because we want to be tied to _this_ Activity) that extends `ASyncTask`. `MovieDownloadTask` is a good name

But what goes inside this class? When we "run" an AsyncTask, it will do four (4) things:
1. `onPreExecute()` is called _on the UI thread_ before we run the task. We can do setup here
2. `doInBackground(Params...)` is called _on the background thread_ to do that work
  - we _must_ override this (it's `abstract`!)
  - note it's params and return type needs to match the `ASyncTask` generic types!
3. `onProgressUpdate()` is called _on the UI thread_ if we want to update our progress (e.g., update a progress bar)
4. `onPostExecute(Result)` is called _on the UI thread_ to process any results (which are passed in when `doInBackground` finished)

The `doInBackground()` is the heart of the method, so let's move our networking code in there!

We can then _instantiate_ a new one of these classes in `onCreate()` and call `.execute(params)` to start it running.
- Does it work? What went wrong? `SecurityException`!

As a [security feature](http://developer.android.com/guide/topics/security/permissions.html), Android apps by default have very limited access to the system (e.g., to do anything other than show a layout). An app can't use the Internet (which might eat at people's data plans!) without explicit permission from the user. This permission is given at _install time_.
- So in order to get permission we need to ask for it ("Mother may I..."). We do that by declaring that the app uses the Internet in the `Manifest` (which has all the details of our app!)
```xml
<uses-permission android:name="android.permission.INTERNET"/>
<!-- put this ABOVE the <application> tag -->
```
- Note that Marshmallow introduced a [new security model](https://developer.android.com/training/permissions/requesting.html) in which users grant permissions at _run-time_, not install time, and can revoke permissions whenever they want. To handle this, you basically need to add code to request the permission each time you use it (for "dangerous" permissions, like location, phone or SMS. Internet is _not_ dangerous).
  - For "normal" permissions (Internet), you declare in the Manifest
  - For "dangerous" permissions (Location), you declare in the Manifest _AND_ request in code each time you want to use it.
  - Since we're targeting Lollipop, won't affect us yet, but something to keep in mind (and we'll look at exactly how to handle this later)

Finally, we can connect to the bloody internet (Log out the results to prove it)!

But how do we get this stuff back into our ListView?
- Remember that `doPostExecute()` function? That happens on the _UI Thread_ so we can use it to update the View (we can _only_ change the View on the UI Thread, to avoid collisions). It also gets the results returned by `doInBackground()` passed to it!
- So we can take that String[] and put it into our `ListView`. Specifically, we'll feed it into the `Adapter`
  - make the Adapter an instance variable so we can access it in this other class
- We'll first clear out any previous entries in the adapter using `adapter.clear()` And then we use `.addAll()` to put all the items into the Adapter (or can loop through the data and `.add()` each item).
  - We can call `.notifyDataSetChanged()` on the `ArrayAdapter` to make sure that the View knows the data has changed, but this is already called by the `.add()` method so isn't necessary here.

How do we search for a different movie?!
- Well we can take our `EditText` field and its button from the other Layout, and use it to get a String. We can then pass that String into the `execute` function, since we've declare that the Generic `ASyncTask` takes that type as the first param.
  - We can actually pass in a lot of Strings, using the `String... params` syntax (arbitrary number of items of that type). See [here](http://docs.oracle.com/javase/tutorial/java/javaOO/arguments.html#varargs). The value we _actually_ get is an array.

Whew! We've now got some data downloading off the net and showing up on our screen! Spiffy!
- We've done a whirl-wind tour of Android in this process: Layouts in the XML, Adapters in the Activity, Threading in a new class, Security in the Manifest... bringing lots of parts together to make a set of functionality.
- Note that this is the same process you'll need to do for your homework this week!

## Extras
- Parse the JSON with `JSONObject` and `JSONArray`
- Toasts (last week)

### Lecture References
- [ListView](http://developer.android.com/guide/topics/ui/layout/listview.html).
- [`Adapter`](http://developer.android.com/reference/android/widget/Adapter.html)
- [`ASyncTask`](http://developer.android.com/reference/android/os/AsyncTask.html)
