# 03-28 Introduction

### Welcome
Good Morning! This is *498 B: Android Development*

Plan for the day:
  - Do introductions & syllabus
  - Android History
  - (break)
  - Getting Started with Android: "Hello World"
    - Tomorrow we'll do Java review, then swing back into Android on Wednesday

### Introductions [10min]
**Who am I?**
- "Hi, I'm Joel". Senior Lecturer
- Grad school at UCI, studied _ubiquitous computing_ and _pervasive games_ I did a chunk of Android development (aimed at sensor sharing and tracking), but also studied a lot of HCI-style theory stuff in the mobile space
  - I'm interested in experiences that are __specific__ to mobile devices: not just "websites on a phone". So we'll be emphasizing a lot of the mobile-specific features in this course.
  - I started around Android 2.0 Eclair (API 5). Things were less organized then.
- Also about me: I live in Tacoma, so have a commute. If I'm not responsive or in the office, probably on bus or train. And I rush home at end of the day to see my daughter (who is 2 and adorable)
  - Also I'm rushing here from MGH and a previous class, so apologize if late!

**Who are you?**
- ("fun facts" from Canvas)
- But also introduce yourself to each other! Make a new friend
  - Who are you? Where are you from?
  - Most excited about for this class?

### Syllabus [20min]
Course materials are on Canvas or GitHub. Syllabus is on Canvas.

Contact info is at the top
  - Office hours in middle of the day M-W (trying to keep Thur free), but in my office often. Feel free to send me an email and make an appointment for another time!
  - As part of "first steps", join Slack! Slack is basically a really slick chat room system with killer search, cross-device syncing, and sweet integration with other tools. It's very much the new hotness in the tech industry, so if you've never tried it, now is your chance
    - Really effective way of asking questions! Often easier than email, and you can get answers from the entire class!

We have no textbook, since Googles documentation and tutorials are online and quite thorough.
  - The relevant sections of the guides are listed on the calendar. I recommend you skim through the "readings" before class to get a sense of what we'll be doing (it will help with moving quickly).

Course Calendar
  - Looking at the calendar, we have a pretty "ambitious" schedule
  - Cover overall basics of Android development in first 4 weeks. Then focus on mobile specific capabilities for a few weeks, before considering aspects of user experience as we wrap up.
  - Will have weekly programming assignments (app-a-week): mostly due early Wednesday **mornings**, which you should read as _"Tuesday night when you go to bed."_
    - First assignment is due week from Tuesday though
    - Get started on assignments early!
  - Last 3 weeks won't have homeworks, but will do a final group project, which will be an app of your own design. We'll talk more about that as we get closer to it.
    - This will still have some intermediate deliverables, so you will be continuing to work hard.
  - **[[Questions?]]**

Assignment **Details** are on GitHub, written into the README of each repo.
  - In order to get access to this stuff, you'll need to (a) sign up for GitHub and (b) let your TA know your GitHub username so we can add you to the org. You should do that ASAP (like during break or immediately after class).

Also on GitHub: lecture notes! These will be uploaded every day. Include all the links to various classes and concepts we'll use. Another reference to check (though will often mirror the Android guide textbook).

Policies are on linked wiki page. Things to highlight:
  - Participation: let's try and keep this interesting! Come to class ready to code along with examples; ask lots of questions; volunteer ideas; etc. Take responsibility for your own learning!
  - To help accommodate schedules, you each have two "late days" you can spend on assignments. Think of them as a "extensions with no questions asked". To use one, submit the assignment and include a comment announcing that you're using a day.
    - Intended to be used for those weeks when stuff just piles up
  - Collaboration: this is Informatics; it's okay to talk to other people and get help! The goal is to get things working, and to learn from whatever resources you can.
    - However, the assignments you turn in should be your own--Gilligan's Island rule is good for making sure you don't accidentally copy off someone else.
    - If you find/adapt examples from the internet, be sure and cite your source! Include a comment with where you got that from, or note it in your submission
    - This is the Academic Integrity issue, which I assume everyone is aware of and familiar with by this stage in your careers?

- **_Questions on administrative/logistic stuff?_**


### Android History [20min]
Start off with a question: _What is Android?_ Why did you sign up for a course called Android Development?
  - It's an Operating System! (software that connects hardware to software and provides general services)
    - Mobile specific!
  - "Android" also refers to "platform" (e.g., devices that use the OS) and the ecosystem that surrounds it.
    - This includes the device manufacturers who use the platform, and the applications that can be built and run on this platform.
History
  - 2003: founded by "Android Inc" to build mobile OS operating system (similar to what Symbian (Nokia) was doing)
  - 2005: acquired by Google, who was looking to get into mobile
  - 2007: announces [Open Handset Alliance](http://www.openhandsetalliance.com/), group of tech companies working together to develop "open standards"
    - Google; HTC, Samsung, Sony; T-Mobile, Sprint NTT DoCoMo; Broadcom, Nvidia, etc. Now 84 companies.
    - Note this is the same year the first iPhone came out!
  - 2008: First Android device (HTC Dream / T-Mobile G1)
    - 528Mhz ARM chip; 256MB memory; 320x480 resolution capacitive touch; slide-out keyboard!
    - fun little device!
  - 2010: First Nexus device: Google-developed "flagship" devices
    - Nexus One: 1Gz Scorpion; 512MB memory; .37" 480x800 AMOLED capacitive touch
    - Comparison: iPhone 6S+ (1.85Ghz dual core ARMv8, 2GB RAM, 5.5" @ 1920x1080)
  - 2014: Android One (low-cost for developing countries)
  - 2015: Project Brillo, aiming for embedded devices

Android is incredibly popular! (e.g., http://www.businessinsider.com/iphone-v-android-market-share-2014-5)
  - There are some questions about what is counted... but what we care about is that there are _a lot_ of Android devices out there!
    - And these are a lot of **different** devices!

Version History
  - [Wiki chart](https://en.wikipedia.org/wiki/Android_version_history)
    - Different "versions" are named after desserts. But from developer perspective, care about API version (e.g., what set of _interfaces_ are available).
    - Most recent version (_N_) developer preview released this month
  - Interactive version: http://www.android.com/history/
  - Usage breakdown at: http://developer.android.com/about/dashboards/index.html

Upgrading
  - Updates to devices historically through purchasing new devices (every 18m on average in US)
  - Otherwise updates--including security updates--come through carriers. This is a problem from a consumer perspective!
    - But Google working to change that; moving some services out of OS into separate "App" (Google Play Services).
  - Android OS is "open source"; latest version at https://source.android.com/. Worth actually digging around in this sometime

Legal Battles
  - Would be remiss if didn't mention some of the legal issues surrounding Android
  - Biggest is **Oracle v Google**: basics is that Oracle says the _Java API_ is copyright, so because Google uses that API is Android, Google is violating the copyright
    - Claim: the method signatures themselves (and how they work) are protected!
    - CA court decided for Google in 2012 (can't copyright an API!); reversed by Federal Circuit in 2014. Supreme Court refused to hear in 2015.
    - https://www.eff.org/cases/oracle-v-google for more
    - Just recently: Google will use OpenJDK implementation of Java in Android N, instead of their own (http://venturebeat.com/2015/12/29/google-confirms-next-android-version-wont-use-oracles-proprietary-java-apis/)
      - won't impact us now (or much at all), but may solidify differences in Android and Java SE
  - Others as well:
    - Apple v Samsung: "You infringed our intellectual property!" < now going to the Supreme Court
    - FairSearch v Google: "You're using predatory pricing!"
  - Take away: Android is a growing, evolving platform that is embedded in and affecting the social infrastructures around information technology.

**BREAK [5min] @11:30 latest**

### Android Architecture and Code [10min]
Let's consider the basic architecture of the Android platform (More details: https://source.android.com/devices/)
- Runs on a linux kernel (for interacting with memory, processor, etc)
- On top of that is the hardware abstraction layer: interface to drivers that let us access the hardware (camera, storage, wifi, etc)
  - These drivers are generally written in C
- But on top of that is the Runtime and Android Framework, which provides the Java language that we all know and love!
- (We'll be developing applications that interact with the Framework, which runs on Java).

There are two languages we'll be working with in this class:
1. **Java**. Android code (the logic, control, data stuff) is written in Java.
  - This will feel a lot like every other Java Program: you write classes, define methods, instantiate objects, and call methods on those objects
  - But we're working within a **framework**: a set of code _already exists_ to call specific methods: we'll just fill in what those methods do to run our code
    - More Angular (framework) than jQuery (library)
  - Important: This course expects you to have "journeyman"-level skills in Java (apprenticeship done, not yet master). We'll be using a number of intermediate concepts (like generics and inheritance) without much fanfare or explanation. And so you should be comfortable with that. If you're not... this class is going to be a challenge. Consider yourselves warned.
    - We will go over some of the topics that people reported being less familiar with tomorrow in lab... but that will be most our Java review. I'm assuming you're already comfortable with the language so that we can focus on the framework.
    - _By the way_: for reviewing; rather do a "work through a tutorial/explanation" or have me lecture/review topics? (e.g., do you want to drive or do you want me to?)
  - Any questions/concerns/issues about that
2. **XML**. Android interfaces and resources are specified in XML (eXtensible Markup Language). So we'll be doing a lot of XML writing.
  - XML is just like HTML, but you get to make up your own tags (though we're going to use the ones that Android made up).
  - If you've never worked with HTML... you'll pick it up fast (it's not complex).

If you think of web programming: the XML is going to contain what we would put in our HTML/CSS, and the Java will contain what would go in our JavaScript.

Building Apps: So we write code in Java & XML. How does that get run on the phone's hardware?
- Pre-Lollipop (5.0): used [Dalvik](https://en.wikipedia.org/wiki/Dalvik_(software)), a virtual machine (similar to the JVM from Java SE)
  - Java code --compile--> JVM bytecode --translated--> DVM bytecode
    - stored in DEX or ODEX files ("[optimized] Dalvik executables")
    - process is called "dexing" (so code is "dexed")
  - For CS people: Dalvik register-based architecture (not stack-based!)
  - Dalvik Includes JIT compilation to native code (like the [Java HotSpot](http://www.oracle.com/technetwork/articles/javase/index-jsp-136373.html))
    - _Why would "native code" be faster?_ Because no translation step is needed to talk to the actual hardware (the OS)
- Post-Lollipop (5.0): uses [Android Runtime (ART)](https://source.android.com/devices/tech/dalvik/)
  - compile into native code on installation! ("Ahead of Time" AOT)
  - accepts DEX bytecode for backwards compatibility
  - faster execution, but longer install time

Android applications are packaged into APK files (basically zip files)
  - Which are then "side-loaded" or cryptographically signed to be uploaded to the App Store.
  - these are the "executable" versions of your program!
- Note: application frameworks are pre-DEXed (compiled) on device; actually compiling against empty stubs!
  - But any other libraries you include are copied into the app code.

So when building an App...
  1. Generate Java source files (e.g., from resource files, which are written XML used to generate Java code)
  2. Compile Java code into JVM bytecode
  3. "dex" the JVM bytecode into Dalvik bytecode
  4. Pack in assets and graphics into an APK
  5. cryptographically sign the APK file to verify it

There are a lot of steps here, but there are tools that take care of it for us.
- We'll just write Java and XML code and run a "build" script to do all of the steps.


### Development Tools [10min]
Since we're writing code for a virtual machine anyway, we can build Android apps on any computer OS (unlike some other mobile OS)
- Physical devices are the best--you'll need USB cable to be able to wire your device into your computer.
  - Any device should be fine; don't need cell service (just WiFi mostly)
  - If you don't normally use an Android phone, play around with it a bit to get use to the interaction language. E.g., how to click/swipe/drag/long-click things.
  - Turn on [developer options](http://developer.android.com/tools/device.html)!
- Also can use the Emulator (a "virtual" android device)
  - Represents a generic device with hardware you can specify... but has some limitations!
  - But the emulator not great on Windows; recommend you use Mac or a physical device
  - Make sure to use the HAXM (Intel acceleration manager!)
  - I'll be doing this for lecture demos (will keep me running slower than you!)

##### Software
- Java 7 **SDK**
- [Gradle](https://gradle.org/) or [Apache ANT](http://ant.apache.org/)
  - These are automated build tools--in effect, they let you specify a command that will do a bunch of steps (e.g., compilation, moving, etc) at once. These are how we make the "build script" to do those 5 building steps from earlier.
  - ANT is the old way, but Gradle is the new hotness
  - You'll be using Gradle for first homework (Java review), and we'll be poking at it periodically through the course.
    - We'll be doing minor tweaks to build files, not learning the entirety of the gradle build system (though as you get into professional development systems you'll want to check them out)
- Finally, the [Android Studio & SDK](http://developer.android.com/sdk/index.html).
  - This is our IDE and build system (everything else goes to support this).
  - Make sure the SDK command-line tools are installed!
    - put `tools` and `platform-tools` on the `PATH`
    - run `adb` to check
- SDK [Tools](http://developer.android.com/tools/help/index.html) include:
  - `android`: does SDK/AVD (virtual device) work. Basically IDE commands, but from command-line
  - `emulator` runs the emulator
  - `adb` "Android Device Bridge"; connection between your computer and the device (physical _or_ virtual). Used for console output!
  - all of these are built into IDE, but command-line is a good fall-back!


### Hello World [30min]
In time remaining, lets get an App up and running and see what we actually will be working with!
- Hopefully everyone got email about installing Android Studio over the weekend? If not... can at least see what this will look like.
1. Launch Android Studio if you have it (may take a few minutes to open)
- Start a new project!
  - use uwnetid in domain
  - note project location! Desktop is good for now
  - Target: this is the "minimum" SDK you support. Going to target Ice Cream Sandwich (4.0.3) for most this class. This is the earliest version we'll support.
    - Note that this is different than the "target SDK", which is the version you tested on! We'll adjust this in a moment, and will test on API 21 (Lollipop).
    - If you're testing on a physical device running older version, you can target that. But we'll grade at the 5.0 target.
- Select an Empty Activity
  - Activities are "Screens" in your application (things the user can do). We'll talk more about this on Wednesday.
- And boom, have an Android app! Aren't frameworks lovely?

#### Emulator
We can run our app by clicking the "Play" or "Run" button

We will need to define a device, so let's make an emulator!
- Nexus 5 is a good choice
  - Use Lollipop, Google APIs, x86!
  - Snapshot to speed up loading
- Specify **hardware keyboard**
- Want to go in and edit it (`Tools > Android > AVD Manager`) so it accepts keyboard input!

Can slide to unlock, and there is our app!

#### Project Contents
So what does our app look like in code? What do we have?

Note that Android Studio by default shows the "Android" view, which organizes files thematically. If you got the "Project" view you can see what the actual file system looks like.

- `app/` folder contains our application
  - `manifests/` contains the **Android Manifest** files, which is sort of like a "config" file for the app
  - `java/` contains the Java source code for your project
    - And we can find the `MyActivity` file in here
  - `res/` contains resource files used in the app. These are where we're going to put layout/appearance information
- Also have the `Gradle` scripts. There are a lot of these:
  - `build.gradle`: Top-level Gradle build; project-level (for building!)
  - `app/build.gradle`: Gradle build specific to the app **use this one to customize project!**
    - we can change the target SDK here!
  - `proguard-rules.pro`: config for release version (minimization, obfuscation, etc).
  - `gradle.properties`: Gradle-specific build settings, shared
  - `local.properties`: settings local to this machine only
  - `settings.gradle`: Gradle-specific build settings, shared
  - ANT would give:
    - `build.xml`: Ant build script integrated with Android SDK
    - `build.properties`: settings used for build across all machines
    - `local.properties`: settings local to this machine only
    - We're using Gradle, but be aware of ANT stuff for legacy purposes
- `res` has resource files. These are **XML** files that specify details of the app--such as layout.
  - `res/drawable/` : graphics (PNG, JPEG, etc)
  - `res/layout/` : UI XML layout files
  - `res/mipmap/` : launcher icon files
      - fun fact: MIP comes from "multum in parvo", latin for "much in little". Map cause image mapping
  - `res/values/` : general constants
  - See also: http://developer.android.com/guide/topics/resources/available-resources.html
    - We'll talk about these more next week!

Let's look at what the application does. We'll revisit this on Wednesday, but this will let us start seeing how the framework is structured
- We start with the **Activity** Java source
  - We extend [`Activity`](http://developer.android.com/reference/android/app/Activity.html) (actually a subclass that supports Material Design stuff), making our own customizations.
- Override `onCreate()` method that is called by the framework when the Activity starts (more on lifecycle tomorrow).
  - Call super, and then `setContentView` to specify what the content (appearance) of our activity is.
  - Passing in a value from something called `R`. What _type_ of thing is this? (It's a class!)
  - `R` is a class that is **generated at compile time** and contains constants that are defined by the "resource" files!
    - Those files are converted into Java variables, which we access through the `R` class.
- `R.layout` refers to the "layouts" resource, so can go there.
  - Can open then up in "design" view. This lets you use GUIs to lay out your application (like a powerpoint slide). But frowned upon--much cleaner/nicer to write it out in code.
    - Same difference between writing your own HTML and using FrontPage or DreamWeaver or Wix. Legit, but less nice.
  - We can see the XML: tags, attributes, values. Tags nested inside one another.
  - Defines a layout, and inside that is a TextView (view of some of text), which has a value--text!
    - Can change that, and re-run the app!
  - If time: can also define this in `values/strings` (e.g., as a constant, refer to as `@string/message`)
- We'll talk about the layout and using these resources _A LOT_ more next week.
- If time: we can also set an icon! (`File > New > Image Asset`)

### Conclusion
That gets us started! Tomorrow we'll do some Java review and go into more details about these Activity things.

#### Action Items:
* Sign up for GitHub and let us know your username!
* Get started on Homework 1 (warmup)!
  - Two parts: some Java work to practice with those concepts (we'll demo the second part more on Wednesday), and then basically making the Hello World app we just created.
  - If you get started done early and can dive into Hwk 2, that can help.
