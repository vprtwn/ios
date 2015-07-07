### iOS Touch and Graphics Pipeline
* Display frame = display interval = display segment
* Many processes triggered by display refresh

1. Multi-touch
    * can take up to the entire display frame
2. Your app responds to touch input
    * can take up to one display frame
3. Core Animation creates GPU instructions & GPU renders frame
4. Rendered on screen for one frame

* CoreAnimation batches changes to your views
* Can optimize amount of time spent in CoreAnimation
* CoreAnimation has a slower mode & faster mode
    * can split core animation & GPU work over two display frames
    * no manual way to control which mode you're in
* Faster mode = Double Buffering
* Slower mode = Triple Buffering
    * 5 frames of latency

* What about Metal and OpenGL?
    * CoreAnimation still acts as an arbiter for Metal
    * still 4 frames of latency

### Low Latency Core Animation
* New in iOS9
* Happens automatically
* Disabled for animations (CAAnimations, UIView animations)
    * for better performance, disable animations while touches are on the display

### Touch Coalescing
* iPad Air 2 has 120Hz touch scan
    * `coalescedTouchesForTouch`
    * coalesces touches in each display frame
* Coalesced touches are unique `UITouch` objects
* Main touches have a share identity

### Touch Prediction
* `UIEvent.predictedTouchesForTouch(touch: UITouch)`
* iOS guesses path of touch

### Fine Tuning your Application
* Only render content that will actually make it on screen
* Use time profiler to show CPU time (can change sampling rate to match display frame)
* Use GPS gauge in Xcode as high-level overview
* (new) GPU Driver instrument
    * shows how long GPU is active
    * if app is spending more time in CoreAnimation than GPU, you'll see 3 colors (triple buffering)



