## Menus, Dialogs, & Loading Providers

### Admin [5min]
- Homework check-in
  - Sunspotter come together okay? Any questions/issues from that? Will it be sunny?
  - Next assignment is online: a classic to-do list app!
    - Two main components: fragments and dual-pane layouts (like you did in lab yesterday), and loading and saving data to a database. We'll talk about the later today briefly, but there is a pretty thorough explanation in the assignment as well.
    - This is going to feel like a really big app (possibly one of the biggest). Get started early!
- Fork and clone the repo for today; working off a new blank Activity with some placeholder text.
  - Includes a dynamically loaded fragment! (note it is loaded with `.replace()` as a hacky assurance that we don't get overlaps from instant-run)

Today is going to be a bit of a grab-bag of topics. Overall theme for this week is user interface components, but we're going to slip in some data loading stuff so that we have content to show in our user interface!
- Yesterday was Fragments, which let us separate out components into modules. Today we'll talk about other kinds of separations.
- Note we're aiming at exposure/familiarity, rather than in-depth mastery. So you are aware of some basic components, then you can look up in the documentation how to customize them to best fit your needs. _There are lots of customization options!_


## Menus [20min?]
Let's start with the most prominent visual component in our default app: the [___App Bar___](http://developer.android.com/training/appbar/index.html) or ___Action Bar___. This acts as the sort of "header" for your app, providing a dedicated space for navigation and interaction (e.g., menus!).
- The `AppCompatActivity` that we've been using provides this, but we can also add it ourselves (e.g., if we're not using the subclass)
- The [`ActionBar`](http://developer.android.com/reference/android/support/v7/app/ActionBar.html) is a generalization of a [`ToolBar`](http://developer.android.com/reference/android/support/v7/widget/Toolbar.html), which is basically how you can add "action bars" where-ever you want (e.g., in the middle of your app, or if you want it to stick to the bottom.
  - To add your own Action Bar, you specify a _theme_ without an ActionBar, and then include a `<android.support.v7.window.Toolbar>` element inside your layout wherever you want the toolbar to go. See [Setting up the App Bar](http://developer.android.com/training/appbar/setting-up.html) for details; we'll return to this later when we talk about Themes and Styles.

We can get access to the ActionBar by calling the `getSupportActionBar()` method. We can then call utility methods on it to interact with it; for example `.hide()` will hide the toolbar!

However, the main use for the ActionBar is a place to hold [___Menus___](http://developer.android.com/guide/topics/ui/menus.html). A Menu (specifically, an [**options menu**](http://developer.android.com/guide/topics/ui/menus.html#options-menu)) is a set of items (think buttons) that appear in the ActionBar.
- pre-API 11 they appeared at the bottom of the screen!
- Menus can be declared both in the `Activity` and in a `Fragment`; if they are in both places, they are combined into the ActionBar (so you can have menus be contextual pretty easily).

Menus, like all other user-facing elements, are defined as `xml` resources, specifically of type `menu`.
- You can create one by selecting `New > Android Resource File` and then the `Menu` Resource type. This gives you an xml file with a main [`<menu>`](http://developer.android.com/reference/android/view/Menu.html) element.

We can add items inside the XML menu, particularly [`<item>`](http://developer.android.com/reference/android/view/MenuItem.html) elements
- Common `<item>` attributes include: `id` (to refer to it later), `icon` (image to show), `title` (text to show), and `app:showAsAction` (for whether it should be listed in the ActionBar or collapsed under a button). See [Menu resources](http://developer.android.com/guide/topics/resources/menu-resource.html) for the full list of options!
  - Note that we can use the `android:drawable/ic_*` elements for built-in icons! [Android Drawables](http://androiddrawables.com/) includes the full list. Or we can make our own
- Example: We can make a `"say hello"` and a `"show list"` button.
- We can also add **one level** of submenus (`<menu>` inside an `<item>`). And we can also use `<group>` tags to group the items together. We can give the group an `id` to refer to the "group" of items, and if one item is hidden then the others will be as well. `orderInCategory` is useful for ordering within the group. See [the docs](http://developer.android.com/guide/topics/resources/menu-resource.html) for more details.

In order to get the menu to show up, we to tell the ActionBar which menu resource it should use (there may be a lot!) To do this, we override the `onCreateOptionsMenu` callback, and then use the `MenuInflater` to expand the menu:
```java
public boolean onCreateOptionsMenu(Menu menu) {
    MenuInflater inflater = getMenuInflater();
    inflater.inflate(R.menu.main_menu, menu);
    return true;
}
```
- This is similar in concept to how our Fragment's `onViewCreated()` method would inflate the Fragment into the XML. In this case, we're inflating into the ActionBar.

We then respond to these items being selected by filling in the `onOptionsItemSelected()` callback. Use a `switch` on the `item.getItemId()` to determine what item was selected, and then act accordingly.
- On default, pass up to `super`!
```java
public boolean onOptionsItemSelected(MenuItem item) {
    switch(item.getItemId()){
        case R.id.menu_item1 :
            //do thing;
            return true;
        default:
            return super.onOptionsItemSelected(item);
    }
}
```

But wait there's more! We can also add [Action Views](http://developer.android.com/training/appbar/action-views.html) that are expandable widgets in the action bar (e.g., search example). Or we can add an [Action Provider](http://developer.android.com/training/appbar/action-views.html#action-provider) (like [`ShareActionProvider`](http://developer.android.com/training/sharing/shareaction.html)), which gives us a bunch of interaction built into the menu!
- We'll talk about these more next week when we start working with Intents

### Context Menus (if time; end by 11:00)
In addition to ActionBar Options menus we can also use menus as contextual pop-ups (on long-press).
- start by using `registerForContextMenu()`, passing it the `View` we want to be able to create the menu for
- We then specify to menu to use through the `onCreateContextMenu()` callback, just like we did for Option menus!
  - We can even pass it the same menu if we wanted; this is part of why it's defined in XML!
- We respond to _those_ items being selected with the `onContextItemSelected()` (see a pattern)? So basically works the same way, but different set of callbacks.

That is a very brief introduction to menus. There are lots more options and complex things you can do with them, and I _highly_ recommend reading through the guide just so you can see what some of your options are!
- Alternatively, look at at any app you've used and think "how did they do that?" There is most likely a documented way!


## Dialogs [30min]
We made this "say hello" menu item that logs out some text, but wouldn't it be nice to show that to the user? Maybe as a kind of pop-up? Well there are a couple of different ways to create pop-up messages, and many of them involve Fragments (so we can review that as well!)

A [___Dialog___](http://developer.android.com/guide/topics/ui/dialogs.html) is a "popup" modal (doesn't fill the screen) that either asks the user to make a decision or provides some additional information. Think like the `alert()` in JavaScript.

There is a base `Dialog` class, but almost always we use a pre-defined version instead (similar to how we used `AppCompatActivity`). [`AlertDialog`](http://developer.android.com/reference/android/app/AlertDialog.html) is the most common version: a simple message with buttons you can respond with. Like `JOptionPane` if you've used that.

We don't actually instantiate an `AlertBuilder` directly. Instead we're going to use a helper _factory_ class called an `AlertDialog.Builder`.
- We instantiate a new builder (for this context)
- We can then set the title of the dialog, the messages, etc.
- We can also specify what happens when we click on individual buttons--the "positive" button is normally "OK" but we can customize the text
- We then create the actually `AlertDialog` with the `builder.create()` method, and finally remember to **`show()`** that dialog!

```java
AlertDialog.Builder builder = new AlertDialog.Builder(this);
builder.setTitle("Alert!")
       .setMessage("Danger Will Robinson!"); //note chaining
builder.setPositiveButton(R.string.ok, new DialogInterface.OnClickListener() {
  public void onClick(DialogInterface dialog, int id) {
    // User clicked OK button
  }
});

AlertDialog dialog = builder.create();
dialog.show();
```

[[If time: **practice reading API**: can you add a "cancel" button to the alert?]]


Note that there are other kinds of dialogs as well: [`DatePickerDialog`](http://developer.android.com/reference/android/app/DatePickerDialog.html) and [`TimePickerDialog`](http://developer.android.com/reference/android/app/TimePickerDialog.html) have pre-defined UI for picking date and time.
- You'll need to use these for your homework, but they work in pretty much the same way and the documentation has lots of example. I'll leave implementing them as an exercise for the reader, since part of the skill you need to develop is being able to learn new widgets and components!


### DialogFragments
What we've done will show a dialog, but has a few problems with how it interacts with the rest of the framework---namely, with the lifecycle of the Activity in which it is embedded.

For example, what happens if we rotate the screen while the dialog is being shown? (Rotating the display causes the Activity to be destroyed and recreated; it's `onCreate()` method will be called again).
- An exception is thrown and the dialog is lost!

To avoid these problems, we need to have a way of giving that dialog its own lifecycle which can interact with the the Activity's lifecycle... sort of like making it a modular piece of an Activity...
- That's right, we're going to use `Fragments`! Specifically, we'll use a subclass of Fragment called [`DialogFragment`](http://developer.android.com/guide/topics/ui/dialogs.html#DialogFragment).

To do this, just like with the `Fragments` we did yesterday, we'll need to create our own subclass of `DialogFragment`. We'll just make it a nested class for now (since it's not doing a lot).
- We'll then override the `onCreateDialog()` callback: this is the method that gets called when we want to show the Fragment as a dialog.
- This method returns a `Dialog`, so we'll need one of those... which we can create with the `AlertDialog.Builder` class!
```java
public static class MyDialogFragment extends DialogFragment {
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        AlertDialog.Builder builder = new AlertDialog.Builder(getActivity());
        //...
        AlertDialog dialog = builder.create();
        return dialog;
    }
}
```

Finally, we can actually show this `DialogFragment` by instantiating it and then calling the `.show()` method on it to make it show as a dialog.
- And now we can rotate the phone and everything works fine!

Here's the other neat trick: a `DialogFragment` is just a `Fragment`. That means we can use it _anywhere_ we normally used `Fragments`... including embedding them into layouts!
- If we make our `WordListFragment` into a `DialogFragment`, it will still be able to be used the exact same way (because it's still a `Fragment`, just with extra features now).
- And one of those features is to show it as a dialog using `.show()`!
- use ```setStyle(DialogFragment.STYLE_NO_TITLE, android.R.style.Theme_Holo_Light_Dialog);``` to make it look neat :)

Aside: notice that we're not calling the constructor, but instead a static _factory_ method that instantiates the Fragment for us. this lets us easily specify arguments to pass into it (e.g., as parameters to _this_ method), and those arguments will persist over time.

So those are Dialogs!
- ...the truth is that Dialogs are not really that commonly used in Android (compared to other GUI systems). Apps are more likely to just change the Fragment/Activity/screen being shown. And 80% of Dialogs that are used are `AlertDialogs`. But the process is worth being aware of!

## Toasts
`Dialogs` are pretty "heavy", but in terms of their interaction and the effort required to program them:
- E.g., they stop all other interaction to show the user a message

However, sometimes you want a "pop-up" message that isn't quite as prominent and doesn't require the user to click "okay". A simple, quick way of giving some short visual feedback is to use what is called a [Toast](http://developer.android.com/guide/topics/ui/notifiers/toasts.html). This is a tiny little text box that pops up at the bottom of the screen for a moment.
- Toast because it pops up :p

Toasts are pretty simple to implement, as with the following example (from the docs):
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
- Note that this uses a static `makeToast()` method, rather than a constructor. This is another example of that Factory method!
- Toasts are intended to be ways to interact with the user (e.g., giving them quick feedback), but can possibly be useful for testing too! Though in the end, Logcat is going to be your best bet for debugging.

### Break!

## Providers and Loaders
For the rest of the lecture we're going to focus on the `WordListFragment`, though these concepts apply to any Fragment or Activity.

This Fragment includes a `ListView` that shows a list of words. Recall that `ListView` follows the **model-view-controller** architecture... and in this case, our "model" (data) is a list of words that I hard-coded in.
- But there are other lists of words as well! Databases of them! Even on your phone!
- For example, Android keeps track of the list of the spellings of "non-standard" words in what is called the **User Dictionary**
  - We can see this list at `Setting > Language & Input > Personal dictionary`. We can even add words to it (`embiggen`, `cromulent`, `fleek`)

Note that the User Dictionary keeps a [**database**](http://developer.android.com/guide/topics/providers/content-provider-basics.html#Basics) of words. You can think of this as like an SQL table: it's a set of _entries_ (rows) each of which have some values (columns), on of which is likely an `ID` (primary key)
- How many people have taken 340 or otherwise worked with a relational database? Have you ever written a `SELECT` statement? (hopefully many)

What if we want to access this list of words (to show them in our ListView)? We can do that!
- First, we're going to need to request permission to access this list. This is similar to how we asked permission to use the Internet
  ```xml
  <uses-permission android:name="android.permission.READ_USER_DICTIONARY">
  ```

### Content Providers
So these words are stored in a database or some kind that we'd like to access. But we don't know the format of this database, and don't want to have to write specific code to deal with that exact format (e.g., is it an SQLite database or a file?)
- Android does include support for working directly with SQLite databases... but that's a lot of work that we don't want to deal with (though we'll look at it eventually maybe)

Because we don't want to deal with the specific format of how the data is stored, Android offers an _abstraction_ in the form of a [**Content Provider**](http://developer.android.com/guide/topics/providers/content-providers.html). A Content Provider offers an interface to interact with structured data, whether that data is stored in a database, in a file, online, whatever.
- You can think of a Content Provider as a "Data Source".
- We'll talk about defining these later in the course, but for now we're just interested in _using_ them.

A Content Provider is basically a "data source" that has a **URI** (Universal Resource Indicator: think URL but not necessarily on the Internet) that you can query, similar in concept to how you get data from a website by sending a query. In particular, these Uri's specify the `content://` protocol, allowing you to access the data as content (rather than, say, HTTP).
- The "URI" for the Dictionary's content is at `UserDictionary.Words.CONTENT_URI`. We don't care about the exact path, because that's stored as a constant!

We are able to access this provider via a [`ContentResolver`](http://developer.android.com/reference/android/content/ContentResolver.html), which is a class used to access the data stored in `ContentProviders`. We fetch the `ContentResolver` for the current Context (e.g., Activity) by calling `getContentResolver()` (note that for a Fragment we'll need to fetch the Context with `getActivity()`).
- The `ContentResolver` contains various "CRUD" style methods we can use to access the data stored in that provider (whatever format it takes): `.insert()`, `.query()`, `.update()`, `.delete()`. These are named after equivalent commands in databases.

Let's take a look at the <a href="http://developer.android.com/reference/android/content/ContentResolver.html#query(android.net.Uri, java.lang.String[], java.lang.String, java.lang.String[], java.lang.String)">`query()`</a> method:
```java
getContentResolver().query(
    uri,              // The content URI
    mProjection,      // The columns to return for each row
    mSelectionClause  // Selection criteria
    mSelectionArgs,   // Selection criteria
    mSortOrder);      // The sort order for the returned rows
```
This is basically a wrapper around an SQL `SELECT` statement.
- We start by specifying the URI for the Provider (data source)
- We then specify a **projection**. This is a `String[]` of all the "columns" (attributes) we want to fetch from the data source. This is what you'd put instead of the `*` in SQL. (Note we can pass in `null`, but that's inefficient--better to give a list of everything).
  - We can see what column names are available for the User Dictionary in [`UserDictionary.Words`](http://developer.android.com/reference/android/provider/UserDictionary.Words.html)
  - Make sure you grab `_ID`; we'll need it for later!
- We can then specify the selection criteria. This is what goes after `WHERE` in an SQL statement. Leave null for "all rows"
- After that we specify values to enter into that `WHERE` statement, which will be escaped against SQL injection attacks. null for nothing
- Finally, we can pass in a sort order (`ORDER BY` in SQL). null for nothing.

So overall, the query is breaking apart an SQL Select statement into different pieces as parameters to a method, so you don't _quite_ have to write the selection yourself.

### Cursors
What kind of object does this method return? A [`Cursor`](http://developer.android.com/reference/android/database/Cursor.html). A `Cursor` is a lot like an `Iterator` in Java (have you worked with those?) It basically represents a set of entries from a database, and it keeps track of where you are in a list (e.g., what `i` would be on in a loop). It then provides methods that let us fetch values from the object at that spot in the list. For example:
```java
cursor.moveToFirst(); //move to the first item
String field0 = cursor.getString(0); //get the first field (column you specified) as a String
String name = cursor.getString(cursor.getColumnIndexOrThrow("word")); //get the "name" field as a String
cursor.moveToNext(); //go to the next item
```
- Again, we can see what columns are available at `UserDictionary.Words`

The nice thing about `Cursors` though is that they can easily be fed into `AdapterViews` by using a [`CursorAdapter`](http://developer.android.com/reference/android/widget/CursorAdapter.html) (as opposed to the `ArrayAdapter` we've used previously). The [**`SimpleCursorAdapter`**](http://developer.android.com/reference/android/widget/SimpleCursorAdapter.html) is a concrete implementation that is almost as easy to use as an `ArrayAdapter`.
- You instantiate a new `SimpleCursorAdapter`, passing it:
   -  a `Context`
   -  a layout resource to inflate,
   -  a `Cursor` (which can be `null`)
   -  an array of column names to fetch from each entry in the Cursor (the **projection**)
   -  a matching list of view resource ids (which should all be `TextViews`) to assign each column to.
   -  any flags (`0` means no flags, and is the correct option for us).

- Then we can set this adapter just like we did with `ArrayAdapter`!

### Loaders
In order to get the `Cursor` to pass into the adapter, we need to `.query()` the database. But we'll be doing this a lot (and so would like to do it off the UI Thread---database accessing is slow!). And every time we do that query, we also want to update the `Adapter` so that the changes to the list show up!

In order to easily update your list with new data loaded on a background thread, we're going to use a class called a [`Loader`](http://developer.android.com/guide/components/loaders.html). This is basically a wrapper around `ASyncTask`, but one that let's you quickly specify what action should occur in the background thread. In particular, Android provides a [`CursorLoader`](http://developer.android.com/reference/android/content/CursorLoader.html) specifically used to load Cursors, which can then be "swapped" into the adapter.

To use a `CursorLoader`, we need to specify that our _Fragment_ implements the [`LoaderManager.LoaderCallback<Cursor>`](http://developer.android.com/reference/android/support/v4/app/LoaderManager.LoaderCallbacks.html) interface---basically saying that this fragment can react to Loader events.

  - (Loaders need to work with Fragments. If you have your Activity extends [`FragmentActivity`](http://developer.android.com/reference/android/support/v4/app/FragmentActivity.html), you can get the "Fragment" capabilities to use a `Loader` in an Activity). Note that `AppCompatActivity` is a subclass of this, so everything can work for Activities as well!

We then fill in the callbacks to write the code to use the `CursorLoader`.
- In `onCreateLoader()` we specify what the Loader should _do_. Here we would instantiate and return a `new CursorLoader(...)` that queries the `ContentProvider`. This looks a lot like the `.query()` method we wrote earlier, but will run on a background thread!

- In the `onLoadFinished()` callback, we can `swap()` the `Cursor` into our `SimpleCursorAdapter` in order to feed that model data into our controller (for display in the view). See the [guide](http://developer.android.com/guide/components/loaders.html) for more details.

- In the `onLoaderReset()` callback just swap in `null` for our Cursor, since there now is no content (it is "reset").


Finally, in order to actually _start_ our background activity, we'll use the `getLoaderManager().initLoader(...)` method. This is similar in flavor to the `AsyncTask.execute()` method we've used before (using a manager similar to the `FragmentManager`).
```java
getLoaderManager().initLoader(0, null, this);
```
- The first parameter to the `initLoader()` method is an id number for _which cursor you want to load_, and is passed in as the first param to `onCreateLoader()` (or is accessible via `Loader#getId()`).
- Second param is a `Bundle` of args, and the third is the `LoaderCallbacks` (e.g., who handles the results)!
- Note that you can use the `.restartLoader()` method to "recreate" the `CursorLoader`, like if you want to change the arguments passed to it.

And with that, we can fetch the words from our database on a background thread--and if we update the words it will automatically change!

### Adding Words (if time)
To `insert` a new Word into the `ContentProvider`, we just call a different method on the `ContentResolver`:
```java
//Example from Google:
ContentValues mNewValues = new ContentValues();
mNewValues.put(UserDictionary.Words.APP_ID, "edu.uw.decorations");
mNewValues.put(UserDictionary.Words.LOCALE, "en_US");
mNewValues.put(UserDictionary.Words.WORD, word);
mNewValues.put(UserDictionary.Words.FREQUENCY, "100");

Uri mNewUri = getContentResolver().insert(
    UserDictionary.Words.CONTENT_URI,   // the user dictionary content URI
    mNewValues                          // the values to insert
);
```
- Note that we specify the "details" of the Word in a [`ContentValues`](http://developer.android.com/reference/android/content/ContentValues.html) object, which is a HashMap almost exactly like a `Bundle` (but only supports values that work with `ContentProviders`)

Whew! And that covers what you should need for the next assignment: you'll be working with a `ContentProvider` I've provided, and doing some work to display its content, insert new items into it, and update the items that are there---all in a 2-pane layout (but you don't need to worry about switching between landscape and portrait!)

Questions?

### Lecture References
- [`ActionBar`](http://developer.android.com/reference/android/support/v7/app/ActionBar.html)
- [Menu resources](http://developer.android.com/guide/topics/resources/menu-resource.html)
- [Android Drawables](http://androiddrawables.com/)
- [Action Views](http://developer.android.com/training/appbar/action-views.html)
- [___Dialog___](http://developer.android.com/guide/topics/ui/dialogs.html)
- [`AlertDialog`](http://developer.android.com/reference/android/app/AlertDialog.html)
- [User Dictionary DB](http://developer.android.com/guide/topics/providers/content-provider-basics.html#Basics)
- [`UserDictionary.Words`](http://developer.android.com/reference/android/provider/UserDictionary.Words.html)
- [`ContentResolver`](http://developer.android.com/reference/android/content/ContentResolver.html)
