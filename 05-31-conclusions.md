## Conclusion

### Admin & Presentations
Today is going to be our "last day of class" (because tomorrow are project demos!)

Plan: we're going to:
  - do a few concluding remarks (10mins)
  - then take 15 minutes to do course evaluations (15mins)
  - then go over the logistics for tomorrow (10min)
  - then any time left over you can check in with the group (15min)

### Conclusions
So to sum up: **what did we do in this class?**

- We considered the **Android Framework**: a system that acts as a "frame" for logic/content/code that we want to add to create our own applications. This framework is designed to let us deal with the restrictions and limitations of mobile platforms: limited resources and memory, different modes of interaction, and a heterogenous set of devices (different screen sizes, processors, etc.). The framework is structured the way it is to try and support these limitations (e.g., minimizing memory usage)

- We started with `Activities` and their lifecycles, which is the main "frame" for our work, which we can fill in the details of
  - We define our logic in the Activities, and our _content_ in XML resources (for flexibility!)

- This really leans on that _model-view-controller_ architecture... and really once we had `Activities` we just dove off from there
  - We covered Views to use (buttons, lists, etc), different ways to structure the Controller (fragments, etc), and ways we could store the data for our model (files, preferences, SQL)
  - And how to hook all these together!

- We used `Intents` to have Activities communicate between one another, and other forms of telephony and networking to talk to other devices

- We covered a number of "widgets" that are part of a normal Android app: things like notifications, alerts, menus, and preferences (though there are more as well!)

- And we also looked at mobile-specific capabilities: location, motion, touch, etc. The framework is intended to make harnessing these capabilities as easy as possible.

On the other hand, there are some notable themes relevant to making Android apps that **we did not cover** (because 10 weeks). In particular:

1. [**Design**](https://developer.android.com/design/index.html). While we pointed at this a little bit (and tried to follow recommended practices), there is a lot more that goes into designing an effective mobile application.

  - Google provides [lots of details and patterns](https://developer.android.com/design/patterns/index.html) on what "Android Design" is; I encourage you to look through these documents to get a feel for what the ecosystem should feel like. (There are lots of diagrams and pictures, which makes this fun reading)

  - Of course you can design an interface however you want, but there are benefits to being **consistent** with other applications (like the ones provided by Google and the OS).

  - You can also take INFO 365: Mobile App Design for more in-depth study, though that focuses on mobile interfaces in general than Android patterns specifically

2. [**Publishing**](http://developer.android.com/tools/publishing/preparing.html) and marketing. This is in part because it costs money (which I don't want to require), and because this isn't a business class: we're developing not selling.

  - Publishing an app [on the Play store](http://developer.android.com/distribute/googleplay/start.html) is not too difficult, and is well documented with lots of guidance from Google on how to best manage your app (i.e., in terms of updates, advertising, etc).

  - The most complicated piece is signing a release `apk`, which you've done (and will do for your project)
    - You'll want to pick a good package-name: once you've released your app, you can't change it (other than releasing a different app)
    - Turn off logging and debugging! This slows down the app and eats up the log file; so removing these is a good idea
    - Review all of your manifest and gradle settings to make sure everything is up to date

  - Google had a lot of details about [how to prepare your app for publishing](http://developer.android.com/tools/publishing/preparing.html#publishing-configure), including a handy [Launch Checklist](http://developer.android.com/distribute/tools/launch-checklist.html).

Again, I point you at the documentation for further learning; you should be comfortable following and interpreting the Android docs by now!

#### Parting Thoughts
So a couple of concluding thoughts before we wrap up---things that I want to make sure you keep in mind and take-away from doing mobile development:

___Develop for Flexibility___
Android in particular runs across a wide variety of different devices (from low-end to high-end, from old to new). And the framework is used for increasingly large number of platforms (including wearables, auto, etc). So you want to design your applications to be as **flexible** as possible.
- We've looked at a number of ways to support flexibility (resource configurations, permissions/feature checking, compatibility libraries, etc). These are a core part of the framework _because_ of the variety in the platform... but it's also something you should keep in mind with your own designs.
- Don't assume that phones will be a certain size or used in a certain way. Let the user solve their problem in a variety of ways!

___Develop for Universal Usability___
Android devices _and users_ may have limitations that you and your device don't:
- Device limitations like small screens, slow processors, etc
- Context limitations like lack of data access or GPS signal
- User limitations (physical disabilities, etc)

As you're developing your apps, you want to make sure to support users and devices with these limitations. This wlll make the apps work better for _everyone_
- Using relative measurements like font sizes and layouts to work across less-capable devices
- Keep resource and data usage low (makes it run faster for everyone!)
- Support accessibility systems like Talkback; consider how to integrate other modalities like voice and motion
- Make sure your app "gracefully degrades" if features like GPS or sensors aren't available


___Design for New Experiences___
Finally, when building mobile apps think about what makes these apps unique to _mobile_. As you've seen in INFO 343: the web is capable of a lot of functionality, and mobile phones have full-featured browsers. So you'd want to think about why this should be a mobile application (and only work on mobile platforms) rather than a web site (and work everywhere).
- Some of it is speed and ease of use and how that fits into a user's interaction (e.g., open up the Twitter app rather that going to Twitter)
- Lots of capabilities unique to phones to harness: location sensing, touch and motion, etc.

I challenge you to think through these issues to make sure you're developing something that's useful and _actually solves a problem in the world_. And that goes beyond this class, but for all your work in the future.

Any last questions/thoughts/etc about the course?


### Evaluations
Well you do have one other forum for (anonymous) feedback for the course: evaluations! For this course, there are **two (2)** evaluations you need to complete:
1. _The course evaluation_ This should be in your email (check for a reminder from yesterday).
  - It's really important that you feel these out; I read them very carefully to think about how to improve the course, and overall we use them to guide our curriculum development (e.g., what courses remain permanent)
    - In particular: the first **4** questions are used to produce overall "scores" that the administration likes, and at the end there should be a pair of extra questions about assignments that are useful to me.

2. _Peer evaluations_ I sent out a link to this in yesterday's email, but you should also be able find it in Catalyst. This is to give me feedback about if there were any issues with the "group-work" nature of the project (namely: if there were problems I didn't hear about, or if someone did above-and-beyond work we should acknowledge).
  - The _combined_ feedback will be used to weight individual scores on the project. So if you did extra work according to your team, you can get extra credit. If you did less work according to your team, well...
  - This will be available until Thursday afternoon; can fill it out tomorrow after class once project is turned in, or Thursday morning when you have a chance to rest, or today!

To make sure that evaluations actually get completed (particularly course evaluations). I'm going to give you time to fill them out **now**. I'm going to step out of the room for 10-15 minutes; you should use this time to fill out the course eval. I'll then come back and we'll talk about the plans for tomorrow.
  - This is intentional: you have no where else to go and need to hang out to figure out how to present your project, so might as well fill in the eval!

Go forth: see you all in 15.


### Presentation Logistics
Final projects are due tomorrow morning **at the start of class**. That means you should have pushed your final code to GitHub and submitted a link on Canvas.
  - Also include a _signed release apk_ that I can install to test your app.

Because many people are using location and proximity-based systems, in order to make sure you can really show off your app we're going to do demos outside! So yes, we can run around to show off the location changes :)
  - It's supposed to be really nice out too. Mobility FTW.
  - Though note you'll want to be careful that your app/device is still usable in the sun, though there should be some shade/clouds if needed

The best suggestion I've had is to meet out on the [Quad](https://www.google.com/maps/dir//47.6569089,-122.3077957/@47.6567629,-122.3073261,367m/data=!3m1!1e3). This should give us a nice spot to work out, but I'm also open to suggestions
  - ((i.e., if we did it right outside Mary Gates, you could also show off stuff to people coming in to the see the Info 200 students!))

We will gather at the indicate spot at **10:30am** (our usual class starting time). Be there!!

Demos will be given a bit like a cross between a poster session and game of bingo. What we will do is split the teams up into 3 "groups" of 3 teams each. Within each group, each team will give a "demo" of their application for **10 minutes**.
  - Should have app running on two devices, so that someone who _isn't_ on your team can play around with it!
  - Plan on a 5 minute "pitch" and then 5 minutes to answer questions/play around/etc.

After one group demos for 10 minutes, then the next group can go, then the next. And after 30 minutes... we'll shuffle around. We'll reform the 3 groups, and you'll give your demo again for two new teams! And then 30 minutes after that, we'll do _one more_ switch. So in the end, you'll have seen all the project demos and given your demo 3 times!
  - Have someone different give the "demo" each time, so that everyone has a chance to practice and take credit for the app :)
  - This will be good practice for capstone if you didn't do that already.

_While you are doing this_, Min and I will be running between groups like a crazy-person so we can get a demo from each group. It will be very precise and scheduled; I'll give you print-outs of the ordering so you can follow along. It'll just be nuts.

Note I often do a kind of "peer review" form for people to fill out, but I'm going to trust that everyone will show up and participate in this process... right?

Any questions about the day?

### Work time
Great, and that wraps things up for us. You can use any extra time to check in with your group about the final set of changes you need to make before tomorrow.
- Note that I will be in my office during office hours today if there are questions/issues/etc.

Thanks so much for a great class!


### Lecture References
- [Android design patterns](https://developer.android.com/design/patterns/index.html)
- [Publishing](http://developer.android.com/tools/publishing/preparing.html)
- [Distributing on the Play store](http://developer.android.com/distribute/googleplay/start.html)
- [Launch Checklist](http://developer.android.com/distribute/tools/launch-checklist.html).
