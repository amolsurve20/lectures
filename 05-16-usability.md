## Usability & Accessibility

### Admin [10-15min]
Homework: get turned in okay?
- We'll get scores back to you as quickly as we can. We fell behind because of Todoer (just like you) so working to catch up.

Project check-in: any questions/issues? Sounds like a good set of apps--I'm looking forward to them.
- The next deliverable is due **Wednesday**, and is a **high-fidelity prototype** (e.g., a "fake app"). The intention here is to be able to design and demonstrate _how your app is used and what it does_ in order to get all of the interface-design stuff out of the way before you dig into implementation.
  - For full transparency: some previous projects in my courses did not meet expectations for _basic usability_, and so I want to make sure there is an explicit step where you consider and get feedback on whether your app is usable or not
  - If I can't figure out how to use it, then it doesn't matter what it does under the hood--so we need to get that together!

These prototypes will be presented **in class** on Wednesday. We're going to load them up on a computer hooked up to the projector, so make sure they are online and accessible (ideally _before_ the deadline at start-of-class).

Logistics are as follows:
- A (randomly-chosen) "volunteer" will come up and act as a "user" for the app.
  - You can give the user a 1-sentence summary of the app's purpose (for context), but otherwise you do **not** get to give the user any guidance for how to use it. Usage should be self-evident.
- The user will "use" the app, trying the various "features" in the prototype. They'll do this for 3-5 minutes.
  - The user will [think-aloud](https://en.wikipedia.org/wiki/Think_aloud_protocol) to explain their thought process for how they're trying to achieve their goals or overall use the application.
- (The group needs to awkwardly stay silent--just listen to what someone does when they use your program, and think about how to make it easier to use to avoid any problems they encounter).
- Afterwards, we'll have the balance of 10 minutes per project to get feedback from the rest of the class
  - What do you like / what looks good; what seems weird / needs improvement.
- I'll make a survey available on Catalyst so you can give any additional written feedback as well if we run out of time
  - The goal is to get feedback and figure out how to improve the design before you start implementing!!

We'll do this for all 9 groups, plus a few minutes to switch between. We might even get out early!
- Are you familiar with this kind of user testing? If not, this is a good low-pressure way to try it out :)

Questions?

### Usability Principles [20min]
Last week we looked at some of the "back-end" stuff (services, databased). But this week we're swinging hard back to to the "front-end" parts of an Android app and consider overall **User Interface** design.

I want to take a little bit to talk about some overall usability principles that apply not only to mobile apps, but really to any interactive system. We'll then narrow in on some Android specific issues.
- This might be review from other courses, and if so please chime in with other examples/ideas.
- This is my take, and I self-identify as a developer rather than a designer, so take that as you will. But I did spend my grad school years studying this stuff so...

When we're developing a system's **user interface**, what we're really interested in is creating a system with a "good" interface. So the question becomes: how do we tell if an interface is any good?
- There may be a "right" and a "wrong" way to design an interface (or rather, "more right" and "less right" ways).
- This is an almost philosophical question, but that decision is almost always based on **experience** ("this was good in the past").

Lots of people over years, decades, and centuries have gotten a lot of experience figuring out what works and what doesn't. So in order to communicate what are "good" and "bad" user interfaces, designers have tried to _encapsulate_ their experiences them as **design principles**: loose rules that describe what you should do when designing interfaces.
- These are "less rules, more of guidelines". You can also think of them as heuristics.

There are lots of sets of _interface design principles_ out there.

One early example: the design principles for the [Xerox STAR (1981)](https://en.wikipedia.org/wiki/Xerox_Star), an early office machine (think: computer just for MS Office). It was one of the first computers to come with a mouse!
- The designers _iterated_ their design work repeatedly before actually building the machine and came up [a set of principles](http://www.guidebookgallery.org/articles/designingthestaruserinterface), including:
  1. Familiar User's Conceptual Model
    - e.g., pick a good metaphor. They used the office/desktop metaphor, and was the first commercial system to do so!
  2. Universal Commands
    - e.g., a generic "move" command across applications
  3. Consistency
    - e.g., don't change in the middle (internal consistency) and don't mess with what people expect (external consistency)
  4. Simplicity
    - e.g., Alan Kay's maxim: "simple things should be simple; complex things should be possible."
  5. User Tailorability
    - e.g., letthe users customize their desktop, set their own defaults, etc.

_Think any of these might apply 35 years later to an Android app?_

As another set of principles I'd like to consider in a little more detail come from _Rogers, Sharp, and Preece_'s book [Interaction Design](http://www.id-book.com/):

- **Visibility**
  - Functions and controls that are more visible are easier to use
  - This is the complaint against the [hamburger menu](http://deep.design/the-hamburger-menu/)
- **Feedback**
  - Let the user know what is happening, and do it quickly!
    - c.f. [living with lag](https://www.youtube.com/watch?v=_fNp37zFn9Q)
  - But it also means to let the user know if something works or doesn't work
    - e.g., if I click a "send message" button, does anything happen?
- **Constraints**
  - It's okay to restrict what the user can do to keep them from making mistakes
  - Common format: graying out (disabling) buttons that aren't applicable
- **Consistency**
  - This again! It's close to the _main_ rule in good UI design
  - It supports learnability/memorability--easy to learn and use!
- **Affordance**
  - The "actionable" properties of an object, person, or environment.
    - "the ball affords throwing"; "the chair affords sitting"
    - "When all you have is a hammer, everything looks like a nail"
  - (Originally conceived by Gibson, developed by Donald Norman as _perceived affordance_)
  - Example: a button.
  - _Controls should look the way they work, and work the way they look!_

### Universal Usability
One last set: in the project write-up, I linked to [Shneiderman and Plaisant's Golden Rules](https://www.cs.umd.edu/users/ben/goldenrules.html). this is a really nice list to work from; in particular the first three items
- Fun fact: Plaisant's research (like what lead to this list) was used to help [invalidate Apple's "slide-to-unlock" patent](http://arstechnica.com/tech-policy/2016/02/appeals-court-reverses-apple-v-samsung-ii-strips-away-apples-120m-jury-verdict/). So remember that if a professor ever has you read old design research articles; they could be work $120 million.

**Consistency** and **Feedback** again as \#1 and \#3, but what is \#2? **Universal Usability**. What is that?

**Universal Usability** (also known as [Universal Design](https://en.wikipedia.org/wiki/Universal_design)) is the goal of designing products that are _inherently accessible_. The premise here is that designing for _accessibility_---being usable by all people no matter their ability (physical or otherwise)---benefits not just those with some form of limitation or disability, but **everyone**.

- The classic example is [curb cuts](https://en.wikipedia.org/wiki/Curb_cut): the "slopes" built into curbs to accommodate those in wheelchairs. However, these cuts end up making life better for _everyone_: people with rollerbags, strollers, temporary injuries, etc.

- Or the idea that if you design for someone with one arm, then you support people with disabilities, but also people with temporary disability (e.g., an arm in a sling/cast) and people who are just massively inconvenienced (e.g., a person holding a baby in that arm).

- This is just as important in the mobile design space: if we can support people with vision impairment (e.g., by supporting touch or voice controls), then we also support people who want to use the app while driving or otherwise visually occupied.

- Or in the mobile space we want to support people who have low-end devices with limited data allowance (such as by supporting older versions of Android, or being frugal with our data downloading via a Service, etc). The idea is that this will not only help people or communities that can't afford unlimited 4G connections, but it also supports people who are currently without data connections (in the woods, on an airplane, over their data plan, etc).

- Also see also a [recent article](http://www.fastcodesign.com/3054927/the-big-idea/microsofts-inspiring-bet-on-a-radical-new-type-of-design-thinking) about how Microsoft is approaching this problem.

The main point: accessibility is important and worth considering. Not that people with disabilities aren't worth helping for their own right, but a little bit of effort to support those who are normally disenfranchised makes the world a better place for everyone.
- And for this reason, we're going to talk about supporting accessibility a bunch more today.

### Dynamic Android Resources
So my point is that we want to make sure our app (or really, any system/experience you design) supports as many people as possible, regardless of their ability or background (including things like language, culture, or nationality).
- We can handle a lot of this in Android by specifying user interfaces in the **resources**. These can by dynamically adjusted based on the _system configuration_

For example, we've been specifying various element sizes in `dp` (density-independent pixels), and font-sizes in `sp` (size-independent pixels). Why? Because this way if the user needs to adjust their phone's font-size (e.g., to support visible disabilities) our app accomodates
- Examples: using `sp` (taken from [Ryder Ziola](https://www.linkedin.com/in/ryder-ziola-9b93b34))

We also want to make sure that our user interfaces are designed to support **internationalization** (e.g., different languages)
- Why focus only on US or Western audiences? What makes them special?
- Examples: pseudo-localization
- Examples: supporting Right-To-Left (RTL) languages

So this doesn't really _change_ what we've done, but are things that can be considered in the implementation of our user interfaces.

Questions/thoughts?

### Break!
Break, then we'll do a bit more hands-on coding stuff towards supporting accessibility.

## Accessibility
Fork & clone repo for today. Our starting app should look familiar--it was the "Intent" demo from a few weeks back!
- Starting out with it completed, and we'll go from there.

So let's talk about supporting **Accessibility**---specifically, supporting users with some kind of disability.
- Note that this isn't as well tested as it should be (I ran out of time this weekend), but we'll make it work
- E.g., no `completed` branch yet. I hate that this is the lecture that finally gets short shrift...

[Accessibility in Android](http://developer.android.com/guide/topics/ui/accessibility/index.html) is not as effective as it could be, but there are a number of simple steps you can take to support users with disabilities _especially_ if you're just using the built-in `View` elements (like we've been doing in this class). So we're going to go over some of the "low-hanging fruit" today.
- There are basically two goals supported by the framework itself:
  1. Enable the application to be used with "sighted assistance" (e.g., for visually-impaired)
  2. Support interaction without the touch-screen (e.g., with directional controls for other external devices)

To test all this stuff out, we're actually going to want a _new_ emulator hardware profile
- Clone the Nexus 5, enable Keyboard and D-Pad support. This will let us control the emulator/device with alternative devices, such as those used for accessibility support (will come back to this in a moment, and makes testing a bit more sane)


### TalkBack
These capabilities are provided by [accessibility services](http://developer.android.com/guide/topics/ui/accessibility/services.html), or **background services** (like what we did last week) that are able to respond to "accessibility events" (more later). The two most common built-in services are [TalkBack](http://developer.android.com/tools/testing/testing_accessibility.html#testing-talkback), which "speaks" the name of UI elements as the user focuses on them, and [Explore by Touch](http://developer.android.com/training/accessibility/testing.html#testing-ebt) (4.0+), which allows the user to drag a finger around a screen and get verbal feedback of what is there.
- These can be turned on by going to `Settings > Accessibility > TalkBack`... but we need to install it on the emulator (`adb install package.apk` will work if `adb` is available)

[[Demo: TalkBack]]

A lot of interface design gives usability hints (e.g., what a button does) through visual cues: icons, labels, etc. This isn't effective for visually impaired--as you can see, our "icon" buttons are just explored as "Button 59".
- We can improve this by adding an `android:contentDescription` attribute to our button, which specifies how TalkBack refers to it!
  ```
  android:contentDescription="Dial"
  ```
- Super easy addition that does a lot to support accessibility

For the second piece, we'd also like to support interaction that _doesn't_ use the Touch Screen. The way to do this is to make sure that input elements can get _focus_: that is, they can be "currently selected" supporting interaction without the touch screen.
- To do this, we need to add another attribute: `android:focusable=true`
  - We can also specify `android:nextFocusDown|Up|Left|Right=@+id/...` to adjust the "tab order" of inputs. This is a thing to do in web development as well!
- Note that in order to test this, we need to enable our device to support a physical keyboard and/or D-Pad

Finally, I want to flag that Android supports a wider variety of accessibility services&mdash;and indeed we can write our own!
- We can support existing services in our _custom_ views by firing off [Accessibility Events](http://developer.android.com/training/accessibility/accessible-app.html#events). For example,
```java
AccessibilityManager accessibilityManager =
        (AccessibilityManager) MainActivity.this.getSystemService(Context.ACCESSIBILITY_SERVICE);
if (accessibilityManager.isEnabled()) {
    View v = //... findViewById(R.id.masterLayout);
    v.sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_TEXT_CHANGED);
}
```
- More details [here](http://developer.android.com/guide/topics/ui/accessibility/apps.html#send-events). Be sure to check out information about [supporting custom touch views](http://developer.android.com/guide/topics/ui/accessibility/apps.html#custom-touch-events)

But more than that, we can also build our _own_ accessibility services! We do this by extending the `AccessibilityService` class and overriding the appropriate methods:
```java
public class MyAccessibilityService extends AccessibilityService {
    @Override
    public void onAccessibilityEvent(AccessibilityEvent event) {
    }

    @Override
    public void onInterrupt() {
    }
}
```
- We declare this as a `<service>` in the `Manifest`, with an `<intent-filter>` for `<action android:name="android.accessibilityservice.AccessibilityService" />` so that it can respond to the events generated (events being when something happened that is relevant to accessibility services... it's a bit tautological).
  - We also need to give it `android.permission.BIND_ACCESSIBILITY_SERVICE` permissions, both as a `<uses-permission>` and as a `android:permission` attribute
- Will need to configure the service, per [these instructions](http://developer.android.com/training/accessibility/service.html#configure). Easiest way is to do this in XML (provided in starter code), and then include in manifest as a meta attribute:
```xml
<service android:name=".AService"
   android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE"
   >
   <intent-filter>
       <action android:name="android.accessibilityservice.AccessibilityService" />
   </intent-filter>

   <meta-data android:name="android.accessibilityservice"
       android:resource="@xml/accessibility_service_config" />
</service>
```


- And then finally we can have the `onAccessibilityEvent()` method do something helpful when the event occurs!

For example, we can provide spoken feedback using the [text-to-speech engine](http://android-developers.blogspot.com/2009/09/introduction-to-text-to-speech-in.html) (untested):
```java
private TextToSpeech mTts;

mTts = new TextToSpeech(this, this);
mTts.setLanguage(Locale.US); //optional?
String myText1 = "Did you sleep well?";
String myText2 = "I hope so, because it's time to wake up.";
mTts.speak(myText1, TextToSpeech.QUEUE_FLUSH, null);
mTts.speak(myText2, TextToSpeech.QUEUE_ADD, null);

mTts.shutdown(); //in onDestroy
```

Can check for TTS capabilities if needed:
```java
Intent checkIntent = new Intent();
checkIntent.setAction(TextToSpeech.Engine.ACTION_CHECK_TTS_DATA);
startActivityForResult(checkIntent, MY_DATA_CHECK_CODE);

protected void onActivityResult(
        int requestCode, int resultCode, Intent data) {
    if (requestCode == MY_DATA_CHECK_CODE) {
        if (resultCode == TextToSpeech.Engine.CHECK_VOICE_DATA_PASS) {
            // success, create the TTS instance
        } else {
            // missing data, install it
            Intent installIntent = new Intent();
            installIntent.setAction(
                TextToSpeech.Engine.ACTION_INSTALL_TTS_DATA);
            startActivity(installIntent);
        }
    }
}
```

It's also worth noting that Android has a [Vibrator](http://developer.android.com/reference/android/os/Vibrator.html) class that can be used to vibrate the phone.
  - e.g., we could vibrate in morse code?

#### Checklist
This is just a taste of how we might support accessibility: the `contentDescriptions` and `focusable` are the easy to support ones.

Android also provides a general [checklist][accessibility checklist] of things you should check for as you're developing/accessorizing your app, including links to design guidelines
- check this out! A lot of it may not be relevant when using the standard widgets, but as soon as you make your own custom stuff you'll want to start including this to support as many different users/situations as possible.

Any questions?


<!-- references -->
### Lecture References
- [Accessibility in Android](http://developer.android.com/guide/topics/ui/accessibility/index.html)
- [Configure Accessibility Service](http://developer.android.com/training/accessibility/service.html#configure)
- [text-to-speech engine](http://android-developers.blogspot.com/2009/09/introduction-to-text-to-speech-in.html)
- [Vibrator](http://developer.android.com/reference/android/os/Vibrator.html)
- [accessibility checklist](http://developer.android.com/guide/topics/ui/accessibility/checklist.html)
