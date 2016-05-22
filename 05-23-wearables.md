## Wearables

### Admin
Any general questions/issues with projects?

Alpha check-ins tomorrow! I sent out a sign-up sheet over the weekend; if you _cannot_ make a time, let me know and we'll work something out

Goals for the checking:
1. Stubs for each of your Activities/Fragments (so have "outline")
2. Proof-of-concept for the mobile component (e.g., location, sensor, messaging, etc)
3. Progress towards multiple configurations
4. Contributions (e.g., commits) from everyone on the team!

- Scheduling question: my original plan for _Wednesday_ was to talk about Virtual Reality a little bit, maybe demo with some of the Cardboard units the school has. But it would really more of an "FYI" without a lot of development skills attached (because "Hello World" is pretty hugely complicated and requires a solid understanding of Computer Graphics if you want to build it in Java)
  - So the question is: would people still like to take a look at this (for exposure/interest), or would you rather have the day to work on your projects (since I know we basically have 2 weeks for development)?


### Wearable Intro
Today we're going to talk a little bit about building apps for [Android Wear](https://www.android.com/wear/). That is, watches!
- Why? Not because I think that wearables are the "next big thing" (I don't, but then I'm not a futurist). But because it is a testable example of how the Android platform can be used across lots of different form-factors (TVs, cars, refrigerators, etc). I personally don't think smart-watches are all that great. But I do think embedded systems are a pretty cool, and I expect that having thought about something like the wearable API will be informative for those systems as they appear.
- Note that I don't have a watch, and have only played with the SDK a little bit, so today will be a tad "experimental" (but that's okay!)

#### Setup
We need to get [setup](http://developer.android.com/training/wearables/apps/creating.html) First we need to make sure the Wear SDK is installed
- Update SDK tools (`Tools > Android > SDK Manager`)
  - Install wearable SDK (or update to `4.4W.2`) [under SDK Platforms]
  - Android SDK tools need to be at 23.0.0+ [under SDK Tools] (should be so you have the latest emulator)

We can then start creating a project (from scratch!):
- Note that it's extremely common to have Wearable apps be partnered with a "traditional" phone app; for example, with the phone app running background services that the Wearable can use. This allows the wearable to offload a lot of the "heavy lifting" (things like network connections, etc) to the phone, acting a a kind of "thin client."
  - So you would create _two_ "apps" for your project... but we'll just do the Wear app because we're only browsing today.
- Give it a `Blank` Activity (which will still give us some starter code we can look at).

The next thing we need to do is set up a wearable device! If you have a smart-watch awesome; but the rest of us need to use the emulator. Go to `Tools > Android > AVD Manager` and create a new device.
- Pick a wear model (I'm going with circle cause interface difference, and useful for showing how UI changes dramatically). Run Marshmallow so we have latest features.
- Then run the emulator!
  - Slide through the welcome screens

Note that in order to have full features on your Wear device, you'd want to [pair the emulator with an actual device](http://developer.android.com/training/wearables/apps/creating.html#SetupEmulator)
  - To do this, you will need to install the `Android Wear` app from Google Play, plug in your phone over USB, and then forward the communication from the emulator via
  ```
  adb -d forward tcp:5601 tcp:5601
  ```
  - Start up Android Wear app, and click "Connect Emulator" in the settings menu. If you want to play around with what a watch might be like

Once we have the device (virtual or otherwise) in place, we can launch our starter app, and voila!

### Screen Shape
To begin looking at the code, let's start by talking about [Wearable layouts](https://developer.android.com/training/wearables/ui/layouts.html). Remember how when we created the emulator we had to pick square or round (or chin)? Watches can have drastically different screen shapes (unlike phones, which while the resolution/sizes may be different are still basically rectangles). So our UI work has to accommodate that.
- What happens if we try to just apply a square layout to a round screen? [this](http://developer.android.com/wear/images/01_uilib.png)
  - Can test it in the layout Preview window; select the device model from the dropdown at the top


How might we handle these different layouts? Wear gives us two options. First, we can specify different layout resources for `rect` and `round` screens, and then simply inflate the appropriate one at runtime.
- Used when we want square/round interfaces to be significantly different

This is the ___equivalent___ to defining different portrait and landscape (`land`) resources. But rather than specifying a different configuration folder, we're instead going to use a [`WatchViewStub`](http://developer.android.com/reference/android/support/wearable/view/WatchViewStub.html).
- Does everyone remember the `ViewStub` View you (should have) used in the Sunspotter assignment? This is the same kind of thing--a "placeholder" for a view that will be inflated into the layout.
- In particular, the `WatchViewStub` specifies two attributes
  ```xml
  <android.support.wearable.view.WatchViewStub
      app:rectLayout="@layout/rect_activity_wear"
      app:roundLayout="@layout/round_activity_wear">
  </android.support.wearable.view.WatchViewStub>
  ```
  Which then specify which view should get inflated depending on the screen configuration.
- (The provided code also does some work to lookup the view inside those resources only _after_ they've been loaded. You can actually use the same pattern in the regular SDK!)

The second solution I like a little more, because it's overall less work. This is useful if we want the layouts to be the same, but we want everything to automatically fit inside the round screen and not get cut off. Effectively, we want to lay things out only in the **inscribed rectangle**.
- This is equivalent to deciding that a round screen is just a square screen with lots of border space, so we'll just use the square portion and treat them as having different resolutions just like we have in the past.
- If we were trying to do this work ourselves, how would we make it so that the content was only inside the inscribed rectangle? _Padding!_
  - Of course, this padding would need to be different depending on the configuration...

Luckily, Android provides [shape-aware layouts](https://developer.android.com/training/wearables/ui/layouts.html#same-layout) that give us this padding automatically. This is a class called [`BoxInsetLayout`](http://developer.android.com/reference/android/support/wearable/view/BoxInsetLayout.html). It is a `ViewGroup` (it extends `FrameLayout`), and so can act as a "container" for our content, and will apply padding to the contained elements to make them fit on round screens if needed.
- We have the "nested" elements automatically assume a (**minimum**) padding necessary for the inset box by using the `app:layout_box` property. The easiest way to make sure the content fits is to create _another_ nested View Group (like a `FrameLayout`), give that the requisite padding, and put content inside of it:
```xml
<android.support.wearable.view.BoxInsetLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_height="match_parent"
    android:layout_width="match_parent">
    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_box="all"> <!-- inset margins all around -->
        <TextView
            android:layout_height="wrap_content"
            android:layout_width="match_parent"
            android:gravity="center"
            android:text="Hello World"
            android:textColor="@color/white" />
    </FrameLayout>
</android.support.wearable.view.BoxInsetLayout>
```
- The docs suggest to use `layout_gravity` to position elements, we can use `|` to combine values together (e.g., `center|right`).
- Note the slick part of this is that the `BoxInsetLayout` fills the screen, but the `FrameLayout` content does not! If we assign colored backgrounds, we can see how this works (and how it is modified by the padding we specify)


//cards (fragment review)
  //not look like much... but we can also do paging!

//can also include a voice activation


### Cards
There are a number of different wear-specific [ui elements](http://developer.android.com/training/wearables/apps/layouts.html#UiLibrary) (i.e., Views) we can use, and I'd encourage you to look through those options. We're only going to demo a few of them today (for time).

One of the most common UI elements specific to wearables is [Cards](http://developer.android.com/training/wearables/ui/cards.html), which are basically chunks of content that usually appear along the bottom of the watch (a common UI element). Luckily, cards are pretty easy to create...

We're going to use the [`CardFragment`](https://developer.android.com/reference/android/support/wearable/view/CardFragment.html) class. This is used identically to every other Fragment we've ever created. We instantiate the `Fragment` object, and then use a `FragmentManager` to create a transaction that will add the Fragment to a container of some kind.

- First we'll want to make sure our `FrameLayout` (inside our `BoxInsetLayout`) has an id so we can inflate the fragment into it.
- We can then create the `Fragment` in code. While we could probably subclass `CardFragment`, it's easier to just use a [factory](http://developer.android.com/reference/android/support/wearable/view/CardFragment.html) method to produce a generic card for us (remember, this allows us to not have to instantiate new fragments?)
- Finally, we can use a `FragmentManager` (just like before) to put our fragment into the layout:
```java
CardFragment cardFragment =
    CardFragment.create("A card", "This is my card"); //no icon

FragmentManager fragmentManager = getFragmentManager();
FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
fragmentTransaction.add(R.id.container, cardFragment);
fragmentTransaction.commit();
```

Note that We can define the Fragment inside the `xml` resource [directly](http://developer.android.com/training/wearables/ui/cards.html#card-layout) by using a couple other utility Views. I'll leave that as an exercise to the reader.

Yeah, this isn't super impressive or exciting. But it does give us a starting point to doing some custom layout work (e.g., it gives us a bit of a border).

### ViewPager
However, Cards do give us some simple Fragments to work with, which we can use to create multiple "screens" of our app. In particular, we can set it up so that rather than needing to tap on a button to change what Fragment is shown, we can just "swipe" to one side to page through the different Views.
- We do this using a class called a [`ViewPager`](http://developer.android.com/training/animation/screen-slide.html). We can actually use this class on mobile as well (and there is a tutorial for making it work; in fact, you may have used it already?). But we'll play around with it on Wear because it is a closer fit to the kinds of interactions you'll have with a watch.
- Note that besides the documentation, you could also check out one of the many [samples](http://developer.android.com/samples/index.html) (in a painful to navigate list) for an example. E.g., the Wearable [Flashlight](http://developer.android.com/samples/Flashlight/project.html) demo uses a `ViewPager`

Overall, the thing to remember about a `ViewPager` is that it follows a similar architecture to a `ListView` (which we've worked with extensively). Just instead of scrolling through item Views, we're going to swipe ("page") through fragment Views.

Just like with `ListView`, we'll start by defining the "Pager" container in the xml:
  ```xml
  <android.support.v4.view.ViewPager
    android:id="@+id/pager"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
  </android.support.v4.view.ViewPager>
  ```
- We can then fetch access to this in the `MainActivity` using `findViewById()` (as usual).

We're then going to need an **adapter**; in particular, we'll want to use a [`FragmentPagerAdapter`](https://developer.android.com/reference/android/support/v13/app/FragmentPagerAdapter.html), which specifically lets us page through fragments (that is: it _adapts_/maps between the current "page" and which Fragment to show).
- **IMPORTANT**: in order to make this work correctly, we need to use the `FragmentPagerAdapter` in the **`v13` support library** (not `v4`). We can add this requirement to our gradle module by simply changing the imported support library to `v13`. Then when we reference the class, just confirm we're getting the right version!
- `FragmentPagerAdapter` is generic, so we'll need to subclass it (gee, that again?)
  - It's pretty simple to fill in: we just need `getItem()` to return which Fragment to show, and `getCount()` to return how many there are.
  - The easiest way to handle this is to have the adapter actually keep an `ArrayList<Fragment>` variable, which we can then query.
  - Throw in an `addFragment()` method and we're gold (remember to call `notifyDataSetChanged()` so that the View updates!)

    ```java
    public class MyPagerAdapter extends FragmentPagerAdapter {
        private ArrayList<Fragment> fragments;

        public MyPagerAdapter(FragmentManager fm){
            super(fm);
            fragments = new ArrayList<Fragment>();
        }

        public Fragment getItem(int position) {
            return fragments.get(position);
        }

        public int getCount() {
            return fragments.size();
        }

        public void addFragment(Fragment fragment){
            fragments.add(fragment);
            // Update the pager when adding a fragment.
            notifyDataSetChanged();
        }
    }
    ```
- Finally, we can instantiate our Fragments (maybe an extra card or two) and add them to the Adapter!
  - And voila; we can swipe to page

Note that Wearable also includes a [`GridViewPager`](http://developer.android.com/reference/android/support/wearable/view/GridViewPager.html) which works the same way, but allows both horizontal _and_ vertical scrolling! This is a bit more complicated because it goes in two dimensions, but same basic principle.
- [[If we want to fight through it as a demo, we can]]


### Interaction: Speech
So we've done some interesting layout work, which is similar to what you've done in mobile (and hopefully is a useful review/extension). But interacting with a watch is more tricky: you don't have a keyboard, can only tap and swipe (and only have one hand!)... how do you do anything more complex _effectively_?
- One neat solution is to ignore your hands entirely and just use your mouth... by which I clearly mean [voice commands](http://developer.android.com/training/wearables/apps/voice.html). These are actually pretty easy to integrate into a wearable app (and phone apps as well).

Wearable includes a list of [built-in voice command](https://developer.android.com/training/wearables/apps/voice.html)---basically system-detected audio that launches an `Intent` to handle the command. Thus we can let our app support these commands simply by registering an `IntentFilter` for the intent!
- I would demo this... except that the emulator has these voice commands disabled (if anyone actually has a watch that they're building on, we could watch them demo :p)
- And again, mobile has [system-level commands](https://developers.google.com/voice-actions/system/) as well.

However, what does inexplicably work (on my machine at least) is getting free-form voice input! This can be done simply by launching an `Intent` for a separate activity (e.g., one provided on the wearable or on your phone) that responds to the **`RECOGNIZE_SPEECH`** action:
```java
private void displaySpeechRecognizer() {
  Intent intent = new Intent(RecognizerIntent.ACTION_RECOGNIZE_SPEECH);
  intent.putExtra(RecognizerIntent.EXTRA_LANGUAGE_MODEL,
                  RecognizerIntent.LANGUAGE_MODEL_FREE_FORM);
  startActivityForResult(intent, 0);
}
```
- We can launch this intent at the click of a button... maybe inside an additional Fragment?
  ```java
  public static class SpeakFragment extends Fragment {
      public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
          return inflater.inflate(R.layout.fragment_button, container, false);
      }
  }
  ```
  - (make an `ImageButton` in the layout resource)

When we get the result back (because we started the `Intent` for one), we can do further processing on what the user said:
```java
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == SPEECH_REQUEST_CODE && resultCode == RESULT_OK) {
        List<String> results = data.getStringArrayListExtra(
                RecognizerIntent.EXTRA_RESULTS);
        String spokenText = results.get(0);
        // Do something with spokenText. E.g., log out.
    }
    super.onActivityResult(requestCode, resultCode, data);
}
```
- For example, we could make a new `CardFragment` containing our note :)
  - And then jump to there: `pager.setCurrentItem(adapter.getCount()-1, true);`

This is pretty easy, and not a bad way to support voice operations in your app!


### Watch Faces
As one last example (if time): a big part of watches is what's on the _face_ of the watch. How do you show the time? Or how do you show other information (like current speed or your heart-rate if you're running).

Android Wear let's you develop your own [Watch Faces](http://developer.android.com/training/wearables/watch-faces/index.html), which are basically `Services` that are run in the background to update a displayed View... that View being the watch's face.


Last set of samples to look at (if time): a big part of watches is what's on the face: e.g., how do you show the time? Or maybe you want to be showing other information (like current speed or heartrate if you're running). Android Wear lets you develop your own [Watch Faces](http://developer.android.com/training/wearables/watch-faces/index.html), which are basically `Services` that are run in the background to update a displayed View... which is the watch's face.

Implementing this can be fairly complex, so for time we might just look at and/or run the [WatchFace sample](http://developer.android.com/samples/WatchFace/project.html).
  - We extend the `CanvasWatchFaceService` and `Engine` classes, and fill in the methods that specify what happens on each "tick" of the display. We then need to do a bunch of Canvas-style drawing to specify what the watch-face looks like (using a `SurfaceHolder` in fact!)
  - [Can explore as time allows]


### Wrap-up
This was intended to be a fun "first look" at wearables. There are many more components, but I wanted to give you a taste of what is involved (and so you could see how the same framework concepts allow for implementation on a significantly different form factor)!
  - Other fun sample to play with on watch: [Eliza](http://developer.android.com/samples/ElizaChat/project.html)
  - Wear also has a big "Data Layer Connection" to allow synchronization between mobile and wearable; check out the [Quiz sample](http://developer.android.com/samples/Quiz/project.html) as an illustration.


#### Lecture References
- [Wearable layouts](https://developer.android.com/training/wearables/ui/layouts.html)
- [`BoxInsetLayout`](http://developer.android.com/reference/android/support/wearable/view/BoxInsetLayout.html)
- [`CardFragment`](https://developer.android.com/reference/android/support/wearable/view/CardFragment.html)
- [`ViewPager`](http://developer.android.com/training/animation/screen-slide.html)
- [`FragmentPagerAdapter`](https://developer.android.com/reference/android/support/v13/app/FragmentPagerAdapter.html)
- [`GridViewPager`](http://developer.android.com/reference/android/support/wearable/view/GridViewPager.html)
- [built-in voice command](https://developer.android.com/training/wearables/apps/voice.html)
- [WatchFace sample](http://developer.android.com/samples/WatchFace/project.html)
