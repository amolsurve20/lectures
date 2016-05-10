## Content Providers

### Admin
- One more check-in for homework and project proposals. Questions? Issues?
  - I know it's getting towards the end of the year and the sun is out, but stay the course!
- Fork & clone repo for today. There is quite a bit of provided code that we'll review

### Review: Content Providers
Today we're going to talk in depth about [**Content Providers**](http://developer.android.com/guide/topics/providers/content-providers.html) and [**Databases**](http://developer.android.com/training/basics/data-storage/databases.html). You've actually done work with these before (quite a lot in fact: both in the Todoer and in the Yama assignment). But in those assignments you just used data stores that were given to you; today we're going to talk about how to make your own database and provider for it---that is, how that `TodoListProvider` was created!

- We've spent quite a bit of time emphasizing how to store and access data on a mobile device (and perhaps less time on how to add different user interfaces, though we'll return to that more next week). This is partly because, as we're in the iSchool, we're interested in how information and data is accessed and utilized. User interfaces and fancy widgets _are_ important, but I'm leaving the design decisions on how to utilize those up to you.

As you may recall, a **Content Provider** is a an abstraction for a source of structured data (like a database, but also possibly files, internet resources, etc). They represent a "data source" that we can read, add to, update, or delete data from (the basic [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) operations).

The starter code for today has a complete app that accesses and modifies the data represented by a Content Provider: specifically, the phone's User Dictionary. We actually used this example previously when we first looked at [`Loaders`](http://developer.android.com/guide/components/loaders.html); I've taken the "completed" branch and refactored it a bit for our purposes.

To make sure we understand how to _use_ a Content Provider, let's review the provided code:

Our layout provides a `ListView` and a button for text entry.
- The `ListView` is backed by a `SimpleCursorAdapter`, which takes a "cursor" (think: an Iterator) and connects each item from that data set (in memory) to a `View` on the screen
  - In particular, we're showing the word (`WORD`) and the "frequency"/prominence (`FREQUENCY`) of that word.

In order to get this list of items into memory from data store itself, we set up a `Loader`.
- This loader fetches 3 "columns" (attributes) from the data store: `_ID`, `WORD`, and `FREQUENCY`. This does mean that we're fetching a column (`_ID`) from the database that is _not_ shown on the View---but the adapter needs that column to figure out how to show things
- We don't have any selection or sorting criteria, though we could add them in if we wanted
- This loader will fetch data from the data store, loading it into memory. It then tells the `adapter` to take that loaded data and put it on the screen.
- Why are we using a `Loader` to do this? Because it will do two things for us: it will fetch the data from the data store _on a background thread_ (not on the UI thread), and it will automatically _reload_ the data when the data store's content changes.

That covers the **read** operation. But the provided code also supports two other operations:
- When we click the "add word" button, we can `insert()` a new entry into the data store.
  - We construct a `ContentValues` (like a `Bundle`) that contains the columns/attributes for the new entry
  - And then use the `ContentResolver` to insert this item into the `ContentProvider` at our target `URI`.
  - (demo)
- Additionally, when we click on an item in the list, we can `update()` that item.
  - We use the `AdapterView` to get the `Cursor` (again: think "iterator") representing the item that was clicked. In this case, the cursor will be pointed at the item that was clicked on.
  - We can fetch data from that for things like logging out
  - We can also use the `ContentResolver` to update this particular entry:
    - Construct a `ContentValues` bundle with _only_ the fields we want to change
    - And then we need to make sure we only update a single item. The easiest way to do that is to specify the `URI` _of that item_. We do this by constructing a new URI representing that item (effectively appending `"/id"` to the URI). This means that we don't need to use the selection criteria (though we could do that as well).
  - (demo)

Again, to review: the `ContentProvider` acts as the data store, keeping information somewhere away from our app (e.g., in a database). The `Loader` grabs a bunch of rows and columns from that database, and hands it to the `Adapter`. The `Adapter` then takes a subset of those rows and columns and puts them in the `ListView` so that they display to the user. The buttons then let data from the `Views` get put back into the data store.

Any questions about this process? (I'm sorry we didn't go over this more clearly before the Todoer assignment!)


### SQLite Databases
Content Providers can abstract all kinds of data stores (files, urls, etc.). They abstract these as a _structured information_ similar to a database... and in fact the most common kind of store they represent is a [relational database](https://en.wikipedia.org/wiki/Relational_database_management_system) (specifically, an [SQLite database](https://en.wikipedia.org/wiki/SQLite)). Android [comes with an API](http://developer.android.com/guide/topics/data/data-storage.html#db) for creating an querying a database; these databases are stored on [_internal storage_](http://developer.android.com/guide/topics/data/data-storage.html#filesInternal) meaning that each application can have its own private database (or multiple databases, in fact)!

- If you've taken INFO 340 or have otherwise worked with a database such as `MySQL`, this interaction will seem familiar. If you've never worked with a database, the simplest explanation is to think of them as a spreadsheet (like an Excel file) where you manipulate _rows_ of data given a set of pre-defined columns. [SQL](https://en.wikipedia.org/wiki/SQL) (Structured Query Language) is its own command language for working with these spreadsheets; we'll see some samples of those queries in class today. The full spec for SQLite's version of SQL can be found [here](http://sqlite.org/lang.html).

- I've also included a [short tutorial](SQLite-Tutorial.pdf) (borrowed from Google) as a `.pdf` in this repository, in case you want the refresher.

To walk through how this works, let's go ahead and build our _own_ database of words (separate from the User Dictionary) that we can access through a Content Provider (e.g., just change which "data store" we're querying; the rest of the interface will be the same!) This will let us demonstrate how to put together a Content Provider from scratch.

- We'll start with setting up the database, and then build the `ContentProvider` that uses it. Note that setting up a database is a bit wordy and round-about, though there aren't really a lot of new concepts.

The first thing we're going to do is make a class (e.g., `WordDatabase`) to act as a "namespace" for the various pieces of our database (we're not going to instantiate it; we could make a _private_ default constructor to keep it from being instantiated ever).

We're going to start off by making a _bunch_ of constants:
- `DATABASE_NAME` to refer to the name of the database file (e.g., `words.db`)
- `DATABASE_VERSION` to refer to the current version number of our database's schema. This is used more for supporting [migrations](https://en.wikipedia.org/wiki/Schema_migration) like if we want to update our database later, but we'll mostly ignore that for now (this isn't a databases class).

We're then going to specify constants that define the database's **schema** or **contract**. This is so that other classes (e.g., the `ContentProvider` and the `MainActivity`) can refer to column names consistently without having to remember any weird names we give it. Similar to how we used `UserDictionary.Words.WORD` rather than the String `"word"`.
  - (These were the values that I had refactored into `TodoItem` in the homework to try to keep things clear. We saw how that worked out).

- By convention, we actually will define this schema as a separate _`static` nested class_ (e.g., `WordEntry`), to keep things organized. It will then contain constants to hold the column names:
  ```java
  static class WordEntry implements BaseColumns {
       //class cannot be instantiated
       private WordEntry(){}

       public static final String TABLE_NAME = "words";
       public static final String COL_WORD = "word";
       public static final String COL_COUNT = "count";
  }
  ```
- We will have this class implement [`BaseColumns`](http://developer.android.com/reference/android/provider/BaseColumns.html), which lets it get a couple of framework specific constants for free--in particular `_ID` which `ContentResolvers` rely on the database to have.

Now we're ready to actually create and work with the database. In order to help us do this, we're going to use a class called [`SQLiteOpenHelper`](http://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper.html). This class offers a set of methods to help manage the database being created and upgraded (e.g., for migrations).
- We do this by making _another_ inner class that will extend the base `SQLiteOpenHelper` (because of course we do).
- We'll make a constructor that just takes in a `Context` and then passes up the database name and version:
  ```java
  public DatabaseHelper(Context context){
      super(context, DATABASE_NAME, null, DATABASE_VERSION);
  }
  ```

We then have two methods we need to overwrite: what happens when the database is created, and what happens when the database is upgraded.
- When the database is first created, we'll need to actually create the table (sheet) to hold our words. This involves sending it an `SQL` command to create the table! We'll write this out as yet another constant for readability.
- We can do the same for dropping (deleting) the table as well
```java
  private static final String CREATE_TASKS_TABLE =
    "CREATE TABLE " + WordEntry.TABLE_NAME + "(" +
        WordEntry._ID + " INTEGER PRIMARY KEY AUTOINCREMENT" + ", "+
        WordEntry.COL_WORD + " TEXT" + ","+
        WordEntry.COL_COUNT + " INTEGER" +
        ")";

  private static final String DROP_TASKS_TABLE =
      "DROP TABLE IF EXISTS "+ WordEntry.TABLE_NAME;
```

We can run these `SQL` statements by using the `db.execSQL()` method, called on the [`SQLiteDatabase`](http://developer.android.com/reference/android/database/sqlite/SQLiteDatabase.html) object that is passed to us. Note that this runs a "raw" SQL query (that isn't a `select`--so one that doesn't return anything), without any kind of checks against SQL injection attacks. But since we're hard-coding the information to run, it's not a problem (we'll otherwise _never_ use this method).
- We can also use the `.insert()` method to add some sample words, for clarity of testing:
  ```java
  ContentValues sample1 = new ContentValues();
  sample1.put(WordEntry.COL_WORD, "Embiggen");
  sample1.put(WordEntry.COL_COUNT, 0);
  db.insert(WordEntry.TABLE_NAME, null, sample1);
  ```
- `onUpgrade` we'll just drop the table and recreate.

Now if we want to interact with this database, we can initialize the `DatabaseHelper` (which will create the database as needed) and then use that to fetch the database we want to query (using `getReadableDatabase()`)
- Note that this could take a long time, and so we should _not_ be doing it on the UI thread... but we will just for testing/demo purposes

We can check that our database is set up correctly in one of two ways:
- We can directly explore the SQLite database that is on your device by using `adb` and the `sqlite3` tool. See [this link](http://developer.android.com/tools/help/sqlite3.html) for more details.
    ```
    $ adb -s emulator-5554 shell
    # sqlite3 /data/data/edu.uw.package.name/databases/words.db
    # sqlite> select * from words;
    # sqlite> .exit
    ```
- We can call a `query()` method on our `SQLiteDatabase`, and log out the results. A `SQLiteQueryBuilder` can offer some help if our query was going to be complex (e.g,. with `JOIN`):
  ```java
  SQLiteQueryBuilder builder = new SQLiteQueryBuilder();
  builder.setTables(WordDatabase.WordEntry.TABLE_NAME);

  Cursor results = builder.query(
          db,
          new String[] {WordDatabase.WordEntry.COL_WORD, WordDatabase.WordEntry.COL_COUNT},
          null, null, null, null, null); //5 nulls!

  while(results.moveToNext()){
      String word = results.getString(results.getColumnIndexOrThrow(WordDatabase.WordEntry.COL_WORD));
      int freq = results.getInt(results.getColumnIndexOrThrow(WordDatabase.WordEntry.COL_COUNT));
      Log.v(TAG, "'"+word+"' ("+freq+")");
  }
  ```
  - Note we copy in the cursor processing from above, but replacing with our column names!
  - We could even remove the `Loader` call and just pass in this query directly to the Adapter!

Voila, we have a database that we can call methods on to access!

#### Break?

### Implementing `ContentProvider`
Again, we don't want to do this database creation and querying on the main thread (because it might block!) And since we want to easily let our `ListView` update when the database changes, we want to use the `Loader`. And in order to use the `Loader`, we need to wrap the database in a `ContentProvider`. So let's do that.
- There are a lot of steps and a lot of code to making a `ContentProvider`, and most of them are pretty standard/common for most databases. So much so that there is [thorough example code](http://developer.android.com/guide/topics/providers/content-provider-creating.html) in the Google documentation, which we can copy-and-paste from as needed.

We'll start by creating another class that extends `ContentProvider` (_can you understand why?_).
- This will actually have a lot of abstract methods we'll need to fill in... so we can actually use one of Android Studio's generators to help us along (I normally say not to use these, but with the `ContentProvider` it's not too messy)
  - We will have to specify an [**authority**](http://developer.android.com/guide/topics/providers/content-provider-creating.html#ContentURI) for the Provider. This acts as a unique, Android-internal "name" (sort of like a package name)---and in fact, we usually use the package name with an extra `.provider` attached!
  - This is the "name" by which others will be able to refer to our particular provider (as opposed to some other provider).
- You'll notice that an entry for this `<provider>` has been added to our `Manifest`; similar to what you had to do for the Todoer.

#### Uris and Types
The most important piece of a `ContentProvider` (that makes it more than just helper methods for a database) is how it can be accessed at a particular [**Uri**](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier) (Uniform Resource Identifier). So the first thing we need to do is specify this Uri.
- We'll actually want to specify _multiple_ Uris. This is because each piece of content we Provide (each record in our database!) is itself a distinct resource, and thus should have its own Uri. Thus we need to design a _schema_ for the Uri so that we know how to refer to each kind of content offered by our provider.
- Designing a Uri schema is like designing a URL structure for a website (if you took 343 and used Angular Router to specify routes, this will feel familiar).

The most common Uri approach is to give each resource we provide a Uri of the format:
```
content://com.example.app.provider/resource/id
```
- This says: go to a particular provider, then go to a particular resource type (`resource`: think db table), then go to the particular resource (`id`, think row id)
- Leaving off the `id` would refer to the entire table, or rather the "list" of resources
  - So really we have two main kinds of Uris: the whole list, and an individual resource. Both have the same "base", but will need to be handled slightly differently.
- See [designing content URIs](http://developer.android.com/guide/topics/providers/content-provider-creating.html#ContentURI) for more discussion on how to structure these.

Let's begin putting these together... as constants of course:
- The base URI includes the **authority**, so let's go ahead and define a constant for that.
- We can also define the resource type we care about (which happens to be the name of the table, but it doesn't need to be!)
- Finally, we can define the overall base URI (the "Content URI") by combining these together and parsing it into a Uri object:
```java
public static final Uri CONTENT_URI = Uri.parse("content://" + AUTHORITY + "/"+WORD_RES);
```

But we also need to handle both types of resources: the "list" of words, and the individual words themselves. To enable this, we're going to use a class called a [`UriMatcher`](http://developer.android.com/reference/android/content/UriMatcher.html). This class provides a _mapping_ between Uri paths (think: URLs) and the actual "type" of data we're interested in (either lists or words).
- This helps us do the "routing" work, without having to parse the URL we got ourselves.
- We'll represent the "type" or "kind" with `int` constants; that we can easily refer to "which" kind of resource we're talking about.
  ```java
  //integer values representing each supported resource Uri
  private static final int WORD_LIST_URI = 1; // /words
  private static final int WORD_SINGLE_URI = 2;// /words/:id
  ```
  - So if you give me `/words`, I can tell you that you're interested in "resource kind \#1"

- We want to make a a `static UriMatcher` object (like a constant) that we can query... but because it takes more than one line to set this up, we need to put it inside a `static` block so that all this code is run together.
  ```java
  private static final UriMatcher sUriMatcher; //for handling Uri requests
  static {
      //setup mapping between URIs and IDs
      sUriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
      sUriMatcher.addURI(AUTHORITY, WORD_RESOURCE, WORD_LIST_URI);
      sUriMatcher.addURI(AUTHORITY, WORD_RESOURCE + "/#", WORD_SINGLE_URI);
  }
  ```
  - Note the wildcard `#`, meaning "any number" (after the slash) will "match" this Uri.

We can then figure out which "kind" of task by using the `UriMatcher#match(uri)` method, which will return the "kind" `int` that matches the given Uri.

As an example of this, let's fill in the `getType()` method (which I'll move to the bottom). The purpose of this method is to allow the `ContentProvider` to let whoever queries it know the [MIME Type](https://en.wikipedia.org/wiki/Media_type) (media type) of the resource a Uri is accessing.
- The idea is if we get some content from the provider, is that content an image? text? music? what?
- The type we're going to give back is `Cursors` (rows in a table), so we'll specify [MIME Types for that](http://developer.android.com/guide/topics/providers/content-provider-creating.html#TableMIMETypes):
  ```java
  switch(sUriMatcher.match(uri)){
      case WORD_LIST_URI:
          return "vnd.android.cursor.dir/"+AUTHORITY+"."+WORD_RESOURCE;
      case WORD_SINGLE_URI:
          return "vnd.android.cursor.item/"+AUTHORITY+"."+WORD_RESOURCE;
      default:
          throw new IllegalArgumentException("Unknown URI "+uri);
  }
  ```
  - `vnd` stands for "vendor specific"

#### Query Methods
Now that we have all our Uris specified, we can start responding to requests for content at those Uris. Specifically, when a request for content at a Uri comes in, we're going to fetch data from the _database_ we made earlier and then return that!
- We handle these "requests" through 4 different methods: `query()`, `insert()`, `update()`, and `delete()` (CRUD operations!). So we'll fill in those methods and have them fetch from the DB and return the results.
  - Can reorder for readability

First we need to get access to the database, which we'll do just like we did in the `MainActivity`.
- We'll instantiate the `DatabaseHelper` in `onCreate()`, saving it in an instance variable for later
  - can get Context for the provider with `getContext()`
- Then in the CRUD methods we can call `getWritableDatabase()` to get access to that database (off of the Main Thread!)

Let's start with `query()`. Basically, we need to do the same query we had before--so can just move it in (though can pass in the query params instead of always having them be `null`. This includes the `projection`)!
- But we need to be able to handle both types (lists or single words). So we can use the `UriMatcher` again to determine how to adjust our query
  - The easiet way to do this is to use the `UriBuilder#appendWhere()` method to add a "selection" argument:
  ```java
  switch(sUriMatcher.match(uri)){
      case WORD_LIST_URI: //all words
        break; //no change
      case WORD_SINGLE_URI: //single word
        builder.appendWhere(WordDatabase.WordEntry._ID + "=" + uri.getLastPathSegment()); //restrict to that item
      default:
        throw new IllegalArgumentException("Unknown URI "+uri);
  }
  ```
We'll then just return the `Cursor` that we get as a result of the query.

...but there is one more piece. We want to make sure that the `Loader` that is reading from our provider (handling this `Cursor`) is notified of any changes to its results:

```java
cursor.setNotificationUri(getContext().getContentResolver(), uri);
```

And now, we can go back to our `MainActivity` and swap all the column names and Uris for our own custom `WordProvider`!
- Rerun the app... and voila, we see our own list of words!

We can do basically the same thing to support `insert()` and `update()`.
- We can use the `UriMatcher` to make sure we only get proper Uris--you can't insert into a single record, and you can't update the entire list.
  ```java
  if(sUriMatcher.match(uri) != WORD_LIST_URI) {
    throw new IllegalArgumentException("Unknown URI "+uri);
  }
  ```
- We could also do work to make sure that no "empty" entries are added to the database
  ```java
  if(!values.containsKey(WordDatabase.WordEntry.COL_WORD)){
    values.put(WordDatabase.WordEntry.COL_WORD, "");
  }
  ```
- And return the result if it is successful
  ```java
  long rowId = db.insert(WordDatabase.WordEntry.TABLE_NAME, null, values);
  if (rowId > 0) { //if successful
      Uri wordUri = ContentUris.withAppendedId(CONTENT_URI, rowId);
      getContext().getContentResolver().notifyChange(wordUri, null);
      return wordUri; //return the URI for the entry
  }
  throw new SQLException("Failed to insert row into " + uri);
  ```

- Update is just sorta awkward because we need to basically add our `id` restriction to the user-given selection args:
  ```java
  int count;
  switch (sUriMatcher.match(uri)) {
    case WORD_LIST_URI:
      count = db.update(WordDatabase.WordEntry.TABLE_NAME, values, selection, selectionArgs); //just pass in params
      break;
    case WORD_SINGLE_URI:
      String wordId = uri.getLastPathSegment();
      count = db.update(WordDatabase.WordEntry.TABLE_NAME, values, WordDatabase.WordEntry._ID + "=" + wordId //select by id
            + (!TextUtils.isEmpty(selection) ? " AND (" + selection + ')' : ""), selectionArgs); //apply params
      break;
    default:
      throw new IllegalArgumentException("Unknown URI " + uri);
  }
  if (count > 0) {
    getContext().getContentResolver().notifyChange(uri, null);
    return count;
  }
  throw new SQLException("Failed to update row " + uri);
  ```

And now we have a working `ContentProvider`!! Lots of steps, but we could now store data in our own database and easily access it off the UI thread for use in things like `ListViews`. This is great for if you want to track and store any kind of structured information in your apps.


#### Lecture references
- [SQLiteDatabase API](http://developer.android.com/guide/topics/data/data-storage.html#db)
- [`SQLiteOpenHelper`](http://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper.html)
- [`BaseColumns`](http://developer.android.com/reference/android/provider/BaseColumns.html)
- [`SQLiteDatabase`](http://developer.android.com/reference/android/database/sqlite/SQLiteDatabase.html)
- [Provider example code](http://developer.android.com/guide/topics/providers/content-provider-creating.html)
- [Designing Content URIs](http://developer.android.com/guide/topics/providers/content-provider-creating.html#ContentURI)
