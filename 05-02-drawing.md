## Drawing and Animation ("visual motion")

### Admin
- Homework check-in
  - Any issues with the GeoPainter to address?
  - Just one more assignment; is online. Very open-ended, and more pair-work. Basically an application of drawing, gestures, and motion (what we'll talk about this week).
    - These are examples of "features" you can include in Android apps (not required for _all_ apps), but it's the stuff that I think makes them novel and unique and so worth considering!

#### Another reminder about the PROJECT!
- Project details are, and have been, [online](https://canvas.uw.edu/courses/1041406/pages/project)
  - This is a ___group___ project, and you're allowed to choose your own groups (unless you'd rather me randomly assign?). I hope you've talking to others and doing some organizing. On **Wednesday** we'll take 10 minutes to check in and figure out who is in what groups; anyone left will be randomly organized.
- Fork & clone repo for today, as always. Actually has starter code we'll go over!
  - We'll be doing a lot of "play-developing", so holler out any questions, experiments, etc. that we should try out!

### Drawing
As I've said, at this point we're focusing on the special capabilities (interfaces, sensors) of mobile devices. We looked at _location_ last week, and this week we'll be focusing on gestures and motion sensors (i.e., "move your hand around" functions).

But in order to really show off this stuff, we're going to take a side-trip and talk about how we can introduce "visual motion" into Android. Specifically, we'll look at my favorite topic in programming: drawing pretty pictures! That's right: graphics and media!

- Have you drawn images using the HTML5 [Canvas](http://www.w3schools.com/html/html5_canvas.asp) or in Java using the [`Graphics2D`](https://docs.oracle.com/javase/8/docs/api/java/awt/Graphics2D.html) library? Well in Android it's really similar!

In fact, Android provides us a class called [`Canvas`](http://developer.android.com/reference/android/graphics/Canvas.html) that does a lot of the same work as the HTML5 Canvas or `Graphics2D`: it represents a "drawable" context. We can call methods on it to draw rectangle, circles, and even images (represented by `Bitmaps`), and those drawings can be shown on the screen.

#### Custom Views
In order to draw stuff, we need to have a `View` to draw on. The easiest way to get ourselves a `View` we can _programmatically_ draw on is to create our own **custom View**, which we can specify as having a "drawn" appearance. We can make our own special `View` by subclassing it (remember: `Buttons` and `EditTexts` are just subclasses of `View`!), and then filling in a method (`onDraw()`) that specifies how that `View` is rendered on the screen.
- _Render_: generating an image and putting it on the screen.

Customizing a `View` isn't too hard, but to move things along I've provided one: `DrawingView`. Instead of typing out the code, we'll try just walking through what exists (let me know if this is more or less helpful).
- We `extend View` to subclass
- `View` has a pile of constructors; we'll override them all (though they end up calling the last that actually does setup)
- In here we also set up a [`Paint`](http://developer.android.com/reference/android/graphics/Paint.html), which is a set of information about how we want to draw: color, stroke width, font-size, anti-aliasing options, etc. We'll mostly use them for color.
- We can then start overriding methods. There are a few we're going to focus on:
  - `onSizeChanged()` will get called when the `View` changes size: this is on inflation (which occurs as part of `onCreate`, so on rotation). It can act a little bit like `onResume`.  
  - `onMeasure()` is recommended to be overwritten in the [docs](http://developer.android.com/guide/topics/ui/custom-components.html#custom), but we're going to skip that for time and space (and since our `View` is going to take up the entire screen)
    - This is basically where we put how our View would size itself to `wrap_content`. It's more important if we want to make something like a custom button.
  - `onDraw()` is where the magic happens: this method gets called whenever the `View` needs to be displayed (like `paintComponent()` in Swing). This callback is handed a `Canvas` object that we can draw on!
    - **We never call `onDraw()`; the Android UI system calls it!!**

The `Canvas` can be drawn on in a couple of ways:
- We can call methods like `drawColor()` (to fill the background), `drawCircle()`, or `drawText()` to draw specific shapes/etc. on it.
- We can also draw a [`Bitmap`](http://developer.android.com/reference/android/graphics/Bitmap.html), which represents a graphics raster (e.g., a 2D array of pixels). If we have a `Bitmap`, we can set the colors of individual pixels (using `setPixel(x,y,color)`), and then draw the `Bitmap` onto the Canvas (thereby double-buffering!). This is useful for pixel-level drawing
  - If you've used MS Paint, it's the difference between the shape drawing and the "zoomed in" drawing

Note that we cause the `Canvas` to be "redrawn" (so our `onDraw()` method to be called) by calling `invalidate()` on the `View`: this causes Android to need to recreate it, thereby redrawing it. By repeatedly calling `invalidate()` we can do something approximating animation!

Let's talk a bit about **Animation** (well, we'll actually talk about it a lot today). Animation is the process of imparting life (from latin ___"anima"___). We tend to mean giving _motion_&mdash;having images change over time
- Animation (and video in general) involves showing a sequence of images over time. If the images go fast enough, then the human brain interprets them as being part of the same successive motion.
- Each image is called a "frame". Film tends to be 20-24 frames per second (fps), video is 29.97fps, and video games aim at 60fps.
- Thus we can achieve animation by `invalidating()` and redrawing over and over, more than 16 times per second. Like with a loop!
- To do this, we'll call `invalidate()` at the end of `onDraw()` to cause a _recursive_ loop (because we need that method to be called over and over, but **we don't call it**)
- Hitting that 16fps can actually be pretty difficult, since determining _what_ to draw is computationally expensive! If we're calculating every pixel on a 600x800 display, that's half a million pixels we have to calculate! At 60fps, that's 28 million pixels per second. For scale, a 1Ghz processor can do 1 billion operations per second---so if each pixel requires 5 operations, we're at 15% of our processor.
  - And that's part of why we use the GPU; it does massive parallelization for this!

Demo: make the ball slide off screen: `ball.cy += ball.dx;`
- Can also add in collisions (see below)

Note that Android does have other systems for doing animation as well; we'll look at those more shortly.

#### SurfaceViews
Since all this calculation (at pixel-level detail) can take some time, we want to be careful it doesn't block the UI thread! We'd like to instead do the drawing in the background.
- `AsyncTask` won't work, because we want to do this repeatedly!
- Additionally, the UI Thread is called such because _that's where UI work goes!_ And guess what: rendering a `View` is UI work. So we need to do the drawing logic on the background thread, but need to show it on the screen from the UI thread.

Luckily, Android gives us a class that is specially designed for being drawn on in a background thread: the [`SurfaceView`](http://developer.android.com/reference/android/view/SurfaceView.html). Unlike normal `Views` that are somewhat ephemeral, a `SurfaceView` is a dedicated drawing surface that we can interact with in a separate thread. It's basically designed to be drawn on from a separate thread without requiring us to do a _ton_ of synchronization work.

These also take some work to setup, so let's again look at one that is provided!
- We extend `SurfaceView` and _implement_ `SurfaceHolder.Callback`. A `SurfaceHolder` is an object that "holds" (contains) the underlying drawable surface. We interact with the `SurfaceView` through the holder to make sure that we're _thread-safe_: that only one thread is interacting with the surface at a time.
  - In general there will be two threads trading off use of the holder: our background thread that is drawing on the surface, and then UI thread that is showing the surface to the user.
- We register the holder in the constructor with `getHolder()`, and register ourselves for callbacks when it changes. We also instantiate a new `Runnable`, which is an object that can be "run" in a separate thread (e.g., an object with a `run()` method that we can execute).
  - Recall the `Runnable` from the second week's lab?
- Our `SurfaceHolder.Callback` interface requires methods about when the surface changes, and so we fill those in.
  - `onSurfaceCreated()` starts our background thread (because created)
  - `onSurfaceChanged()` ends up acting a lot like `onSizeChanged()` from the other `View`
  - `onSurfaceDestroyed()` stops the background thread in a "safe" way (code from Google)
- If we look at the `Runnable`, it's basically an infinite loop:
  1. Grab the Surface's Canvas, locking it so only used in this thread
  2. Draw on it
  3. Then "push" the Canvas back out to the rest of the world, basically saying "we're done drawing, you can show it to the user now"
- Overall, this process will cause us to "redraw()" as fast as possible, all without blocking the UI thread! Great for animation, which can be controlled and timed in the `update()` helper method (e.g., only `update()` variables at a particular rate)

And that gives us a drawable surface that we can interact with in the same way.

Demo: Make the ball bounce around!
- This is partly to demonstrate how we can create game and animation logic just using basic Java work; no game engines required (though those exist as well).

```java
if(ball.cx + ball.radius > viewWidth) { //left bound
    ball.cx = viewWidth - ball.radius;
    ball.dx *= -1;
}
else if(ball.cx - ball.radius < 0) { //right bound
    ball.cx = ball.radius;
    ball.dx *= -1;
}
else if(ball.cy + ball.radius > viewHeight) { //bottom bound
    ball.cy = viewHeight - ball.radius;
    ball.dy *= -1;
}
else if(ball.cy - ball.radius < 0) { //top bound
    ball.cy = ball.radius;
    ball.dy *= -1;
}
```

### Basic Gestures
We've got some animation and movement, but it would be nice to include interaction. Our `View` takes up the entire screen so we don't want to add buttons, but there are other options available.

In particular, we can add [Touch Gestures](http://developer.android.com/training/gestures/index.html). Touch screens are a huge part of Android devices (and mobile devices in general, especially since the first iPhone) that are incredibly familiar to most users. We've already indirectly used the touch interface, with how we've had users click on buttons (theoretically using the touch screen). But here we're interested in more than just button clicks, which really could come from anywhere: instead, how we can react to _where_ the user might touch the screen and even the different ways the user might _caress_ the screen: flings, drags, multi-touch, etc.

Android devices automatically detect various touching interactions (it's how buttons work); we can respond to these _events_ by overriding the `onTouchEvent()` callback, which is executed whenever there is something happening involving the touch screen
- We can log out the event to see the kind of details we get!

There are _lots_ of things that can cause `TouchEvents`, so much of our work involves trying to determine what semantic "gesture" the user made. Luckily, Android provides a number of utility methods and classes to help with this.

The most basic is `MotionEventCompat.getActionMasked(event)`, which extracts the "type" of the event from the motion that was recorded:

```java
int action = MotionEventCompat.getActionMasked(event);
float x = event.getX();
float y = event.getY() - getSupportActionBar().getHeight(); //closer to center...
switch(action) {
  case (MotionEvent.ACTION_DOWN) : //put finger down
    view.ball.cx = x;
    view.ball.cy = y;
    //or
    view.ball.dx = (x - view.ball.cx)/Math.abs(x - view.ball.cx)*30;
    view.ball.dy = (y - view.ball.cy)/Math.abs(y - view.ball.cy)*30;
    return true;
  case (MotionEvent.ACTION_MOVE) : //move finger
    view.ball.cx = x;
    view.ball.cy = y;
    return true;
  case (MotionEvent.ACTION_UP) : //lift finger up
  case (MotionEvent.ACTION_CANCEL) : //aborted gesture
  case (MotionEvent.ACTION_OUTSIDE) : //outside bounds
  default :
    return super.onTouchEvent(event);
}
```

This lets us react to basic touching. For example, we can make it so that taps (`ACTION_DOWN`) will teleport the ball to where we click! We can also use the `ACTION_MOVE` events to let us drag the ball around.
- Note that there are some other details you need to consider when doing drags with multi-touch: see [this guide](http://developer.android.com/training/gestures/scale.html#drag) for details, and we'll talk about this more tomorrow (or end of today if time).


#### Break?

### Property Animation
We've seen how we can create animations simply by adjusting the drawing we do on each frame. This is great for games or other complex animations if we want to have _a lot_ of control over our graphical layout... but sometimes we want to have some simpler, platform-specific effects (that run smoother!) Android actually involves a number of different animation systems that can be used within and _across_ Views:

[Property Animation](http://developer.android.com/guide/topics/graphics/prop-animation.html) is a general animation framework, that basically performs animated _linear interpolation_ on whatever values you give it to change.
- We'll go over this in a second

[Scene Transitions](http://developer.android.com/training/transitions/index.html) are a framework for transitioning between layouts; the material animations use this indirectly. See also [Animations](http://developer.android.com/training/animation/index.html)
- These are useful and able to be added to really any application... but are "extra flair" rather than being core to something like games, so I'll leave you to look up these systems on your own.
- [Material Animations](http://developer.android.com/training/material/animations.html) include animations built into the "material design" style (found in Lollipop and later), and include various effects and transitions in response to user interaction (e.g., ripples, sliding fragments, etc).

[OpenGL Animations](http://developer.android.com/guide/topics/graphics/opengl.html) are available for doing 3D animated systems.

...and there might be a few others out there as well. A good place to start if you're interested in adding animations and transitions to your app is the [Material Design Guide](http://developer.android.com/design/index.html), particularly the [animation section](https://www.google.com/design/spec/animation/authentic-motion.html). This has a lot of _design_ guidelines to make sure you're actually adding _useful_ animations.

We're going to focus on **Property Animation** as an example---in part since you can potentially use it for your homework. It also requires less setup. [Property Animation](http://developer.android.com/guide/topics/graphics/prop-animation.html) is an animation system where you specify a start state, an end state, and a duration, and the Android systems changes from the start to the end over that length of time--thereby producing animation!
- This is calculated using a concept called [interpolation](https://en.wikipedia.org/wiki/Interpolation); most usually a variation of **linear interpolation**.
  - Anyone done linear algebra? Fun aside! Basically, we figure out how far along we are in the animation, and then take that a _weighted average_ of the start and end values. We can get _non-linear interpolation_ by adjusting the weights so they are non-linear (e.g., you need to get 70% across to have 50% of the starting value).

The main engine for doing this kind of interpolated animation in Android is the [`ValueAnimator`](http://developer.android.com/guide/topics/graphics/prop-animation.html#value-animator) class. This class lets you specify the start, end, and duration, and then can be run to calculate all the values in between. It has a number of static methods (e.g., `.ofInt`, `.ofFloat`, `.ofArgb`) which creates "animators" for interpolating different _values_. For example:
```java
ValueAnimator animation = ValueAnimator.ofFloat(0f, 1f);
animation.setDuration(1000);
animation.start();
```
- Of course, just running this doesn't do anything we can see, since it's change the numbers but those don't relate to anything! We can get the values by setting a [listener](http://developer.android.com/guide/topics/graphics/prop-animation.html#listeners) and overriding a callback we're interested in (e.g., `onAnimationUpdate()` from `ValueAnimator.AnimatorUpdateListener`).

But more commonly, we want to have our animation change the _property_ of some object--for example, the color of a view, the position of an object (like a ball), etc. We can do this easily using the [`ObjectAnimator`](http://developer.android.com/guide/topics/graphics/prop-animation.html#object-animator) subclass. Basically, this subclass runs an animation like the `ValueAnimator`, but has the built-in functionality to change the property of an object on each step.
- It does this by calling a **setter** for that property---thus the object needs to have a property setter for us to animate it! (If not, we can make a wrapper or just develop our on `ValueAnimator`).
```java
//change the "alpha" property of foo (e.g., call foo.setAlpha())
ObjectAnimator anim = ObjectAnimator.ofFloat(foo, "alpha", 0f, 1f);
anim.setDuration(1000);
anim.start();
```

We can use this to (say) change the ball's radius or position using an interpolated animation. Other options:
- Make a _getter_ and only pass the Animator an ending value if we want (so "from current to end")
- Use `.setRepeatCount(INFINITE)` and `.setRepeatMode(REVERSE)` to cause it to repeat!
- Note that we're changing the property, and because our `SurfaceView` is refreshing all the time, we get to see the updates in action :)

If we want to include multiple animations in sequence, we can use an [`AnimatorSet`](http://developer.android.com/guide/topics/graphics/prop-animation.html#choreography), which gives us methods about the ordering:
```java
//example from docs
ObjectAnimator animX = ObjectAnimator.ofFloat(obj, "x", 50f);
ObjectAnimator animY = ObjectAnimator.ofFloat(obj, "y", 100f);
AnimatorSet animSetXY = new AnimatorSet();
animSetXY.playTogether(animX, animY);
animSetXY.start();
```

These can get complicated, and we may want to reuse them... so we can also defining them in XML as [resources](http://developer.android.com/guide/topics/graphics/prop-animation.html#declaring-xml). This also means that we can have different configurations use different animations (perhaps things move faster on a larger phone?).
- These are put into the `/res/animator` directory (**not** the `/res/anim/` folder, which is for View Animations). See [Animation Resources](http://developer.android.com/guide/topics/resources/animation-resource.html#Property) for full XML details
  ```xml
  <set android:ordering="together"> <!-- together is default -->

    <objectAnimator
             android:propertyName="x"
             android:duration="500"
             android:valueTo="400"
             android:valueType="intType"/>
    <!-- ... -->
  </set>
  ```
  Note that you'll need to **inflate** the Animator resource, just like we did with layouts:
  ```java
  ObjectAnimator anim = (ObjectAnimator) AnimatorInflater.loadAnimator(myContext, R.anim.animator);
  anim.setTarget(myObject);
  anim.start();
  ```

Note that we can also use this same framework to animate changes to `Views`: buttons, images, etc. Views are objects and have properties (along with getters and setters we can use)... so we can use just an `ObjectAnimator`! See [the docs](http://developer.android.com/guide/topics/graphics/prop-animation.html#views) for a list of properties we can change (includes `x, y, rotation, alpha, scaleX, scaleY`, etc.)
- Or to make this even simpler, Views provide a `ViewPropertyAnimator` which modifies multiple properties together, but does so in a much more efficient manner. We can access this with the `.animate()` method on a `View`, then call relevant methods that we want to change:
```java
myView.animate().x(100f).y(300f);
```
(But really, if you want to animate layout changes on a modern device, you should use [transitions](http://developer.android.com/training/transitions/index.html)), which makes this a lot easier (particularly for "normal" transitions)

There are [lots more ways](http://developer.android.com/guide/topics/graphics/prop-animation.html) to customize exactly what you want your animation to be, but this should give you a lot of the basics! You can also look at [official demos](https://android.googlesource.com/platform/development/+/master/samples/ApiDemos/src/com/example/android/apis/animation) for more examples.


#### Fling (if time, else tomorrow)
So far we've just been using very simple touch gestures: press and move. But we can also detect and react to more complex gestures: long presses, double-taps, or flings (a "flick" or swipe on the screen. See [ here](https://www.google.com/design/spec/patterns/gestures.html#gestures-drag-swipe-or-fling-details) for some design language involving these gestures). Android provides a [GestureDetector](http://developer.android.com/training/gestures/detector.html#detect) class that can help identify these actions.

The easiest way to use this---particularly when interested in a particular gesture (like fling)---is to _extend_ `GestureDetector.SimpleOnGestureListener`. We can then override the gestures we're interested in responding to: e.g., `onFling()`.
- We will also need to override `onDown()` and have it return `true` indicating whether we've "consumed" (handled) the event or not. . This is because all gestures begin with a `DOWN` action, and if we return `true` with whether we've "consumed" (handled) the event or not. If we haven't handled it, then it gets passed up the hierarchy to other elements, like the Notification drawer. Same thing we've done for other event handlers.

We can instantiate this class with `mDetector = new GestureDetectorCompat(this, new MyGestureListener());`
- Then in our `onTouchEvent()` method, we can pass the event into our Gesture Detector to process, and if it doesn't see anything continue on it ourselves:
  ```java
  boolean gesture = this.mDetector.onTouchEvent(event); //check gestures first!
  if(gesture){
      return true;
  }
  ```
We can now add the ability to "fling" the ball, say by taking the fling velocity and assigning that to the ball (maybe slowing it down by 1% on update). Scaling velocity down to 3% tends to work okay. Also note we need to negate the velocities since they are registered as "backwards" from the coordinates we expect.

Wednesday we'll also look at multi-touch and how we can handle the most complex of touch gestures.

#### Lecture References
- [Project Description](https://canvas.uw.edu/courses/1041406/pages/project)
- [`Canvas`](http://developer.android.com/reference/android/graphics/Canvas.html)
- [`Bitmap`](http://developer.android.com/reference/android/graphics/Bitmap.html)
- [Property Animation](http://developer.android.com/guide/topics/graphics/prop-animation.html)
- [Scene Transitions](http://developer.android.com/training/transitions/index.html)
- [Material Animations](http://developer.android.com/training/material/animations.html)
- [OpenGL Animations](http://developer.android.com/guide/topics/graphics/opengl.html)
- [`ValueAnimator`](http://developer.android.com/guide/topics/graphics/prop-animation.html#value-animator)
- [`ObjectAnimator`](http://developer.android.com/guide/topics/graphics/prop-animation.html#object-animator)
- [Animation Resources](http://developer.android.com/guide/topics/resources/animation-resource.html#Property)
