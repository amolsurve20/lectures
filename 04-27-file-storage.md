## File Storage

### Admin [5min]
- Yama get turned in? Questions that came up that I can address?
- Remaining questions about maps from lab yesterday?
  - Your homework this week is to build a location-based map drawer. Basically combining what we went over in class Monday with what you did on lab Tuesday. This should be a much more manageable/reasonable assignment than previous ones (it's smaller in scope).
  - This one is **not** a partner assignment!
- We have yet another repo for today (again, providing us an interface so we don't have to re-define it during class)
  - Did everyone manage to run the permissions demo from yesterday and see it working? (If not, we can repeat it!)

## File System Access
Last time we talked briefly about the permission system (which is something you'll need to deal with for your homework). Since permissions are significantly there to control access to the file system, today we're going to talk about [working with files](http://developer.android.com/training/basics/data-storage/files.html) in Android. This is a form of [Data Storage](http://developer.android.com/guide/topics/data/data-storage.html) we can use, in addition to the `SharedPreferences` we talked about last week an the SQLiteDatabases we've access via a `ContentProvider`

Android devices split file storage into two types: **Internal storage** and **External storage**. These names come from when devices had built-in memory, as well as external SD cards (though "external storage" can refer to a part of internal memory as well).
- [Internal storage](http://developer.android.com/guide/topics/data/data-storage.html#filesInternal)) is always accessible, and by default files saved internally are _only_ accessible to your app. Similarly, when the user uninstalls your app, the internal files are deleted. This is usually the best place for "private" file data.
- [External storage](http://developer.android.com/guide/topics/data/data-storage.html#filesExternal) is not always accessible, is world-readable, and only gets removed on uninstall if you do it in a specific way (that we'll talk about!)
- [When do we use each?](http://developer.android.com/training/basics/data-storage/files.html#InternalVsExternalStorage)
  - Basically use _Internal_ for "private" files that you don't want to be available outside of the app, and use _External_ otherwise.
  - Note that there are publicly-visible _Internal_ files and publicly-hidden _External_ files; the big distinction will be about access.

In addition, both of these storage systems also have a **"cache"** location (so _Internal Cache_ and _External Cache_). A [cache](https://en.wiktionary.org/wiki/cache) is "(secret) storage for the future", but in computing tends to refer to "temporary storage"
- The Cache is different from other file storage, in that Android will automatically delete cached files if storage space is getting low... but you really shouldn't rely on it to do that (delete your own cache files when you're done with them!) Basically use this for temporary files, and try to keep them _small_ (less than 1MB).
  - The user can easily clear application cache as well

In code, using all of these systems involve working with the [`File`](http://developer.android.com/reference/java/io/File.html) class. This represents a "file" (or a "directory") object.
- This is the same class you've likely used in Java SE!
- We can instantiate a `File` by passing it a directory (which is another file!) and a filename (String). Note that instantiating the file will create it if it doesn't exist.
- Note that we can test if a `File` is a file with `.isDirectory()`, and we can create new directories by taking a file and calling `.mkdir()` on it. Can get a list of `Files` inside the directory with `listFiles()`.
- The difference between internal and external storage, _in practice_, basically involves which directory you save the file in!

### External Storage
We'll do [External Storage](http://developer.android.com/guide/topics/data/data-storage.html#filesExternal) as a demo, since the code is sort of a super-set of what you can do, and then talk about where to change things for Internal storage.
- E.g., we'll make it so we can save whatever the user typed in to a file!

First we're going to need permission! `WRITE_EXTERNAL_STORAGE`
  - This is a _dangerous_ permission, so we'd need to ask for permission again on Marshmallow... but to keep the demo focused we'll just keep our target SDK down at 21.

Next, we need to do is check if external storage is available (e.g., that the SD card is mounted). We do this with a check:
```java
public boolean isExternalStorageWritable() {
    String state = Environment.getExternalStorageState();
    if (Environment.MEDIA_MOUNTED.equals(state)) {
        return true;
    }
    return false;
}
```

Then we need to pick what directory to save the files in. With external storage, we have two options:
- We can save the file _publicly_. We use `getExternalStoragePublicDirectory()` to access this, passing it what [type](http://developer.android.com/reference/android/os/Environment.html#lfields) of directory we want (e.g., `DIRECTORY_MUSIC`, `DIRECTORY_PICTURES`, `DIRECTORY_DOWNLOADS` etc). This basically drops files into the same folders that everything else is accessing, great for shared data and common formats like pictures, music, etc.
- Alternatively, since API 18, we save the file _privately_, but still on external storage (these are world-readable, but are hidden from the user as media, so don't look like public files). We do this with the `getExternalFilesDir()` method, again passing it a _type_ (since we're basically making our own version of the public folders). We can also use `null` for the type, giving us the root directory
  - Since API 19 (4.4 KitKat), you don't need permission to write to private external storage. So can only get permission for versions lower than that:
  ```xml
  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
                   android:maxSdkVersion="18" />
  ```

We can actually look at the emulator's file-system and see our files by created using `adb`
- Connect with `adb -s emulator-5554 shell` (note: `adb` needs to be on your PATH).
  - Private external files can be found in `/storage/sdcard/Android/data/package.name/files` (on emulator, device path may vary)
  - Public external files can be found in `/storage/sdcard/Folder`

Once we've opened up the file, we can write content to it by using the same IO classes we've used in Java.
- The "lower level" version is to create a a `FileOutputStream` object (or a `FileInputStream` for reading). We just pass this the `File` to write to.
  - We write `bytes` to the stream... but can write a String by calling `myString.getBytes()`.
  - For reading, we'll need to read in _all_ the lines/characters, and probably build a String out of them to show. This is actually the same loop we used when reading data from an HTTP request!
- But we can also use the same decorators as in Java (e.g., `BufferedReader`, `PrintWriter`, etc) if we want those capabilities; it makes writing a little easier
- In either case, **remember to `.close()` the stream when done!** (to avoid memory leaks!)
```java
//writing
if(isExternalStorageWritable()){
    try {
        File dir = getExternalFilesDir(Environment.DIRECTORY_DOCUMENTS);
        File file = new File(dir, FILE_NAME);
        Log.v(TAG, "Saving to  "+file.getAbsolutePath());

        PrintWriter out = new PrintWriter(new FileWriter(file, true));
        out.println(textEntry.getText().toString());
        out.close();
    }catch (IOException ioe){
        Log.d(TAG, Log.getStackTraceString(ioe));
    }
}

//reading
if (isExternalStorageWritable()) {
    try {
        File dir = getExternalFilesDir(Environment.DIRECTORY_DOCUMENTS);
        File file = new File(dir, FILE_NAME);
        BufferedReader reader = new BufferedReader(new FileReader(file));
        StringBuilder text = new StringBuilder();

        //read the file
        String line = reader.readLine();
        while (line != null) {
            text.append(line + "\n");
            line = reader.readLine();
        }
        textDisplay.setText(text.toString());
        reader.close();
    } catch (IOException ioe) {
        Log.d(TAG, Log.getStackTraceString(ioe));
    }
}
```


#### Demo
We can demo this by having our "save" message write to the file, and then having our "read" message load the file (and display it on the screen!)

### Internal Storage & Cache
[Internal storage](http://developer.android.com/guide/topics/data/data-storage.html#filesInternal) works pretty much the same way. Remember that internal storage is always _private_ to the app.
- We don't need write permission for internal storage!

For internal storage, we can use `getFilesDir()` to get the files directory (just like we did with external)
  - This is for the folder `/data/data/package.name/files`

But _alternatively_, we can use `Context#openFileOutput()` (or `Context#openFileInput()`) and pass it the _name_ of the file to open. This gives us back the `Stream` object for that file in the `filesDir()`, without us needing to do any work (cutting out the middle-man!)
  - takes a second parameter: `MODE_PRIVATE` will create the file (or replace a file of the same name). Other modes available are: `MODE_APPEND` (which adds to the end of the file if it exists instead of erasing). (`MODE_WORLD_READABLE`, and `MODE_WORLD_WRITEABLE` are deprecated).
  - You can wrap a `FileInputStream` in a `InputStreamReader` in a `BufferedReader`.

Can access internal cache with `getCacheDir()` (and same process), or external cache with `getExternalCacheDir()` (but usually use internal cache, because why would you make it world-readable? Well for pictures maybe, but we'll skip that)

And again, once you have the file, you use the same process for reading and writing as _External_ storage.

#### Demo
We can have our toggle support an Internal file as well! Will be a _different_ file.
- Ideally we could refactor this to avoid duplicated code... but involves repeated try/catch so will skip.


## Sharing Files
Once we have these files, we can share them with other apps! How do we communicate with other apps? `Intents`!
- Let's craft an _implicit intent_ for `ACTION_SEND`. We'll set our type as `text/plain` (cause we're sending a text file)
- We can then attach the file as an extra---specifically `EXTRA_STREAM`.
  - The extra won't actually be the `File`, but a **Uri** (recall: the "url" or location of the file). We're not sending the file itself, but the _location_ of that file (because it's smaller data to fit in the envelope).
  - We can get this Uri with `Uri.fromFile(File)`.
  ```java
  File dir = getExternalFilesDir(Environment.DIRECTORY_DOCUMENTS);
  File file = new File(dir, FILE_NAME);
  fileUri = Uri.fromFile(file);
  ```

Since multiple activities may support this action, we can then wrap the intent in a "chooser" to force the user to pick which Activity to use:
```java
Intent chooser = Intent.createChooser(shareIntent, "Share File");
//check that there is at least one option
if (shareIntent.resolveActivity(getPackageManager()) != null) {
    startActivity(chooser);
}
```
- This works with email if that is set up; doesn't attach to Messenger though; we'll do another example in a second that will (mostly)!

What happens if we try and share an Internal file? You'll get an error (actually notified the use!), because the other (email) app doesn't have permission to read that file!
- There is a way around this though, and it's by using a `ContentProvider` (haha!) A `ContentProvider` explicit is about making content available outside of a package (that's why we declared it in the `Manifest`). Specifically, a `ContentProvider` can convert a set of `Files` into a set of data contents (e.g., accessible with the `content://` protocol) that can be used and returned and understood by other apps!
  - Kind of like a "File Server"
- Android includes a [`FileProvider`](http://developer.android.com/reference/android/support/v4/content/FileProvider.html) class in the support library that does exactly this work.
  - There are a few details about setting one of these up in the lecture notes if you're interested, though we won't be using it or covering it (so just FYI).

Note that for your homework this week you **will** be asked to share files using a [ShareActionProvider](http://developer.android.com/reference/android/support/v7/widget/ShareActionProvider.html), which is basically a menu item that gives quick access to choosing which app to send the `Intent` to. There are details in the assignment write-up, as well as in last week's lecture notes.


## Saving Pictures
As another example of how we might use the storage system (because it's fun and we have some time), let's go back to our "taking picture" system from last week. We have this in a separate `Activity` (accessible via the options menu).
- To review: we sent an `Intent` with the `MediaStore.ACTION_IMAGE_CAPTURE` action, and the _result_ of that `Intent` included an _Extra_ that was a `BitMap` of a low-quality thumbnail for the image.

But if we want to save a higher resolution version of that picture, we'll need to put it in the file system.
- To do this, we're actually going to modify the `Intent` we _send_ so it includes an additional Extra: a file in which the picture data can be saved. Effectively, we'll have _our Activity_ allocate some memory for the picture, and then tell the Camera where it can put the picture data that it captures.

Before we send the `Intent`, we're going to go ahead and create a file:
```java
File file = null;
try {
    String timestamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date()); //include timestamp

    File dir = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES);
    file = new File(dir, "PIC_"+timestamp);
    boolean created = file.createNewFile(); //actually make the file!
    Log.v(TAG, "File created: "+created);

} catch (IOException ioe) {
    Log.d(TAG, Log.getStackTraceString(ioe));
}
```

We will then specify an Extra to give that file's location to the camera: if we use `MediaStore.EXTRA_OUTPUT`, the camera will know what to do with that!
- Again, send it a _Uri_ for the file, via `Uri.fromFile(File)`
  - We'll also save this Uri in an instance variable so we can access it later!
- Then when we get the result back (in our `onActivityResult` callback), we can access that file and use it to display the image! The `ImageView.setImageUri()` is a fast way of doing that.

#### Decoding images (only demo if time)
Note that when working with images, we can very quickly run out of memory (because images can be huge!) So we'll often want to ["scale down"](http://developer.android.com/training/displaying-bitmaps/index.html) the images as we load them into memory.
- The `BitmapFactory` class provides a number of static methods to do this work, though specifying the options is somewhat complex.
- This kind of image processing can also take a while... so we'd want to do it off the background thread (like in an `AsyncTask`)!
  - Of course this can lead to further issues if you're trying to decode lots of images (like for a ListView), particularly if they might be disappearing while scrolling.

Third-party libraries like [Picasso](http://square.github.io/picasso/) can help ease this development (though the gradle install seems to be buggy in Android Studio 2.0)
- Can skip the demo, unless want to practice reading the details from the website.


### Sharing Images
We can also share this image file just like we did the text file. The only difference is we'll specify the data type as `image/*` to mark that this an image type.
- Note that the emulator has given me some problems (freezing) moving between Activities to share this image. But it works fine on a physical device (can demo that if needed).

And that covers the basics of the file-system. Any questions?
- If extra time, can review old topics?


### Bonus: Using a `FileProvider`
Setting up a `FileProvider` is luckily not too complex, though it has a couple of steps. You will need to declare the `<provider>` inside you Manifest (see the [guide link](http://developer.android.com/training/secure-file-sharing/setup-sharing.html) for an example).
```xml
<provider
    android:name="android.support.v4.content.FileProvider"
    android:authorities="edu.uw.mapdemo.fileprovider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/fileprovider" />
</provider>
```
The attributes you will need to specify are:
- `android:authority` should be your package name followed by `.fileprovider` (e.g., `edu.uw.myapp.fileprovider`). This says what source/domain is granting permission for others to use the file.
- The child `<meta-data>` tag includes an `androd:resource` attribute that should point to an XML resource, of type `xml` (the same as used for your SharedPreferences). _You will need to create this file!_ The contents of this file will be a list of what _subdirectories_ you want the `FileProvider` to be able to provide. It will look something like:
```xml
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <files-path name="my_maps" path="maps/" />
</paths>
```
The `<files-path>` entry refers to a subdirectory inside the Internal Storage files (the same place that `.getFilesDir()` points to), with the `path` specifying the name of the subdirectory (see why we made one called `maps/`?)

Once you have the provider specified, you can use it to get a `Uri` to the "shared" version of the file using:
```java
Uri fileUri = FileProvider.getUriForFile(context, "edu.uw.myapp.fileprovider", fileToShare);
```
(note that the second parameter is the "authority" you specified in your `<provider>` in the Manifest). You can then use this `Uri` as the `EXTRA_STREAM` extra in the Intent that you want to share!

And that about covers it!


#### Lecture References
- [When do we use each?](http://developer.android.com/training/basics/data-storage/files.html#InternalVsExternalStorage)
- [`File`](http://developer.android.com/reference/java/io/File.html)
- [Public File Directories](http://developer.android.com/reference/android/os/Environment.html#lfields)
- [`FileProvider`](http://developer.android.com/reference/android/support/v4/content/FileProvider.html)
- [Decoding Bitmaps](http://developer.android.com/training/displaying-bitmaps/index.html)
- [Picasso](http://square.github.io/picasso/)
