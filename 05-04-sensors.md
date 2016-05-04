## Touch and Sensors

### Admin
- Homework check-in: geopaint get finished? I've had a lot less questions, so assume it went better :)
  - Anyone draw anything really cool to share? [can pull up on GitHub]
  - Next assignment (Motion Game) is due week from Friday. Can do it in pairs again. Let me know about that.

#### Group selection! [10mins]
But in addition to assignment due next week, you also need to write a [project proposal!](https://canvas.uw.edu/courses/1041406/pages/project). Which means you need a group to work with.
- I know we don't have a lot of room, but everyone should stand up and go physically stand with your groups (useful way of getting people together). Be careful not to knock anything over!
  - If you don't have a group yet, come stand near the front of the room here.
  - Are you missing someone? What's your current idea that someone might be excited to join you for?
  - Or I can randomly assign
  - [[record groups]]

Fork & clone repo for today, as always.
- We're going to start in the `TouchActivity`, and then check out the `SensorActivity` as we go.


### Fling
Yesterday we set up some _very_ simple touch gestures: press and move. But we can also detect and react to more complex gestures: long presses, double-taps, or flings (a "flick" or swipe on the screen. See [the design docs](https://www.google.com/design/spec/patterns/gestures.html#gestures-drag-swipe-or-fling-details) for some design language involving these gestures). Android provides a [GestureDetector](http://developer.android.com/training/gestures/detector.html#detect) class that can help identify these actions.

The easiest way to use this---particularly when we're interested in a particular gesture (like fling)---is to _extend_ `GestureDetector.SimpleOnGestureListener` to make our own "listener" for gestures. We can then override the callbacks for the gestures we're interested in responding to: e.g., `onFling()`.
- Note that the [docs](http://developer.android.com/training/gestures/detector.html#detect) say we should also override the `onDown()` method and have it return `true` to indicate that we've "consumed" (handled) the event---similar to what we've done with OptionsMenus. If we return false from this method, then _"the system assumes that you want to ignore the rest of the gesture, and the other methods of GestureDetector.OnGestureListener never get called."_ It seems to work for me in either way... but we'll go ahead and follow their spec.

We can instantiate this class with `mDetector = new GestureDetectorCompat(this, new MyGestureListener());` (passing in an instance of the class we just defined)
- Then in our `onTouchEvent()` method, we can pass the event into our Gesture Detector to process!
  - Note that this method returns a `boolean` for whether or not a gesture was detected: if not, we could handle it "ourselves":
  ```java
  boolean gesture = this.mDetector.onTouchEvent(event); //check gestures first!
  if(gesture){
    return true;
  }
  ```

We can now add the ability to "fling" the ball, say by taking the fling velocity and assigning that to the ball!
- Also note we need to negate the velocities since they are registered as "backwards" from the coordinates we expect (this doesn't match the docs though). Scaling down the velocity to 3% seems to work okay for me.
- We can also have the ball slow down by 1% on each update so it drifts to a stop!


### Multi-Touch
One of the other neat things about modern touch screens is that they support [multi-touch](http://developer.android.com/training/gestures/multi.html), or the ability to detect two different "contacts" (fingers) independently. Multi-touch is actually a pretty awesome interaction mode; it's very ["sci-fi"](https://www.youtube.com/watch?v=NwVBzx0LMNQ).

Android lets you react to multi-touch by responding to `ACTION_POINTER_DOWN` events, which occur when a second "pointer" (finger) joins the gesture (after `ACTION_DOWN`).
  - The first finger starts the gesture with an `ACTION_DOWN` event. then subsequent fingers produce `ACTION_POINTER_DOWN` events
  - Similarly, there are `ACTION_POINTER_UP` events for removing fingers, until the last finger which causes the `ACTION_UP` event.

Here the tricky part: each finger that is down can cause events _independently_. That is, if we move **a** finger, then an `ACTION_MOVE` event will occur. So the question is, how do we know which finger caused the event?
- Underneath the hood, pointers are stored in a list, so each has a **pointer index** (a number). But these indices can jump around (for example, if I lift up and put down fingers, things can get reshuffled). In fact, the index is allowed to change from event to event---while they _usually_ stay in order, this is not enforced by the framework so we can't trust it.
- However, each pointer that comes down ___is___ assigned a consistent **pointer id** number that we can refer to it by. In general the first finger down will be id "0", and no matter what happens to the list order that pointer's id will stay the same.

So how do we use this?
1. When we detect an Action, we're going to want to figure out what **pointer index** caused the event. We can do this by calling the `MotionEventCompat.getActionIndex(event)` method. Note that this only actually applies to `POINTER_DOWN` or `POINTER_UP` events, otherwise it will just return `0` (for the "main finger").
  - An aside: the `MotionEventCompat` class is just a wrapper around `MotionEvent` methods, so that the correct version of `MotionEvent` is used. So we could alternatively just call `event.getActionIndex()`.

2. We can then get the unique **pointer id** for whichever finger we're looking at. We do this by calling the `MotionEventCompat.getPointerId(event, pointerIndex)` method. This will give us that unique id!

3. Now we know _which_ pointer has gone up or down, so we can respond to that.
  - e.g., for the `DOWN` action**s** we can add another touch ball to our DrawingView (via `synchronized` method). Stores in a Map by `id` so we can associate each ball with a pointer!
  - For the `UP` action**s** we can remove the ball by calling the synchronized method again.

In the `MOVE` action, our event actually doesn't track which finger has moved--instead, the **pointer index** of that action is always going to be 0 (the main pointer), even if a different finger moved!
- However, we do know _how many_ pointers are involved in this event: we can get that number with `MotionEvent.getPointerCount()` (or equivalent Compat method). Then we can just **loop** through all the pointers and react to their current status (some of them may not have moved, but we know at least one of them did!)
```java
int pointerCount = event.getPointerCount();
for(int index=0; index < pointerCount; index++){
    int pId  = event.getPointerId(index);
    view.moveTouch(pId, event.getX(index), event.getY(index));
}
```
  - Note that we get the **pointer id** of each pointer index, because we don't know the order of these things
  - We can also get the `x` and `y` coordinates of _that pointer_, which we can then associate with an id

And voila, we have multi-touch tracking!!
- Tracking individual ids is more commonly used to make sure you're _ignoring_ extra multiple touches. See [the docs](http://developer.android.com/training/gestures/multi.html) or [this developer blog post](http://android-developers.blogspot.com/2010/06/making-sense-of-multitouch.html) for details.


Note that we can respond to common multi-touch gestures (like "pinch to scale") by using _another_ kind of GestureDetector called a [`ScaleGestureDetector`](http://developer.android.com/training/gestures/scale.html#scale).
- Again, subclass the simple version (`ScaleGestureDetector.SimpleOnScaleGestureListener`), and fill in the `onScale()` method. You can get the "scale factor" from the gesture with the `.getScaleFactor()` method. We could (theoretically) use this to scale our circle, but I leave that as an exercise to the reader.

Any questions on touch gestures?

### Break

### Motion Sensors
We've looked at using gesture detection to support (more) complex touch interfaces. Touch gestures are good and well, but lots of modern laptops have touch screens too... and we're interested in the capabilities that make mobile devices _unique_. Because they are mobile, you can pick up and move the devices easy: shake them around, toss them in the air, etc. Android devices often come with [**sensors**](http://developer.android.com/guide/topics/sensors/sensors_motion.html) that are able to measure these movements.

Android supports lots of generic and specific sensors (see [the guide](http://developer.android.com/guide/topics/sensors/sensors_overview.html)), using an overall "sensor framework" to listen for and react to information detected by particular sensors. This is so that apps can harness what might be built into the device (from motion sensors like an **accelerometer**, to environmental sensors like **thermometers** or **barometers**). Additionally, the system is structured so you can develop and attach your own (hardware) sensors if needed, like if you wanted to connect a medical device of some kind (though that's much more low level than we'll be going).

- We're going to focus on the [**accelerometer**](http://developer.android.com/guide/topics/sensors/sensors_motion.html#sensors-motion-accel) today, which is used to detect acceleration force (e.g., how fast the device is moving). This sensor is found on most devices and is general very low power, which makes it useful for detecting motions!

  - The accelerometer is an example of a [_motion sensor_](http://developer.android.com/guide/topics/sensors/sensors_motion.html), which are used to detect device motion: tilt, shake, rotation, or swing. This is distinct from [_positional sensors_](http://developer.android.com/guide/topics/sensors/sensors_position.html) which determine where the device _is_ in space (sort of like GPS, but relative to the device rather than relative to the rest of the world).

  - We could also detect motion using **gravity sensors** (measures direction of gravity) or **gyroscopes** (measures rate of spin) or a number of other sensors, though these are a bit less common.
  - There are really lots of [Sensor Types](https://source.android.com/devices/sensors/sensor-types.html)

Note that if we want to make sure that anyone installing our app (e.g., from the Play Store) has the particular sensor we're using, we can specify that in the manifest with the `<uses-feature>` tag:
```xml
<uses-feature android:name="android.hardware.sensor.accelerometer"
              android:required="true" />
```
- This doesn't actually keep us from installing the app, it's just extra information.
- Otherwise, we don't need any special permissions to access the accelerometer.

In Android, we start our work with sensors by using the [`SensorManager`](http://developer.android.com/reference/android/hardware/SensorManager.html) class, which will tell us information about what sensors are available, as well as let us register listeners for sensors. (It's a lot like the managers for Fragments or Notifications in that respect). We get access to it with
```java
mSensorManager = (SensorManager) getSystemService(Context.SENSOR_SERVICE);
```
- Again, like how we accessed the [Notification Service](https://github.com/info498b-s16/04-20-notifications/blob/master/app/src/main/java/edu/uw/notsetdemo/MainActivity.java#L136).
- We can then do things like get a list of sensors with `.getSensorList(Sensor.TYPE_ALL)`, which returns a `List<Sensor>`. A [`Sensor`](http://developer.android.com/reference/android/hardware/Sensor.html) object represents a particular sensor, including things like its type (which is _not_ represented as subclasses, because otherwise we couldn't invent our own!)

In order to get information from the sensor, we need to _register a listener_ for events that it will produce. Again. This is a _really_ common pattern in interactive systems, and so we're going to do it a lot.
- We get access to the sensor using `.getDefaultSensor(type)` to determine if there is a particular sensor; if it's `null`, then the sensor is missing and so we can quit (`finish()`). We do this is `onCreate()`
- We will want to register the sensor in `onResume()` (and unregister in `onPause()`) this will "turn on/off" the sensor only when we're definitely using it, since sensors can cause significant battery drain (even if the accelerometer is on the low end of that).
  ```java
  mSensorManager.registerListener(this, mSensor, SensorManager.SENSOR_DELAY_NORMAL);
  ```
  - The `this` needs to be a `SensorEventListener`, so we'll need to implement that interface!
  - the `DELAY_NORMAL` specifies a 200,000 microsecond (200ms) delay between samples; we can also use `DELAY_GAME` for a faster 20ms delay.

Finally, we can use the sensor information by filling in the `onSensorChanged(event)` callback. This is called _whenever_ we get a new sensor reading!
  - We'll leave the `onAccuracyChanged()` method blank for now; that would be for handling if the sensor switches modes or something (like moving from GPS to WiFi localization).

Sensor readings are stored in the `event.values` array, but what those numbers mean depends on the sensor that we're working with. With the **accelerometer**, each entry is the acceleration force on a different _axis_ [x,y,z].
- We can take those results and log/display them!

What do you notice about the acceleration readings while the phone is sitting on the table (not moving)?
- For the accelerometer, gravity is always exerting an accelerating force, even when at rest!
- So we'd need to "factor out" the gravity in order to figure out actual acceleration. This actually requires some linear algebra. [The docs](http://developer.android.com/guide/topics/sensors/sensors_motion.html#sensors-motion-accel) have an example of the math, with an additional version in [the sample app](https://github.com/android/platform_development/blob/master/samples/AccelerometerPlay/src/com/example/android/accelerometerplay/AccelerometerPlayActivity.java).

But an easier solution is to use the [`LINEAR_ACCELERATION`](https://source.android.com/devices/sensors/sensor-types.html#linear_acceleration) Sensor, which automatically subtracts out gravity! This is a "virtual" **composite** sensor that combines readings from other sensors to produce useful results. That is, it reads acceleration from the accelerometer, and gravity from the gyroscope (or magnetometer), and then synthesizes that into acceleration measures without gravity.
- [[Demo]]
- There are lots of these compound sensors; and they are very useful for simplifying our code!


### Rotation
Acceleration is good and well, but that only helps detect motion when the phone is _moving_. If I tilt the phone to one side, it will measure that movement... but then the acceleration goes back to 0 since the phone has stopped moving. What if we want to detect something like the **tilt** of the device?
- The **gravity sensor** (`TYPE_GRAVITY`) can give this information indirectly, but it is a bit hard to parse out.

A better option is to use the [`Rotation Vector Sensor`](https://source.android.com/devices/sensors/sensor-types.html#rotation_vector). This is another _composite_ (virtual) sensor that is used to determine the current rotation (angle) of the device.
- It utilizes the accelerometer, magnetometer, and gyroscope as best as it can.

Once again, we get back three values from a sensed event... but these values are in [quaternions](https://en.wikipedia.org/wiki/Quaternion) which is a lovely but complex (literally: it uses complex/imaginary numbers) way of measuring rotations. So we'd like to convert that into something we understand... like degrees of [roll, pitch, and yaw](https://en.wikipedia.org/wiki/Aircraft_principal_axes).
- To do this, we have to be a little bit round-about again, and look at a few interesting concepts along the way.

In computer systems, rotations are almost always stored as [**matrices**](https://en.wikipedia.org/wiki/Matrix_(mathematics)) (a mathematical structure that looks like a table of numbers). Matrices can be multiplied by [**vectors**](https://en.wikipedia.org/wiki/Vector_(mathematics_and_physics)) (or points) to produce a _new_ vector---and a [**rotation matrix**](https://en.wikipedia.org/wiki/Rotation_matrix) is a special set of numbers that when multiplied by a vector, produce a _new_ vector that is the first one rotated by some angle
  - Example: deriving 2D rotation matrix! (see slides) (because I'm mean :p)
  - You can actually use matrices to represent _any_ transformation (including movement, skewing, scaling, etc)... and these transformations can be specified for things like animation. 1/3 of Computer Graphics is understanding these transformations.
- Luckily, we don't actually need to know any of this math (just the _concept_), as Android has a method that will automatically produce a rotation matrix for us: `SensorManager.getRotationMatrixFromVector(targetMatrix, vector)`
  - This takes in a `float[16]`, representing a 4x4 matrix (why 4x4? 3 for each axis x,y,z plus 1 to track the origin. This is for [homogenous coordinates](https://en.wikipedia.org/wiki/Homogeneous_coordinates)), which it will fill with the correct values.
  - It doesn't actually produce a `new` array, because these kinds of methods want to as cheap as possible and allocating memory is expensive.
- A 4x4 matrix has not made things better... but now that we have that matrix, we can convert it into a set of rotations: <a href="http://developer.android.com/reference/android/hardware/SensorManager.html#getOrientation(float[], float[])">`getOrientation(matrix, targetArray)`</a>. This will take in the matrix and a `float[3]` to hold the results.
  - And we can convert them to degrees to have numbers that we're used to working with:
  ```java
  String.format("%.3f",Math.toDegrees(orientation[0]))+"\u00B0"
  ```

Let's talk a little bit about the [coordinate system](http://developer.android.com/guide/topics/sensors/sensors_overview.html#sensors-coords) we're using here. Sensors use a pretty standard 3D coordinate system, with the x and y axis matching what you expect on the screen, and the Z axis coming _out_ of the front of the device (following the [right-hand rule](https://en.wikipedia.org/wiki/Right-hand_rule) like a good coordinate system!)

- Note that the values of `getOrientation()` aren't in x,y,z order, and in fact rotate around -X and -Z. We can confirm this behavior by looking at the numbers.

- The other trick is that the coordinate system is based on the _device's_ **frame of reference**, not on the Activity or configuration! That means when you rotate the device (e.g., into landscape mode), the coordinate system doesn't change, and you need to adjust your values accordingly.
  - One solution is to use the `SensorManager#remapCoordinateSystem()` method, which will let you specify which axes should be transformed into which other axes (e.g., you specify the new X axis and the new Y axis, and it will change the rotation matrix to fit).
  - You can also get the device's current orientation with `Display#getRotation()`:
    ```java
    Display display = ((WindowManager) getSystemService(Context.WINDOW_SERVICE)).getDefaultDisplay();
    display.getRotation();
    ```
- For your game projects, it's okay to just have it work in one orientation: games and graphical systems are often restricted to a single view (since that's how the camera is setup).
  - But your final project will need to support both portrait and landscape modes!

The `ROTATION_VECTOR` sensor works well, but another potential option in API 19 or higher is the [`GAME_ROTATION_VECTOR`](https://source.android.com/devices/sensors/sensor-types.html#game_rotation_vector) sensor. This **compound** sensor is almost exactly the same as the `ROTATION_VECTOR` sensor, but it does _not_ use the magnetometer so is not influenced by magnetic fields. This means it doesn't give rotation around North, but some other starting angle (determined by the gyroscope). This can be useful in a game, as it may give more accurate results.
- We can easily swap this in, without changing most of our code (though we can use an `if` statement and fall back if on older APIs).

So we can pretty easily get the tilt angles for the phone (or gravity for the phone)... meaning it isn't too hard to make it so the ball "rolls" to one side as we tilt it, right?
- And that is left as a potential exercise for your homework!

Any questions?


#### Lecture References
- [Project Description](https://canvas.uw.edu/courses/1041406/pages/project)
- [Gesture design docs](https://www.google.com/design/spec/patterns/gestures.html#gestures-drag-swipe-or-fling-details)
- [`GestureDetector`](http://developer.android.com/training/gestures/detector.html#detect)
- [Notification Service](https://github.com/info498b-s16/04-20-notifications/blob/master/app/src/main/java/edu/uw/notsetdemo/MainActivity.java#L136)
- [`SensorManager`](http://developer.android.com/reference/android/hardware/SensorManager.html)
- [`Sensor`](http://developer.android.com/reference/android/hardware/Sensor.html)
- [Sensor Types](https://source.android.com/devices/sensors/sensor-types.html) (note this is from source.android.com!)
- [Using Accelerometer](http://developer.android.com/guide/topics/sensors/sensors_motion.html#sensors-motion-accel)
- [Rotation Vector Sensor](https://source.android.com/devices/sensors/sensor-types.html#rotation_vector)
- [Coordinate system](http://developer.android.com/guide/topics/sensors/sensors_overview.html#sensors-coords)
