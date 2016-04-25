## Location

### Admin [5+min]
- Homework check-in
  - Want to apologize again for the complexity of the Todoer assignment.
    - I think the relationship between the database, the loader, the model, the adapter, and the view wasn't as clear as I hoped it would be
    - _Any lingering questions about how those pieces work? We have some time, and so am happy to address it since future assignments/projects will draw from these ideas!_
    - Also a few missing instructions on the SQL-esque "gotchyas" which I forgot to include.
  - Next homework (Yama) due Wednesday. I've answered a couple of questions about it which suggests people are getting started on it (and the questions have displayed a good understanding of the overall framework, which suggests learning is occurring!)
- Fork and clone the repo for today. We're going to tweak the SDK/Emulator again, but in a little bit.

## Localization
Today we're going to talk about **Location** and **Mapping**. But before we get into the Android code, I want to give a little bit of background/context for some of the technologies that are in play.

**Localization** in this context is the process of determining _location_. Why is this important?

- Android was originally developed as an operating system for _mobile_ devices. Key word: **mobile**. What makes phones and tablets special, and different from desktops, is that they can and do move around. Their position and location matters _significantly_ for how they are used; it's a major part of what separates the functionality of Android apps from any other computer application.

- Localization gives apps the ability to create new kinds of user experiences, and to adjust their functionality to fit the user's _context_ (classic example: it can determine if you are at home, in the office, or on the bus)! In fact, one of the winners of the _first_ Android Developer Challenge (2008) was [Ecorio](http://web.archive.org/web/20100209012355/http://www.ecorio.org/), an app that figured out whether you were driving, walking, or busing and calculated your carbon footprint from that. Localization lets us know about the user's situation (though mobile phone location is [not necessarily](http://dx.doi.org/10.1007/11853565_8) a proxy for user location).

- Note that this perspective is coming out of the the realm of [Ubiquitous Computing](https://en.wikipedia.org/wiki/Ubiquitous_computing), a discipline that considers technology that is _ubiquitous_ or everywhere, to the point that it "blends into the surroundings." That's _my_ take on why phone development is important; so that you can compute without thinking about it.

  - I highly recommend you read Mark Weiser's [original 1991 Scientific American article](http://dx.doi.org/10.1145/329124.329126). It's a neat vision and is foundational for a lot of modern research into mobile systems. It makes the "official" start of the field of Ubicomp.

### Localization Techniques
Ubicomp researchers have been developing localization systems for _years_. A classical reference is a [survey paper](http://dx.doi.org/10.1109/2.940014) by Jeff Hightower (who was getting his PhD in the CSE department here at UW!)

- Early Example: _Active Badge_ (AT&T) has a name-badge emit an infrared message, that is picked up by sensors in a room to determine where the wearer is! This is room-level precision; improvements and _triangulation_ (calculating angles to what you see) got it down to about _10cm_ precision. However, this requires a lot of infrastructure to be built into a particular room.

With Android, we're interested in more general-purpose localization. Mobile devices use a couple of different kinds of localization (either independently or together).

#### GPS
**GPS** is the most common, and what most people think of when they think of localization. [GPS](https://en.wikipedia.org/wiki/Global_Positioning_System) stands for "Global Position System"---and yes, it can work anywhere on the globe.

_How does it work?_
GPS's functionality depends on satellites: 24 satellites in high orbit (not geo-synchronous) around the earth. Satellites are distributed so that 4 to 12 are visible from any point on Earth, and their locations are known with high precision. These satellites are each equipped with an atomic, synchronized clock that "ticks" every nanosecond. At every tick, the satellite broadcasts its current time and position. You can almost think of them as _really_ loud alarm clocks.

The thing in your phone (and your car and whatever) that you call a "GPS" or a "GPS device" is actually a _GPS_ ___receiver___. It is able to listen for the messages broadcast by these satellites, and determine it's position based on that information.

How? First, the receiver calculates the _time of arrival_ (TOA) based on its own clock and comparing time-codes from the satellites. It then uses the announced _time of transmission_ (TOT; what the satellite was shouting) to calculate the [time of flight](https://en.wikipedia.org/wiki/Time_of_flight), or how long it took for the satellite's message to reach the receiver. Because these messages are sent at (basically) the speed of light, the [time of flight] is equivalent to the distance from the satellite!

  - There is some other synchronization work that is done to make sure clocks are the same, but we won't go into that here.

And once it has distances from the satellites, the receiver can use [trilateration](https://en.wikipedia.org/wiki/Trilateration) to determine it's position based on the satellites it "sees" (Trilateration is like Triangulation, but relies on measuring distances rather than measuring angles. Basically you construct three spheres of a given radius, and then look to see where they intersect). Whew!

- Important terms here: _time of flight_, _trilateration_

GPS precision is generally about 5 meters (15 feet); however, by repeatedly calculating the position (since ticks every nanosecond), we can use _differential positioning_ to extrapolate position with even higher precision, increasing precision to less than 1 meter! This is how Google can determine where you're walking.

But the biggest problem with GPS: you need to be able to see the satellites! This means that it doesn't work indoors, as many building walls block the signals. Similarly, in very urban areas (think downtown Seattle), the buildings can bounce the signal around and throw off the TOF calculations, making it harder to pinpoint position accurately.

- Also: requires a lot of energy to constantly listen for the alarm, so can be a big hit to battery life, which is something we need to worry about with mobile devices!

#### Cell Tower Localization
But your phone can also give you a rough estimate of your location even _without_ GPS. It does this through a couple of techniques, such as relying on the cell phone towers that provide the phone network service. This is also known as [**GSM localization**](https://en.wikipedia.org/wiki/Mobile_phone_tracking#Network-based) (Global System for Mobile Communications; the standard for cell phone communication used by many service providers). The location of these towers are known, so we can determine location based off them in a couple of ways:
- If you're connected to a tower, you must be within range of it. So that gives you some measure of localization right off the bat. Though not very accurate (you might be _anywhere_ within that range).
- If you can see multiple towers (important for handoff purposes), you can trilaterate the position between them (e.g., finding the possible overlapping area and picking like the middle of that). This can give accuracy within 50m in urban areas, with more towers producing better precision.

#### WiFi Localization
But wait there's more! What other kinds of communication antennas do you have in your phone? **WiFi**. As WiFi has became more popular, efforts have been made to identify the _location_ of WiFi hotspots so that they too can be used for [trilateration and localization](https://en.wikipedia.org/wiki/Wi-Fi_positioning_system). This is often done through crowdsourced databases, with information gathered through [war driving](https://en.wikipedia.org/wiki/Wardriving).

War driving involves driving around with a GPS receiver and a laptop, and simply recording what WiFi routers you see at what locations. This then all gets compiled into a database that can be queried--given that you see _these_ routers, where must you be?

- Google got in [hot water](http://www.wired.com/2012/05/google-wifi-fcc-investigation/) for doing this as it was driving around to capture Street-View images.

WiFi localization can then be combined with Cell Tower localization to produce a pretty good estimate of your location, even without GPS.

And in fact, Google provides the ability to automatically use all of these different techniques, abstracted into a single method call (which we will get to in a moment).

I want to flag that just like the old _Active Badge_ systems, all of these localizations systems rely on some kind of existing infrastructure: GPS requires satellites; GSM requires cell towers, and WiFi needs the database of routers. All these systems require and react to the world around them, making localization influenced by the actual location as well, as well as both social and computational systems!

### Representing Location
So once we have a location, how do we represent it?

First, note that there is a philosophical difference between a "place" and a "space." A _space_ is a location, but without any social connotations. For example, GPS coordinates, or Cartesian xy-coordinates will all indicate a "space." A _place_ on the other hand is somewhere that has social meaning: Mary Gates Hall; University of Washington; my kitchen. Space is a computational construct; place is a human construct. When we talk about localization with a mobile device, we'll be mostly talking about _space_. But often _place_ is what we're really interested in, and we may have to translate (Google does provide a few ways to move between the two, such as with its [Places API](https://developers.google.com/places/)).

Our space locations will generally be reported as two coordinates: **Latitude** (lat) and **Longitude** (long) (there's also **altitude** or height, but that isn't super relevant for us). _What are latitude and longitude?_

- Latitude is the **angle** between the equatorial plane and a line that passes through a point and the center of the earth--the angle you have to go up (north-south) the earth's surface from the equator. Effectively, it's a measure of "north/south". Latitude is usually measured in _degrees north_, so going south of the equator leads to a negative latitude (though this can be expressed positively as "degrees south").

- Longitude is the **angle** between the prime meridian plane and a line that passes through a point and the center of the earth--the angle you have to go across (east-west) the earth's surface from the meridian. Effectively, it's a measure of "east/west". Latitude is measured in _degrees east_, so going east of the meridian. That mean that the western hemisphere has "negative longitude" (though this can be expressed as positive "degrees west").

Example: [UW's GPS Coordinates](https://www.google.com/search?q=uw+gps+coordinates) are N/W, so this would be expressed as N (positive) and E (negative).

The distance between degrees and miles depends on where you you are (particularly for longitude!) However, as a very rough estimate in this part of the world, .01 degrees is _about_ a mile (not accurate, but so you have a sense of scale).


### Break: 5min before we get back to programming, as things download!


## Android Location
Okay, let's go ahead and make an app that will access the phone's location. We'll just log it out for now, and then hook up a map at the end of class (or tomorrow in lab, as well as in your homework).

First thing's first: we need to make sure we can access [Google Play Services](http://developer.android.com/google/play-services/setup.html); these are a special set of libraries that provide additional functionality to Android. That functionality will include the location and mapping tools we're interested in (Much of this functionality was originally built into core Android, but Google has since been moving it into a separate app that can be more easily distributed and updated!)

There are a few steps to this...
1. First we need to make sure we've downloaded the proper library. Go to `Tools > Android > SDK Manager` to open up the manager for the various versions of Android we've installed.
  - Should have 6.0 installed for this week's assignment. Not needed for today, but will need it for Wednesday!
  - Under `SDK Tools`, select `Google Play Service` and `Google Play Repository` to download those
2. Next, we want to make sure our emulator will support these services. Go to the `AVD Manager`, and make sure the _target_ platform include the `Google APIs` (we may need to download it).
3. We'll also need to modify your `build.gradle` file so that you can get access to the Location classes. In the ___module-level___ `build.gradle` file, under `dependencies` add
  ```
  compile 'com.google.android.gms:play-services-location:8.4.0'`
  ```
  This will load in the location services (but not the other services, which may require keys)

Tada, now we have access to this set of libraries! Yay!

Lastly, we'll need to request permission to access the location. There are two permission levels we can ask for: `ACCESS_COARSE_LOCATION` (for GSM/WiFi level precision), and `ACCESS_FINE_LOCATION` (for GPS level precision). We'll use the later because we want GPS-level precision
- This is a **dangerous** permission, so in Marshmallow we'd need to ask for permission at run-time. We'll talk about how to do that more on Wednesday.

### Google Play Services
We're going to use Google Play Services to get our location. The Google APIs provide a nice set of methods for [accessing location](http://developer.android.com/training/location/retrieve-current.html) (without us needing to specify the source of that localization, GPS or GSM), and is the recommended API to use.
- There is a built-in `android.location` API (e.g., for non-Google based Android devices), but it's not recommended practice and is harder to use.

The first thing we need to do is get access to the API; we do this with a [`GoogleApiClient`](http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.html) object. We construct this object in the `onCreate()` callback, using a `GoogleApiClient.Builder` (yay another Builder!):
```java
if (mGoogleApiClient == null) {
    mGoogleApiClient = new GoogleApiClient.Builder(this)
            .addConnectionCallbacks(this)
            .addOnConnectionFailedListener(this)
            .addApi(LocationServices.API)
            .build();
}
```
We need to set up what are called the Connection Callbacks, which are callbacks that will occur when we connect to the Google APIs. We do this by implementing the `GoogleApiClient.ConnectionCallbacks` and `GoogleApiClient.OnConnectionFailedListener` interfaces. Each have a single method that we can fill in; in particular, the `onConnected()` method is where we can "start" our API usage (like asking for location!)
- `onSuspended` and `onFailed` are for when the connection is stopped (like `onStop`) or if we fail to connect. See [Accessing Google APIs](https://developers.google.com/android/guides/api-client) for details.

We also specify that we want to access the LocationServices API, before creating the client.

Finally, we need to actually connect to the client. We do this in our `onStart()` method (and disconnect in `onStop()`):
```java
protected void onStart() {
    mGoogleApiClient.connect();
    super.onStart();
}
```
This of course, will lead to our `onConnected()` callback being executed.


### Location
Once we have the the client connected, we can start getting the location! This can actually get pretty complicated, but we'll go through it slowly.

To access the location, we're going to use a class called the [FusedLocationApi](https://developers.google.com/android/reference/com/google/android/gms/location/FusedLocationProviderApi). This is a "unified" interface for accessing location. It fuses together all of the different ways of getting location, providing which-ever one best suits our specified needs. Can think of it as a "wrapper" around more detailed location services.
  - It will let us specify at a high level whether we want to trade accuracy for power consumption, rather than us needing to be explicit about that. And it will make decisions about what how best to fetch location given our stated needs.

We're going to specify this "high level" requirement using a [`LocationRequest`](https://developers.google.com/android/reference/com/google/android/gms/location/LocationRequest) object, which specifies the details of our request (e.g., how we want to have our phone search for it's location).
```java
LocationRequest request = new LocationRequest();
request.setInterval(10000);
request.setFastestInterval(5000);
request.setPriority(LocationRequest.PRIORITY_HIGH_ACCURACY);
```
- We create the object, then specify the "interval" that we want to check for updates. We can also specify the "fastest" interval, which is the maximum rate we want updates (assuming they are available). It's a bit like a minimum and maximum. (5s - 10s is good for real-time navigation).
- We also specify the [priority](https://developers.google.com/android/reference/com/google/android/gms/location/LocationRequest#constant-summary), which is the indicator to the FusedLocationApi about what kind of precision we want. `HIGH_ACCURACY` basically means GPS (trade power for accuracy!)

Once we have this request in place, we can send it off through the `FusedLocationApi`:
```
LocationServices.FusedLocationApi.requestLocationUpdates(mGoogleApiClient, request, this);
```
- The first parameter is going to be the GoogleApiClient, and the second will be the request we just made.
- The third parameter for the `requestLocationUpdates()` method is a `LocationListener`---an object with a callback that can be executed when the location is updated (when we move). So we'll make ourselves into one of those by implementing the interface and filling in the `onLocationChanged()` method.
  - This listener's callback will be handed a `Location` object, which contains the latitude/longitude of the location. We can then use that location (such as display it). We can access the lat/lng with getters.
  ```java
  ((TextView) findViewById(R.id.txt_lat)).setText("" + location.getLatitude());
  ((TextView) findViewById(R.id.txt_lng)).setText("" + location.getLongitude());
  ```

So How can we see this in action (make sure the phone has gotten a location)? Particularly if we're indoors?
- We can give a "fake" location to the emulator! That "extras" menu in the sidebar (where we sent SMS messages) has the ability to "send" the phone a location. It's like our fingers are the GPS satellites!
  - Test by giving it UW (47.6550 N, -122.3080 E), and you can watch it update!
- The `FusedLocationApi` class also has a `setMockLocation()` method, which can allow you to programmatically define locations (e.g., you can make a button that changes your location).

And with that we can now send different locations to the emulator, and watch our position update! Yay!
- Note that you may need to start up the `Maps` application to make sure settings are correct. See [here](http://developer.android.com/training/location/change-location-settings.html) for how we can prompt for that ourselves (it's a lot of repetitous code, so leaving it to you)

Okay, so let's review what's going on:
- We start by creating and connecting to a `GoogleApiClient`, which is going to let us talk to the Play Services.
- This may not be able to connect (or may take a moment), so we have a _asynchronous_ callback for when it does.
- Once it connects (in that callback), start up a repeated request for location updates. These may also take time, so we have _another_ callback.
- And when we get a location update, we finally update our View.

That's pretty much what is involved in working with location. Lots of pieces (because callbacks all over the place!), but that does the work.

Questions?


## Permissions (if time)
Today we're dealing with location, which as we mentioned is sensitive information that you need permission to access. So let's talk about how Android handles [permissions](http://developer.android.com/guide/topics/security/permissions.html) in a bit more detail:
- The biggest part of Android's design is the idea of [_sandboxing_](https://en.wikipedia.org/wiki/Sandbox_(computer_security)): each application gets its own "sandbox" to play in (where all the toys are kept), but isn't able to go outside the box. Apps are not 100% in the sandbox, but parts that are outside the sandbox are things that would be impactful to the user (like network or file access, etc).
  - In order to get outside the sandbox you need to explicitly request permission, as we've done before.

- This also happens at a package level, where packages (applications) are isolated from packages _from other developers_; you can use certificate signing (which occurs as part of our build process automatically) to mark two packages as from the same developer if we want them to interact.

- Additionally, Android's underlying OS is Linux-based, so it actually uses Linux's permission system under the hood (with user and group ids that grant access to particular files or processes).

We ask for permission by declaring usage explicitly in the `Manifest`. These permissions we can access are divided into two categories: [normal and dangerous](http://developer.android.com/guide/topics/security/permissions.html#normal-dangerous).
- [**Normal permissions**](http://developer.android.com/guide/topics/security/normal-permissions.html) are those that may impact the user (so require permission), but don't pose any serious risk. They are granted by the user at _install time_; if the user chooses to install the app, permission is granted to that app. See the list for examples.
- **Dangerous permissions**, on the other hand, have the risk of violating privacy concerns, or otherwise messing with the user's device or other apps. These _also_ need to be granted at install time. But _IN ADDITION_, in the latest version of Android (6.0 Marshmallow, API 23), users _additionally_ need to grant permission access **at runtime**, when you try to actually invoke the "permitted" action.
  - The user grants permission via a pop-up. Note that permissions are granted in "groups", so if the user agrees to give you `RECEIVE_SMS` permission, you get `SEND_SMS` permission as well. See [the list](http://developer.android.com/guide/topics/security/permissions.html#permission-groups).
  - When you grant permission at runtime that permission stays granted as long as the app is installed. But the big difference is that the user can choose to **revoke** or deny privileges at ___any___ time (though System settings) so you always have to check if the user has granted the privileges or not.
    - You don't know if the user _currently_ has given you permission!

Let's walk through how we grant permissions at run-time, by making our Location Demo support Marshmallow.
- For setup: we'll need to change our target SDK to **23**, AND make sure we're running 6.0 on the emulator (because this only works if the OS supports it AND you've targeted that high!)
  - If you're working off a hardware phone... well you won't be able to totally reproduce this demo :/

First we _still_ need to request permission in the `Manifest`; if we haven't announced that we might ask for permission, we won't be allowed to ask in the future.

Before we do something that requires permissions we can check that we have permission with:
```java
int permissionCheck = ContextCompat.checkSelfPermission(activity, Manifest.permission.PERMISSION_NAME);
```
- This is basically a look-up function. It will return either `PackageManager.PERMISSION_GRANTED` or `PackageManager.PERMISSION_DENIED`

If permission is granted, great! We can go about our business. But if it's not, then we have to ask for it. We do this by calling
```java
ActivityCompat.requestPermissions(activity, new String[]{Manifest.permission.PERMISSION_NAME}, REQUEST_CODE);
```
This method takes a context and then an array of permissions that we need access to. We're also going to give it a request code (an `int`), which we can use to identify that request in a callback (just like we've done when sending an Intent for a result... is this pattern looking familiar)?
- This is a lot like sending an `Intent`, if the Intent were to the permission system!

And then we need that callback:
```java
public void onRequestPermissionsResult(int requestCode, String permissions[], int[] grantResults) {
    switch (requestCode) {
        case REQUEST_CODE: {
          if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
            //have permission! Do stuff!
          }
        }
    }
}
```

We check which request we're hearing, what permissions were granted (if any; they can piece-wise grant permissions!), and then we can react if everything is good...like by finally requesting location!

So in the end, our location system has ANOTHER level of callback. Let's review...
- We start by creating connecting to a `GoogleApiClient`. We have the _asynchronous_ callback (`onConnected()`) for when it does.
- We then need need to request permission. This may take time for use input, so we have another callback (`onRequestPermissionResult()`) for when it does.
- If that result is possible, then we can go back and start up out repeated request for location updates. These may also take time, so we have _another_ callback (`onLocationChanged`), when we can finally use the `Location!`

3 levels of callback: to start the API, to request permission, to request the location, and THEN to actually get a location! (Lots of whew)

Note that if they deny us permission once, we might want to try and explain _why_ we're asking permission (see [best practices](http://developer.android.com/training/permissions/best-practices.html)). Google offers a utility method (`ActivityCompat.shouldShowRequestPermissionRationale()`) which we can use to check if they've denied us once. And if that's true, we might show a Dialog or something to explain ourselves--and if they OK that dialog, then we can ask again.
- This would add _another callback_ of some kind!

Note that for your homework this week (after Yama) you'll need to target SDK 23 and support run-time permissions.

Questions?

### Lecture References
- [UW's GPS Coordinates](https://www.google.com/search?q=uw+gps+coordinates)
- [FusedLocationApi](https://developers.google.com/android/reference/com/google/android/gms/location/FusedLocationProviderApi)
- [`LocationRequest`](https://developers.google.com/android/reference/com/google/android/gms/location/LocationRequest)
- [Request Location Settings](http://developer.android.com/training/location/change-location-settings.html)
- [Normal permissions](http://developer.android.com/guide/topics/security/normal-permissions.html)
- [Location Request priority](https://developers.google.com/android/reference/com/google/android/gms/location/LocationRequest#constant-summary)
- [Permissions best practices](http://developer.android.com/training/permissions/best-practices.html)
- [Request permission structure](http://developer.android.com/training/permissions/requesting.html#make-the-request)
