## Fragments

### Admin [10min]
- Homework check-in
  - Warmup grades out this morning
  - Any questions/issues on SunSpotter?
  - Next assignment is online, will use topics we talk today (fragments!) and Thursday (menus, loaders!)
- Fork and clone the repo for today! (Building off the movie example from last week). Recall:
  - We can search for films and they show up
  - We can click on the element and log what movie was selected!


## Fragments [10min]
Today we're going to talk about [Fragments](http://developer.android.com/guide/components/fragments.html). A Fragment is

> a behavior or a _portion_ of user interface in an Activity

You can think of them as "mini-activities" or "sub-activities". Fragments are designed to be **composable** so you can mix and match them within a user interface.

- _What's the difference between a Fragment and a layout resource (e.g., reusable xml)?_ An XML resource is _just_ the view, while the Fragment is the controller (and model). In other words, with a Fragment we can make re-usable pieces of Activities that have their own layouts (views), but also data models, event callbacks, etc.

- Fragments were introduced in API 11 (Honeycomb), which was the "tablet" version of Android. They were designed to provide a UI component that would allow for side-by-side activity displays appropriate to larger screens.
![fragments][frag_img]
  - Instead of needing to navigate between two related views (particular for this "master and detail" setup), the user can see both views within the same Activity... but those "views" could also be easily split between two Activities for smaller screens, because their required _logic_ is kept separated

  - Fragments are intended to be **modular**, **reusable** components. They should ___not___ depend on the Activity they are inside, so that you can be flexible about when and where they are displayed!

As I said, Fragments are like "mini-Activities". However, Fragments are _always_ embedded inside an Activity; they cannot exist outside. While it's possible to have Fragments that are not visible or that don't have a UI, they still are part of an Activity

- Because of this, a Fragment's lifecycle is directly tied to its containing Activity's lifecycle. (e.g., if the Activity is paused, the Fragment is too. Activity destroyed; Fragment too). But while the Activity is running, they can be independently manipulated.

- That said, Fragments also have their own lifecycle, and corresponding lifecycle callbacks (just like an Activity)

![fragment lifecycle][frag_lifecycle]

- This is a lot like the Activity Lifecycle, with a couple of extra steps:
  - `onAttach()` when the Fragment is first associated with an Activity (even if not yet created; e.g., for "headless fragments").
  - `onCreateView()` when the view (ui) is about to be drawn
  - `onActivityCreated()` after everything is setup, but before start

- We're mostly going to be interested in `onCreate()` (like with Activities), `onCreateView` (the UI-setup equivalent), and `onPause()` (for stopping).


### Making a Fragment (refactoring!) [15min]
Let's go ahead and make a Fragment! Or rather, let's ___refactor___ our Activity to use Fragments for the display. This is illustrative to see how the pieces are related.
- _Refactoring_ is the process of re-structuring code without changing the behavior.

To create a Fragment, you subclass the `Fragment` class. Let's make one called `MovieFragment`
  - You can also use Android Studio to do this work: `New > Fragment`. **DO NOT** select any of the other options for now (they provide template code that makes it seem overwhelming)

So there are two versions of `Fragment`: one in `android.app` and one in `android.support.v4`. Which do we use?
  - The later is the class from the [Support Library](http://developer.android.com/tools/support-library/index.html). These are libraries of classes designed to make Android applications backwards compatible: for example, `Fragments` came out in API 11 so aren't in the `android` package for earlier devices; by including the support library, we can include those classes as well!
  - Support libraries _also_ include additional classes that, for whatever reason, are not included in the API. These might include user interface elements (e.g., `ViewPager`) and accessibility classes. So often useful even if you're targeting later APIs (because don't have to "roll your own" versions of some stuff)
  - The disadvantage of the support libraries: they'll make your final APK bigger (avoid premature optimization).
    - You can also occasionally run into problems if you mix and match versions of classes. **So when in doubt**: choose the support library!

The first thing we'll do is edit the layouts to support Fragments.
- You can see that we got a new `fragment_movie` layout. Since we want our Movie list to live in that fragment, let's move it there!

- Then we'll adjust our `activity_main` so instead of having that content, it's just an empty `FrameLayout`
  - a `FrameLayout` is a layout that holds exactly one element in it. Good as a simple "container"
  - Give it an `id` so we can refer to it later
  - To demonstrate how things will work, we can directly specify a `<fragment>` element inside this FrameLayout. Give it a `android:name` that is the full-package class name. This works fine, but it's more worthwhile to define fragments **dynamically** at runtime in the Java (since the whole point is we want to swap them around!) We'll do this in a moment.

Now we can start filling in the Java for our Fragment.
- Android Studio gave us a little starting code, we're mostly interested in filling in the `onCreateView()` method to setup our layout!

  - Look at the equivalent code in the Activity's `onCreate()`. We specify the layout by calling `setContentView()` but we can't just do that, because the Fragment has to belong to an Activity. So instead we're going to figure out what ViewGroup it is inside, and **inflate** to fill that ViewGroup. This "inflated" View we'll call the ___root view___, and it's the View that all of the stuff inside our Fragment will be attached to.
    - Think of a tree of elements, like the DOM tree!

  - We can get this **root view** with the following code:
  ```java
  View rootView = inflater.inflate(R.layout.fragment_movie, container, false);
  ```
    - Params are the layout to inflate/use, the `ViewGroup` given to us, and whether to attach the inflated view to the container (false, because we're filling in the container, not adding to it)

  - we return this **root view** at the end, since it's what is used to specify the fragment that should be shown in the Activity (lifecycle is a tad funky here, but that's the framework pattern)

We're now ready to start moving the functionality from our Activity into our Fragment.
  - Start with the Background Task. We want the Fragment to do this work... so move it over!
  - Move over the Adapter declaration too (since we want to access it)

  - Finally, we can move over the code used to set up our UI (including the Adapter). The **one** difference here is that `findViewById` is a method that Activities have. We're not an Activity... so we instead want to call this method on our **root view**, to use _that_ as the starting point for the search
    - We'll also specify that the first param (the context) of the `ArrayAdapter` is the Activity the Fragment is inside... use `getActivity()` for that (which is how Fragments can refer to their containing activity--used _primarily_ for accessing context!)

But if we left our buttons in the Activity, how can we tell the Fragment to do some work (e.g., to search for movies)? We need to get access to that `XML` element (the same way we would do with `findViewById`).
- To do this, we're going to need a [`FragmentManager`](http://developer.android.com/reference/android/app/FragmentManager.html), which is an object responsible for \*cough\* managing fragments. It can let us `get` references to existing fragments, as well as help us move back and forth between fragments (more on that in a second).
- Access the manager by calling `getSupportFragmentManager()` on the Activity, and then can call `findFragmentById()` to access an XML-defined Fragment:
```java
MovieFragment fragment = (MovieFragment)getSupportFragmentManager().findFragmentById(R.id.fragment);
```
  - Note that we're accessing the **support** `FragmentManager`. This is because our v15+ Activity works with both classes, but because they don't have a shared `interface` we need to specify different Java methods to get the correct version.
- Once we have the fragment, we can just call a public method on this Fragment! (e.g., `downloadMovies()`);

And with that, we should be able to re-run our program and... nothing has changed! It all works, it's just that our work is now encapsulated inside a Fragment that can be re-used in different layouts!
  - Effectively, we've made our own "widget" that can be dropped into any other screen. Like if we wanted to always have the list of movies at the bottom or something.

## Multiple Fragments [30min]
This is great, but our goal was to be able to support multiple fragments in a single Activity. For example, the archetypal ["master/detail"](http://developer.android.com/training/implementing-navigation/descendant.html#master-detail) view, where one pane (Fragment) holds the "master" (list), and another holds details about a particular item. Common examples: email reading, newsfeed reading, etc.
- Let's make it so that when we click on a Movie in the list, we get a detail Fragment that shows information about it.

### Transactions
To do this, we're going to need to specify Fragments dynamically in code, rather than in the XML (so not using the `<fragment>` element). Basically we need to call some code to "load the fragment into the container."

The class we use to do this is called a [`FragmentTransaction`](http://developer.android.com/reference/android/app/FragmentTransaction.html). A transaction represents a change in the fragment that is being displayed ([more details](http://developer.android.com/guide/components/fragments.html#Transactions))
  - Can think of like a bank transaction: add/remove fragments like you add/remove money.

To create and run a Transaction, we're going to use our `FragmentManager` again. Call `.beginTransaction()` to create a **new** Transaction.

Transactions represent a series of changes that all go through at the same (like putting money into and getting out of an account).
- Then you can call `.add()`, `.remove()`, or `.replace()` on that transaction to specify changes to the fragments.
- Finally, call `.commit()` on the transaction to "submit" it and have all the changes go into effect.

Example: have the `Activity.onCreate()` add the fragment, rather than the XML:
```java
FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();
//params: container to add to, Fragment to add
transaction.add(R.id.container, new MovieFragment(), MOVIE_LIST_FRAGMENT);
transaction.commit();
```
- Notice that we **do** instantiate `new Fragments()`. This means we can define a constructor for them and pass them parameters if we wanted!
- The third argument is a "tag" we apply to the Fragment. This gives it a name so that we can refer to it later (via `FragmentManager#findFragmentByTag(tag)`)
  - Alternatively, we could save this Fragment as an instance variable... which is a bit faster but more memory intensive (and can cause possible leaks, since it means its keep that fragment from being destroyed by the system)

### A Second Fragment
Similarly, we can use this behavior to make ourselves a "detail" view for the Movie. When we click on a movie in the list, we want to **`replace()`** this fragment with the displayed fragment. So `onClick()` we'd create a new transaction and run it (with a `replace()` command).

- But remember that Fragments are supposed to be modular--this Fragment shouldn't know about other Fragments or that it is replaced (what if we want master/detail to be side by side)?
  - This Fragment has _one and only one responsibility_ (good cohesion and encapsulation!!)
  - **Instead**, this Fragment should tell the `Activity` that a Movie has been selected, and then that `Activity` can determine what to do about that.

- The recommended way to [have fragments communicate](http://developer.android.com/training/basics/fragments/communicating.html) is to use `interfaces`. The Fragment should specify an interface that its containing Activity _must_ support--and since it provides that method, we can simply call it on the `Activity` to do something.

  - Create a new `interface` inside the Fragment (e.g., `OnMovieSelectedListener`).

  - In the `onAttach()` method, we can save a reference to the `Activity`, checking that it actually supports the behavior we require:
  ```java
  public void onAttach(Context context) {
      super.onAttach(context);

      try {
          callback = (OnMovieSelectedListener)context;
      } catch (ClassCastException e) {
          throw new ClassCastException(context.toString() + " must implement OnMovieSelectedListener");
      }
  }  
  ```
  - Don't forget to call the method from the click handler (can `getActivity()` and cast to avoid `null`)

  - Finally, have the Activity implement the method!
    - In that method, we can use the `FragmentManager` to run a `replace()` Transaction. Again, specify the `container` and the Fragment we want to use!

Tada, we switch to a different fragment!
- This is not the only way to communicate! We could instead send an `Intent` to the Activity, and have it respond to that `Intent` as desired (e.g., "you want me to show stuff about a movie? Okay!"). But that's a bit heavy-duty for what we're doing.


### The Back Stack
But what happens when we hit the "back" button? The Activity exits! _Why?_ Because "back" says to "leave the Activity"---we only had one Activity, just multiple fragments.

Recall that we can have lots of Activities (even across multiple apps!) running and move between them. How exactly is that "Back" button keeping track of where to go to?
- Do you know what kind of data structure is associated with "back" or "undo"? A **stack**!
- Every time you start a new Activity, Android creates it and puts it on the top of a stack. Then when you hit te back button, that activity is popped off the stack and you're taken to the new head.
![activity stack example][backstack_img]

However, you might have different "sequences" of actions you're working on: maybe you start writing and email, and then go to check your Twitter timeline through a different set of Activities. Android breaks up these sequences into groups called [**Tasks**](http://developer.android.com/guide/components/tasks-and-back-stack.html). A task is a collection of activities arranged in a Stack; and there can be multiple tasks in the background.
- Tasks usually start from the Home Screen. E.g., when you launch an Application, that starts a new Task.
- When you go back to homescreen, that Task is moved to the background, so the "back" button won't let you navigate that Stack.
- Thinking of them like different tabs/browsers and webpages is a pretty good analogy

Important caveat: Tasks are distinct from one another, so you can have different copies of the same Activity on multiple stacks (e.g., the Camera activity could be part of both Facebook and Twitter app Tasks if you are on a selfie binge)
  - Though it is possible to modify this, see [Managing Tasks](http://developer.android.com/guide/components/tasks-and-back-stack.html#ManagingTasks)

Fragments by default are not part of the "back-stack". However, you can specify that a Transaction should include the Fragment shift as part of this navigation by calling `.addToBackStack()` as part of your transaction (e.g., right before you `commit()`). See [this guide](http://developer.android.com/training/implementing-navigation/temporal.html#back-fragments).

- Athough Google's documentation says that this should "just work", for `AppCompatActivity` at least you need to manually "pop" fragments off the stack (if using `android.app.Fragments` instead of the support library version, which I had previously said to do... Ick.)
```java
public void onBackPressed() {
    if(getFragmentManager().getBackStackEntryCount() != 0) {
        getFragmentManager().popBackStack();
    } else {
        super.onBackPressed();
    }
}
```

### Break (~5min; @ 11:35ish)

### Saving State [10min]
We're almost done. But we'd like to make it so the detail view shows stuff about the movie we clicked (rather than placeholder text). So we need to pass some data from the Activity (who knows what movie we've clicked) to the detail fragment (who doesn't).

Do to this, first we need to talk about the `Bundle`.
- You may recall the `Bundle` parameter that is passed into `onCreate()` in the Activity. _Does anyone remember what this is for?_ It lets us **save the state** of the Activity, so that if it is recreated after being destroyed we won't lose state information (e.g., an email we had started composing).
- As the Activity is about to stop, it actually calls another "lifecycle"-like method called <a href="http://developer.android.com/reference/android/app/Activity.html#onSaveInstanceState(android.os.Bundle)">`onSaveInstanceState()`</a>. This method's default implementation causes each View to call the method on itself, saving their state inside the `Bundle`. We can override this method to save additional information as well (as key-value pairs of primitives).
- When the Activity is recreated, that `Bundle` is passed into the `onCreate()` method so we can fetch values from it.
- see [Saving Activity State](http://developer.android.com/guide/components/activities.html#SavingActivityState) for details.
  - if time: we can save the search term to be able to restore it later

### Passing Data to Fragments
We're going to use a similar idea to pass data into the Fragment--or at least, we're going to use a `Bundle`.
- Start by making a `new Bundle()`
- Then use the `putType()` methods to put data into the `Bundle`. Note that these need to be primitive types (`int`, `String`, etc.)
- We can then take the Fragment we are creating and call <a href="http://developer.android.com/reference/android/app/Fragment.html#setArguments(android.os.Bundle)">`setArguments()`</a> to assign a `Bundle` to that Fragment. Note that this is _different_ than the `Bundle` used to save the state!
- Then inside the Fragment's `onViewCreate()`, we can use `getArguments()` to access that `Bundle`, and `getType()` methods to fetch data out of it.
  - This data can then be used to dynamically set the text of our Views!

- Note that this means that every time we show a detail we're technically making a new Fragment. We could instead just call a method on the existing Fragment (saving a reference as an instance variable, say). E.g., give `MovieDetailFragment` a `setMovie()` method. But `arguments()` is considered a more "idiomatic" pattern.



## Dialogs (if time)
One last topic for today that will take advantage of our Fragments: [___Dialogs___](http://developer.android.com/guide/topics/ui/dialogs.html). A dialog is a "popup" modal (doesn't fill the screen) that either asks the user to make a decision or provides some additional information. Think like the `alert()` in JavaScript.

There is a base `Dialog` class, but mostly we use pre-defined versions:
- [`AlertDialog`](http://developer.android.com/reference/android/app/AlertDialog.html) is the most common version: a simple message with buttons you can respond to. Like `JOptionPane` if you used that.
  ```java
  // 1. Instantiate an AlertDialog.Builder with its constructor
  AlertDialog.Builder builder = new AlertDialog.Builder(getActivity());
  builder.setTitle("Alert!")
         .setMessage("Danger Will Robinson!");
  builder.setPositiveButton(R.string.ok, new DialogInterface.OnClickListener() {
    public void onClick(DialogInterface dialog, int id) {
      // User clicked OK button
    }
  });
  AlertDialog dialog = builder.create();
  dialog.show();
  ```
  - The `AlertDialog.Builder` class lets you create them. Then you can specify titles, messages, etc. Allows for easy customization of an Alert.
- [`DatePickerDialog`](http://developer.android.com/reference/android/app/DatePickerDialog.html) and [`TimePickerDialog`](http://developer.android.com/reference/android/app/TimePickerDialog.html) have pre-defined UI for picking things
  ```java
  TimePickerDialog dialog = new TimePickerDialog(getActivity(), new TimePickerDialog.OnTimeSetListener(){
      @Override
      public void onTimeSet(TimePicker view, int hourOfDay, int minute) {
          //...handle time picking...
      }
  }, 11, 50, false);
  dialog.show();
  ```

But the real workhorse, the one we care about, is [`DialogFragment`](http://developer.android.com/reference/android/app/DialogFragment.html). This acts as the "super class" for Dialogs that we'll be working with: by working with Fragments for dialogs, they can fit into the application's lifecycle and handle those events correctly.
- Basically, this can be a floating container for anything else you want to include!
- We subclass this in order to create custom dialogs. Can treat them as fragments (with `onCreateView()`, etc.)... but more common to just override `onCreateDialog()` and use that to "launch" the dialog.
  - Usually create an `AlertDialog` inside here which is then returned. So that you're still building on that, but you can encapsulate it all in your custom `DialogFragment`.
  - Examples: [here](http://developer.android.com/guide/topics/ui/dialogs.html#DialogFragment), [here](http://developer.android.com/reference/android/app/DialogFragment.html#AlertDialog), etc.

The other really neat trick is that a `DialogFragment` is just a Fragment... so can be used in all the same ways! So if you make your Fragment extend `DialogFragment` instead, you can instantiate that and then call `.show()` on the resulting Fragment to make it appear as a Dialog... while still being able to use a Transaction to embed it in a layout! See [Dialog or Embed](http://developer.android.com/reference/android/app/DialogFragment.html#DialogOrEmbed) for details.
```
fragment.show(getSupportFragmentManager(), "dialog");
```
- This doesn't look great without more formatting, but hey! Dialog!

...the truth is that Dialogs are not really that commonly used in Android (compared to other GUI systems). More likely to just change the Fragment/screen. And 80% of Dialogs that are used are AlertDialogs. But worth being aware of!


### Wrap-up
And that should cover the basics of Fragments for UI work (as well as some other stuff we can do with them).
- Homework involves making an app that uses a "dual-pane" fragment view: think about having a `leftContainer` and a `rightContainer` and then having your Activity deploy Fragments to the appropriate spots.
- And it will also involve some Database work... which we'll talk about on Thursday
  - Again, who needs SQL practice?


### Lecture References
- [Fragment Image][frag_img]
- [Fragment Lifecycle][frag_lifecycle]
- [`FragmentTransaction`](http://developer.android.com/reference/android/app/FragmentTransaction.html)
- [activity stack example][backstack_img]


<!-- Links -->
[frag_img]: http://developer.android.com/images/fundamentals/fragments.png
[frag_lifecycle]: http://developer.android.com/images/fragment_lifecycle.png
[backstack_img]: http://developer.android.com/images/fundamentals/diagram_backstack.png
