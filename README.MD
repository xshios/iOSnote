# Lecture 1

# MVC

* Model: What your application
    * data
* View: Controller's minions？？？
    * Views do not own the data
* Controller: How your model is presented
    * 
## Communication Pattern
1. Model <---- Controller ---> View
2. Model ----> Controller through a radio station model
    * *NotificationCenter*
    * *KVO*
3. Controller <--- View
    * target <--- action
    * delegate <--- `XXshould`, `XXwill`, `XXdid` via *protocol*
    *  dataSource <- `data`, `at`, `count` via *protocol*
4. Controller <---> Controller

# demo - Concentration
## Model
* `class` vs `struct`
    * `class` is inheritable 
    * `struct` is a value-copy object.

*A `class` is a container of `struct`*

## View
normally it can be defined by mainboard interactively.

## Controller
* define the action on the UI object and 
* how the data of model changes using the method defined in the model according to user's actions

# Swift Programming Language
## stack

only horizontally or vertically 

## properties
computational properties: get and set

## access control
* `internal` - default, usable by any object in app
* `private` - callable inside
* `private(set)` read-only outside
* `fileprivate` in the source file
* `public` for frameworks only. can be used by objects outside current framework
* `open` for frameworks only .public and objects outside current framework can subclass this

Rules in `Controller`
1. things have something to do with UI should be `private`
2. properties that have useful information to users (other class) or tester should not be `private` or be `private(set)`

## assertion

## extensions
add methods/properties to extend existing data structures `class/struct/enum` even without the source
* `extension Collection` works on all `Collection` type like `String` `list`..




## enum

Associated Data `enum`

## Closures
`seq.filter {$0>0}`
## `as?`
downcast from superclass to subclass

## class
* `NSObject`: root class for any classes in Swift
* `NSNumber`: generic number-holding class, including `boolValue`
* `Date` find the date and time. Also, `Calendar`, `DateFormatter`,`DateComponents`, localization ramifications
* `Data` a "bag" of bits save/restore/transit data.



# Views

a rectangular area:
1. defines a coordinate space
2. for drawing
3. for handling touch events

## Hierarchical
1. A view has only one superview
2. but it can have many/zero subviews
3. the later subview is on top of the ealier subviews
4. A view can clip its subview to its own bound (default not to)
**The hiarachy is most often constructed in Xcode graphically**

But it can be done in code as well
```swift
func addSubview(_ view: UIView)
func removeFromSuperview()
```
## Where does the view hierarchy start?
1. `var view: UIView` is the top of the (useable) view hierarchy - an ancestor for all of your UIViews
2. It's the one whose bounds will change on rotation
3. It's likely the one you will programmatically add subviews to
4. It's automatically hooked up for you in Xcode

## UIWindow
1. The very top of the hierarchy (includes status bar). 
2. only one UIWindow in an entire iOS application


## Initializing a UIView
As always, try to avoid an initializer if possible

```swift
override init(frame: CGRect) // if UIView is created in code
{
    super.init(frame: frame)
    //setup()
}
required init?(coder aDecoder: NSCoder) // if UIView comes out of a storyboard
{
    super.init(coder: aDecoder)
    //setup() 
}
```

you need to implement them both if you need an initializer.

* Another alternative to initializer in UIView
```swift
awakeFromNib() // only if the UIView came out of a storyboard
```
This is not an initializer. Order is not guaranteed, so you cannot message from any other objects in the storyboard here.

## Important types for Coordinate System

* `CGFloat`, fundamental floating value type
* `CGPoint`, a struct with two CGFloats in it: x and y
* `CGSize` a struct with two CGFloats in it: width and height
* `CGRect` A struct with a `CGPoint` and a `CGSize` in it 

```swift

let cgf = CGFloat(5.2)

var point = CGPoint(x: 37.0, y:55.2)
point.y += 20.0

var size = CGSize(width: 100.0, height: 50.0)
size.width += 42.5
//...
// CGRect!
struct CGRect {
    var origin: CGPoint
    var size: CGSize
}

// convenient properties and functions
var minx: CGFloat
var midY: CGFloat
intersects(CGRect) -> Bool
intersect(CGRect) -> CGRect
contains(CGPoint) -> Bool
```

## View Coordinate System

* Origin is upper left
* Units are points, not pixels
    * Pixels are the minimum-sized unit of drawing your device is capable of
    * points are  the units in the coordinate system
    * how many pixels per point are there? 

        `var contentScaleFactor: CGFloat`
* The boundaries of where drawing happens 
        
        `var bounds: CGRect`
    * each view has there own coordinate system!
* where is the UIView

```swift
var center: CGPoint // in its superview's coordinate system
var frame: CGRect   // in its superview's coordinate system
```
* use `frame` or `center` to position
* use `bounds` to draw

## Creating Views
* via storyboard
    * use *Identity Inspector* to changes its class to your subclass
* via code
```swift
let newView = UIView(frame: myViewFrame)
// or 
let newView = UIView() // frame will be CGRect.zero
```

## Draw
the only way to draw through
```swift
override func draw(_ rect: CGRect)
```
* the `rect` is purely an optimzation
* it's our UIView's `bounds` that describe the entire drawing area (the `rect` is a subarea)

**Never call** `draw(CGRect)`, rather use
```swift
setNeedsDisplay() // redrawn
setNeedsDisplay(_ rect: CGRect) // the area that needs to be redrawn
```

1. get a context to draw into `UIGraphicsGetCurrentContext()`
2. Create paths `UIBezierPath()`
3. set drawing attributes like colors, fonts, textures, linewidths, linecaps,...
4. stroke or fill the aboive-created paths with the given attributes

### defining a path in `draw(CGRect)`
1. create a UIBezierPath
2. move around, add lines or arcs to the path
3. close the path (optional)
4. set attributes and stroke/fill
```swift
// 1
let path = UIBezierPath()

// 2 
path.move(to: CGPoint(80,50))
path.addLine(to: CGPoint(140, 150))
path.addLine(to: CGPoint(10,150))

// 3
path.close()

// 4
UIColor.green.setFill()
UIColor.red.setStroke()
path.linewidth = 3.0
path.fill()
path.stroke()
```
Other things you can do:
* draw common shapes

```swift
let roundedRect = UIBezierPath(roundedRect: CGRect, cornerRadius: CGFloat)
let oval = UIBezierPath(ovalIn: CGRect)
// ...
```
* clip drawing to a UIBezierPath's path
```swift
addClip()
```
* hit detection. The path must be closed. The winding rule can be set with `usesEvenOddFillRule` property
```swift
func contains(_ point: CGPoint) -> Bool
```
## UIColor
* use `UIColor` e.g. `let green = UIColor.green`.
* Or create them from RGB, HSB, or even a pattern (using `UIImage`)
* background color of a UIView
* colors can have alpha (transparency)
```swift
let semitransparentYellow = UIColor.yellow.withAlphaComponent(0.5)
```
* If you want to draw in your view with transparency, you **must** let the system know by setting the `UIView`
```swift
var opaque = false
```
* make your entire UIView using its `alpha` property

## Layerss
the draw mechanism underneath UIView

```swift
var layer: CALayer
```
The CA in `CALayer` stands for "Core Animation"

But CALayer can do some cool non-animation oriented things as well, for example
```swift
var cornerRadius: CGFloat // make the background a rounded rect
var borderWidth: CGFloat // draw a border around the view
var borderColor: CGColor? // the color of the border
```

it uses `CGColor`. You can use UIColor's `cgColor` property to transform.

### View transparency
in the order of `subviews` list.

* completely hide a view without removing it from hierarchy
```swift
var isHidden: Bool
```

### Drawing Text
usually we use a `UILabel`

* To draw in `draw(CGRect)`, use `NSAttributedString`

```swift
let text = NSAttributedString(string: "hello")
text.draw(at: aCGPoint)
let textSize: CGSize = text.size
```
* `NSRange` access a range of characters in an `NSAttributedString`
    *  it can handle the `String` and `NSString` weirdness

### Fonts
usually you set fonts in UI elements like UIButton, UILabel,...

* simply way to get a font in code

```swift
// system preferred font
static func preferredFont(forTextStyle: UIFontTextStyle) -> UIFont

// some of the style
UIFontTextStyle.headline
                .body
                .footnote


// automatically adjust size according to system setting i.e. Accessibility
```

* More advanced way
```swift
let font = UIFont(name: "Helvetica", size: 36.0)

// To enable auto-sizing

let metrics = UIFontMetrics(forTextStyle: .body)
let fontToUse = metrics.scaledFont(for: font)
```

* there are also "system fonts"

### Drawing Images
* `UIImageView`
* Or, in `draw(CGRect)` create a `UIImage` object

```swift
let image: UIImage? = UIImage(named: "foo")
```
* Or create one from files in the file system

```swift
let image: UIImage? = UIImage(contentsOfFile: pathString)
let image: UIImage? = UIImage(data: aData) // raw jpg, png, tiff,...
```
* Or, create one by drawing with Core Graphics
    * see documentation for `UIGraphicsBeginImageContext(CGSize)`


Once you have a UIImage, you can blast its bits on screen via
```swift
image.draw(at point: aCGPoint) // upper left conner put at aCGPoint
image.draw(in rect: aCGRect)    // scales the image to fit aCGRect
image.drawAsPattern(in rect: aCGRect) // tiles the image into a CGRect
```

### Redraw on bounds change?
By default, when a UIView's bounds changes, there is no redraw. Instead, the "bits" of the existing image are scaled to the new bounds size.

* This is often not what you want. 
* Use `UIViewContentMode`
    
    1. dont' scale
    2. scale
    3. redraw by calling draw(CGRect)
```swift
// 1
.left/.right/.top/.topRight/.topLeft/.bottomRight/.bottomLeft/.center

// 2
.scaleToFill/.scaleAspectFill/.scaleAspectFit

//3 
.redraw
```

### layout on bounds changes?
usually you would set this up using *Autolayout constraints*

Or you can manually reposition your views when your bounds change by overriding
```swift
override func layoutSubviews(){
    super.layoutSubviews()
    // reposition
}
```

# Gesture
Gestures are recognized by instances of `UIGestureRecognizer`

There are two sides to using a gesture recognizer
1. adding a gesture recognizer to a UIView (asking the UIView to "recognize" that gesture)
    * this is usually done by a Controller.
2. Providing a method to "handle" that gesture (not necessarily handled by the UIView)
    * this is provided wither by the UIView or a Controller, depending on the situation

## Adding a gesture recognizer to a UIView
Usually use `didSet` in a `@IBOutlet weak var`
```swift
@IBOutlet weak var pannableView: UIView {
    didSet {
        let panGestureRecognizer = UIPanGestureRecognizer(
            target: self, action: #selector(ViewController.pan(recognizer:))
        )
        pannableView.addGestureRecognizer(panGestureRecognizer)
    }
}
```

## A Handler for a Gesture
A handler for a gesture needs gesture-specific information

* `UIPanGestureRecognizer`
    * `func translation(in: UIView?) -> CGPoint` cumulative since start of recognition
    * `func velocity(in: UIView?) -> CGPoint` how fast the finger is moving
    * `func setTranslation(CGPoint, in: UIView?)` reset the translation so far. Discrete translation

* abstract superclass also provides state information
```swift
var state: UIGestureRecognizerState { get }
```
* `state`
    * this sits around in `.possible` until recognition starts
    * from `.began` through repeated `.changed` to `.ended`
    * it can go to `.failed` or `.canceled`

* the `selector`

```swift
func pan(recognizer: UIPanGestureRecognizer){
    switch recognizer.state {
        case .changed: fallthrough
        case .ended:
            let translation = recognizer.translation(in: pannableView)
            recognizer.setTranslation(CGPoint.zero, in: pannableView)
        default: break
    }
}
```

* UIPinchGestureRecognizer: 
    * `.scale`, 
    * `.velocity {get}`
* UIRotationGestureRecognizer: 
    * `.rotation`, 
    * `.velocity {get}`
* UISwipeGestureRecognizer: 
    * `.direction: UISwipeGestureRecognizerDirection`, 
    * `numberOfTouchesRequired`
* UITapGestureRecognizer:
    * `numberOfTapsRequired`
    * `numberOfTouchesRequired`
* UILongPressRecognizer:
    * `minimumPressDuration: TimeInterval`
    * `numberOfTouchesRequired`
    * `allowableMovement: CGFloat`


# Multiple MVCs
iOS provides some Controllers whose View is "other MVCs"
* UITabBarController
* UISplitViewController
* UINavigationController

## UITabBarController
Property: 
```swift
var tabBarItem: UITabBarItem! 
```

## UISplitViewController
puts two MVCs side-by-side. It works on ipad and iphone plus...

* left: Master
* right: Detail

## UINavigationController
Pushes and pops MVCs off of a stack

the contents of the top area are determined by the underlying MVC

property:
```swift
var tabItem: UINavigationItem! 
//or
toolbarItem
// important pointer
rootViewController
```
## Accessing the sub-MVCs
* via the `viewControllers` property
    * for a tab bar, they are in order, left to right
    * for a split view, [0] is master and [1] is the detail
    * for a navigation controller, [0] is the root and the rest are in order on the stack
```swift
var viewControllers: [UIViewController]? {get set}
```

* via specific properties
```swift
var tabBarController: UITabBarController? {get}
var splitViewController: UISplitViewController? {get}
var navigationController: UINavigationController? {get}
```
for example, to get the detail of the split view controller
```swift
if let detail: UIViewController? = splitViewController?.viewControllers[1]//{...}
```

## Wiring up MVCs
* via storyboard, ctrl+drag..

## Segues
kinds of segues
* Show Segue (will push in a Navigation Controller, else Modal)
* Show Detail Segue (will show in Detail of a SplitView or will push in a Navigation Controller)
* Modal Segue (take over the entire screen while the MVC is up)
* Popover Segue (make the MVC appear in a little popover window)

**Segues always create a new instance of an MVC**

* ctrl+drag and add Identifier in the Storyboard Segue setting

* Identifier
    * need it to invoke this segue from code using this UIViewController method (but we almost never do this because we set usually ctrl-drag from the instigator)
    ```swift
    func performSegue(withIdentifier: String: String, sender: Any?)
    ```  
    * most important use of the identifier: `prepare` for a segue
        
        when a segue happens, the View Controller containing the instigator gets a chance to prepare the destination View Controller to be segued to.

### Preparing for a Segue
```swift
func prepare(for segue: UIStoryboardSegue, sender: Any?){
    if let identifier = segue.identifier{
        switch identifier {
            case "Show Graph":
                if let vc = segue.destination as? GraphController {
                    vc. property1 = //....
                    vc.callMethodToSetItUp()
                }
            default: break
        }
    }
}
```

### Preventing Segues
* prevent a segue from happening
    ```swift
    func shouldPerformSegue(withIdentifier identifier: String?, sender: Any?) -> Bool
    ```

# Timer
used to execute code periodically

1. Fire one off with this method
    ```swift
    class func scheduledTimer(
        withTimeInterval: TimeInterval,
        repeats: Bool,
        block: (Timer) -> Void
    ) -> Timer
    ```
    example:
    ```swift
    private weak var timer: Timer?
    timer = Timer.scheduledTimer(withTimeInterval: 2.0, repeats: true){
        timer in
        // your code
    }
    ```
    every 2 seconds, the closure will be executed.
2. stop a repeat timer
    ```swift
    timer.invalidate()
    ```
3. Tolerance
    ```swift
    myOneMinuteTimer.tolerance = 10 // in seconds
    ```


# Animation
## Kinds of Animation
* Animating UIView properties (frame/transparency/..)
* Animating Controller transitions 
* Core Animation (underlying powerful animation framework)
* OpenGL and Metal (3D)
* SpriteKit (2.5D animation)
* Dynamic Animation (Physics - based animation)

## UIView Animation
* changes to certain UIView properties can be animated over time
    ```swift
    frame
    center
    bounds  // transient size, does not conflict with animating center
    transform   // translation, rotation and scale
    alpha   // opacity
    backgroundColor
    ```
* Done with `UIViewPropertyAnimator` using closures
    ```swift
    class func runningPropertyAnimator{
        withDuration: TimeInterval,
        delay: TimeInterval,
        options: UIViewAnimationOptions,
        animations: () -> Void,
        completion: ((position: UIViewAnimatingPosition) -> Void)? = nil
    }
    ```
    **the output will be immediate but the presentation will be animated**

    example:
    ```swift
    if myView.alpha == 1.0 {
        UIViewPropertyAnimator.runningPropertyAnimator(
        withDuration: 3.0,
        delay: 2.0,
        options: [.allowUserInteraction],
        animations: { myView.alpha = 0.0 },
        completion: { if $0 == .end { myView.removeFromSuperview() } }
        )
        print("alpha = \(myView.alpha)")
    }
    ```
    the console will output `0.0` instantly while the alpha on the screen won't be  0 for 5 more seconds
### UIViewAnimationOptions
* state related
    ```swift
    beginFromCurrentState // pick up from other on-going animations of these properties
    allowUserInteraction // allow gestures while animation is in progress
    layoutSubviews // animate the relayout of subviews with a parent's animation
    repeat  // repeat indefinitely
    autoreverse // forwards, then retro
    overrideInheritedDuration   // if not set ,use duration of any in-progress animation
    overrideInheritedCurve  // if not set, use curve (e.g. ease-in/out) of in-progress animation
    allowAnimatedContent    // if not set, just interpolate between current and end "bits"
    curveEaseInEaseOut  // slower at the beginning, normal throughout, then slow at end
    curveEaseIn // slower at the beginning, but then constant through the rest
    curveLinear // same speed throughout
    ```

* pattern related
    ```swift
    transitionFlipFrom{Left,Right,Top,Bottom}   // flip entire view
    transitionCrossDissolve // Dissolve from old to new state
    transitionCurl{Up, Down}    // Curling up or down
    ```
    example
    ```swift
    UIView.transition(
                with: myPlayingCardView,
                duration: 0.75
                options: [.transitionFlipFromLeft],
                animations: { cardIsFaceUp = !cardIsFaceUp}
                completion: nil)
    ```

## Dynamic Animation
set up physics relating animatable objects and let them run until they resolve to stasis.

1. create a UIDynamicAnimator at the top view of animation parts
    ```swift
    var animator = UIDynamicAnimator(referenceView: UIView)
    ```
2. create and add UIDynamicBehavior instances
    ```swift
    let gravity = UIGravityBehavior()
    animator.addBehavior(gravity)
    collider = UICollisionBehavior()
    animator.addBehavior(collider)
    ```
3. add UIDynamicItem to a UIDynamicBehavior
    ```swift
    let item1: UIDynamicItem = ...// usually a UIView
    let item2: UIDynamicItem = ...// usually a UIView
    gravity.addItem(item1)
    collider.addItem(item1)
    gravity.addItem(item2) 
    ```
4. UIView implement this protocol, if you change center or transform while the animator is running, you must call this method in UIDynamicAnimator
    ```swift
    func updateItemUsingCurrentState(item: UIDynamicItem)
    ```

### Behaviors
* UIGravityBehavior
    ```swift
    var angle: CGFloat // in radians; 0 is to the right; clockwise
    var magnitude: CGFloat // 1.0 is 1000 points/s/s ~ 9.8
    ```
* UIAttachmentBehavior: a bar keeps two items like a stick? Unity
    ```swift
    init(item: UIDynamicItem, attachedToAnchor: CGPoint)
    init(item: UIDynamicItem, attachedTo: UIDynamicItem)
    init(item: UIDynamicItem, offsetFromCenter: CGPoint, attachedTo[Anchor]...)
    var length: CGFloat // can be set at any time
    var anchorPoint: CGPoint    // can be set at any time
    ```
    the attachment can oscillate like a spring with frequency and damping

* UICollisionBehavior
    ```swift
    var collisionMode: UICollisionBehaviorMode // .items, .boundaries or .everything
    ```
    * if `.items`, then any items added to a UICollisionBehavior will bounce off each other
    * if `.boundaries`, then you add UIBezierPath boundaries for items to bounce off of
        ```swift
        func addBoundary(withIdentifier: NSCopying, for: UIBezierPath)
        func addBoundary(withIdentifier: NSCopying, from: CGPoint, to: CGPoint)
        func removeBoundary(withIdentifier: NSCopying)
        var translatesReferenceBoundsIntoBoundary: Bool // referenceView's edges
        NSCopying // means NSString or NSNumber
        ```
    * find out when a collision happens
    ```swift
    var collisionDelegate: UICollisionBehaviorDelegate
    func collisionBehavior(behavior: UICollisionBehavior,
                began/endedContactFor: UIDynamicItem,
                withBoundaryIdentifier: NSCopying
                                    at: CGPoint)
    ```

* UISnapBehavior
    ```swift
    init(item: UIDynamicItem, snapTo: CGPoint)
    ```
    * `var damping: CGFloat` to control the damping of these "four springs"

* UIPushBehavior
    ```swift
    var mode: UIPushBehaviorMode // .continuous or .instantaneous
    var pushDirection: CGVector
    // or
    var angle: CGFloat // in radians and positive numbers are clockwise
    var magnitude: CGFloat // magnitude 1.0 moves a 100*100 view at 100 pts/s/s
    ```
    * remember to clear up the `.instantaneous` push behavior

* UIDynamicItemBehavior: meta behavior
    ```swift
    var allowsRotation: Bool
    var friction: CGFloat
    var elasticity: CGFloat
    //...
    ```
    * get information about items with this behavior
    func linearVelocity(for: UIDynamicItem) -> CGPoint
    func addLinearVelocity(CGPoint, for: UIDynamicItem)
    func angularVelocity(for: UIDynamicItem) -> CGFloat 
    ```

* UIDynamicBehavior: superclass of behaviors. You can create your own subclass which is a combination of other behaviors. Usually you override init method(s) and addItem and removeItem to call..
    ```swift
    func addChildBehavior(UIDynamicBehavior)
    ```
    This is a good way to encapsulate a physics behavior that is a  composite of other behaviors.
    * `action` property: Every time the behavior acts on items, this block of code that you can set is executed 
        ```swift
        var action: (() -> Void)?
        ```
        * you can set this to do anything you want. But it will be called **a lot**, so make it very efficient.
        * If the action refers to properties in the behavior itself, watch out for memory cycles.

        For example, in the case of an `.instantaneous` UIPushBehavior, when it is done acting on its items, it would be nice to remove it from its animator.
        ```swift
        if let pushBehavior = UIPushBehavior(items: [...], mode: .instantaneous){
            pushBehavior.magnitude = ...
            pushBehavior.angle = ...
            pushBehavior.action = { [unowned pushBehavior] in
                pushBehavior.dynamicAnimator!.removeBehavior(pushBehavior)
            }
            animator.addBehavior(pushBehavior) //will push right away
        }
        ``` 
        But this will cause memory cycle without `[unowned...]`.

Aside: Closure Capturing
        
* you can define local variables on the fly at the start of a closure.
    ```swift
    var foo = { [x = someInstanceOfaClass, y = "hello"] in
    // use x and y here
    }
    ```
* these locals can be declared `weak`        ```swift
    var foo = { [weak x = someInstanceOfaClass, y = "hello"] in
    // use x and y here, but x is now an Optional
    }
    ```
 * or they can even be declared "unowned": the reference counting system does count them
     ```swift
    var foo = { [unowned x = someInstanceOfaClass, y = "hello"] in
        // use x and y here
    }
    ```

Typical way to break memory cycle
```swift
class Zerg{
    private var foo = { [weak weaSelf = self] in 
    weakSelf?.bar()
    }
    private func bar() {}
}
```


* UIDynamicAnimator: all behaviors know the `UIDynamicAnimator` which they are part of
    *  They can only be part of one at a time
        ```swift
        var dynamicAnimator: UIDynamicAnimator? {get}
        ```
    * And the behavior will be sent this message when its animator changes
        ```swift
        func willMove(to: UIDynamicAnimator?)
        ```
    * `UIDynamicAnimator`'s delegate tells you when animation pauses. Just set the delegate
        ```swift
        var delegate: UIDynamicAnimatorDelegate
        func dynamicAnimatorDidPause(UIDynamicAnimator)
        func dynamicAnimatorWillResume(UIDynamicAnimator)
        ```
# View Controller Lifecycle
## Primary Setup
only called onece
```swift
override func viewDidLoad() {
    super.viewDidLoad()
    // update my View using my Model
}
```
**Do not do geometry-related setup here! Your bounds are not yet set**

## Will Appear
called many times
```swift
override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    // catch my View up to date
}
```

## Did Appear
```swift
override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)
    // maybe start a timer or an animation or start observing something (like GPS position)
}
```
This is also a good place to start something expensive (e.g. network fetch) going.
 
However, in this case, your UIs should always work even if the expensive thing has not come back yet.

For example, I wanna push an image on screen. So, I will put an image displaying thing on screen, that is not loaded, with a little spinner. And then, later,  the user can do whatever they want.

## Will Disappear
```swift
override func viewWillDisappear(_ animated: Bool){
    super.viewWillDisappear(animated)
    // undo what you did in viewDidAppear
    // stop a timer or stop observing something
}
```

## Did Disappear
"clean up" MVC. For example, you could save some state or release some large, recreatable resource.
```swift
override func viewDidDisappear(_ animated: Bool) {
    super.viewWDidDisappear(animated)
    // clean up MVC
}
```

## Geometry

```swift
override func viewWillLayoutSubviews()
override func viewDidLayoutSubviews()
```
Usually you don't need to do anything here because of Autolayout.

But if you do have geometry-related setup to do, this is the place to do it.

These can be called often. (just as `layoutSubviews()` in UIView can be called often)

## Autorotation
```swift
override func viewWillTransition(
    to size: CGSize,
    with coordinator: UIViewControllerTransitionCoordinator
)
```

## Low Memory
It is rare, but occasionally your device will run low on memory. Deal with memory leaking problem!.

```swift
override func didReceiveMemoryWarning(){
    super.didReceiveMemoryWarning()
    // stop pointing to any large-memory things
    // that I am not currently using
    // and that I can recreated as needed
}
```

## Waking up from an storyboard
earliest thing.
```swift
override func awakeFromNib(){
    super.awakeFromNib()
    // can initialize stuff
    // before outlets are set and 
    // before you're prepared as part of a segue
}
```

## Summary
Instantiated (from storyboard usually)

awakeFromNib (only if instantiated from a storyboard)

segue preparation happens

outlets get set

viewDidLoad

viewWillAppear and viewDidAppear

viewWillDisappear and viewDidDisappear

these "geometry changed" methods might be called at any time after viewDidLoad...

    viewWillLayoutSubviews and viewDidLayoutSubviews

if at any time, if memory gets low, you might get
viewDidReceiveMemoryWarning

# ScrollView
how big the scroll view is
```swift
scrollView.contentSize = CGSize(width: 3000, height: 2000)
```
where is current scroll view positioned?
```swift
let upperLeftOfVisible: CGPoint = scrollView.contentOffset
```

what area in a subview is currently visible?
```swift
let visibleRect: CGRect = aerial.convert(scrollView.bounds, from: scrollView)
```

## create
* drag out in a storyboard
* use `UIScrollView(frame:)`
* "Embed in -> Scroll View" from Editor menu

## Add UIView to UIScrollView
```swift
if let image = UIImage(named: "bigimage.jpg"){
    let iv = UIImageView(image:image)
    // iv.frame.size will = image.size
    scrollView.addSubview(iv)
}
```

## set the `contentSize`
```swift
scrollView.contentSize = imageView.frame.size
```

## Scrolling programmatically
```swift
func scrollRectToVisible(CGRect, animated: Bool)
```

## other things you can control
call `flashScrollIndicators` when your scroll view appears

`contentInset` whether the actual content is "inset" from the content area.

## Zooming
will not work without minimum/maximum zoom scale being set
```swift
scrollView.minimumZoomScale = 0.5
scrollView.maximumZoomScale = 2.0
```
will not work without `delegate` method to specify view to zoom
```swift
func viewForZooming(in scrollView: UIScrollView) -> UIView
```
zooming programatically
```swift
var zoomScale: CGFloat
func setZoomScale(CGFloat, animated: Bool)
func zoom(to rect: CGRect, animated: Bool)
```
## Delegate method
notify you when zooming ends
```swift
func scrollViewDidEndZooming(UIScrollView,
with view: UIView,  // from delegate method above
atScale: CGFloat)
```
# Multithreading
## Queue
*Main Queue*: All UI activity must occur on this queue and this queue only. Conversely, non-UI activity that is at all time consuming must **NOT** occur on that queue. Because we want our UI to be highly responsive.

    Functions are pulled off and worked on in the main queue only when it is "quiet"

*Global Queues*: A shared, global, concurrent queue for non-main-queue work.

## Getting a queue
Getting the main queue
```swift
let mainQueue = DispatchQueue.main
```
Getting a global queue. This is almost always that you will use to get activity off the main queue.
```swift
let backgroundQueue = DispatchQueue.global(qos: DispatchQos)
DispatchQoS.userInteractive // high priority, only do something short and quick
DispatchQoS.userInitiated // high priority, but might take a little bit of time
DispatchQoS.background // not directly initiated by user, so can run as slow as needed
DispatchQoS.utility // long-running background processes, low priority
```

* `.userInteractive`: access during drag or animation

## Putting a block code on the queue
Multithreading is simply the process of putting closures into these queues. There are two primary ways of putting a closure onto a queue.

You can just plop a closure onto a queue and keep running on the current queue
```swift
queue.async { ... }
```
Or you can block the current queue waiting until the closure finishes on that other queue
```swift
queue.sync { ... }
```
We almost always do the former.

## Getting a non-global queue
Very rarely you might need a queue other than main or global.

Your own serial queue (use this only if you have multiple, serially dependent activities)
```swift
let serialQueue = DispatchQueue(label: "MySerialQ")
```
Your own concurrent queue (rare that you would do this versus global queue)
```swift
let concurrentQueue = DispatchQueue(label: "MyConcurrentQ", attributes: .concurrent)
```

## More
there is a lot more to GCD (Grand Central Dispatch).
You can do locking, protect critical sections, readers and writers , synchronous dispatch, etc..

There is also another API to all of this
`OperationQueue` and `Operation`

Usually we use the DispatchQueue API, however. This is because the "nesting" of dispatching reads very well in the code. But the Operation API is also quite useful. (especially for more complicated multithreading)

## Multithreaded iOS API
Don't forget that is you want to do UI stuff there, you must dispatch back to the main queue!

### Example of a multithreaded iOS API:
This API lets you fetch the contents of an http URL into a Data off the main queue!
```swift
let session = URLSession(configuration: .default)
if let url = URL(string: "http://..."){
    let task = session.dataTask(with: url) { (data: Data?, response, error) in
    // do non-UI things
    DispatchQueue.main.async{
        // do UI stuff here
    }

    }
    task.resume()
}
```

How do we use the data to affect the UI then?

One way is to use a variant of this API that lets you specify the queue to run on (main queue).

Or do it in the `DispatchQueue.main.async{ ... }`

* Timing
a b g h c d f e. 

## example
Load image after an user chooses the segue.

1. use global thread
```swift
DispatchQueue.global(qos: .userInitiated).async {
//...
}
```
2. fetch data and if you don't wanna this command to be held in the heap. For example, maybe when the user won't wanna wait for its completion, set self to a weak pointer. Remember to add `?`
```swift
DispatchQueue.global(qos: .userInitiated).async { [weak self] in
                // try? return nil if failed.
                let urlContents = try? Data(contentsOf: url)
                if let imageData = urlContents{
                    self?.image = UIImage(data: imageData)
                    
                }
            }
```

3. global queue can't do UI thing. Bring the UIImage setting to main queue
```swift

```

4. check if the url is the image we want before fetch it
```swift
DispatchQueue.main.async {
                    if let imageData = urlContents, url == self?.imageURL{
                        self?.image = UIImage(data: imageData)
                    }
                }

```

5. add spinner to the View rather than the Scroll View
```swift
spinner.startAnimating()
spinner?.stopAnimating()
```

# Autolayout
We've seen a lost of Autolayout in the storyboard.

## size class
"compact" or "regular" width or height

* iPhone in portrait compact in width and regular in height
* non-Plus iPhones in landscape are compact in both dimensions
* iPhone plus in landscape is compact in height but regular in width

* iPad always regular in both dimension
* depending on the environment (like split view master), iPad might be compact.

## what can we do based on our size class?
Vary many properties in UIView: Fonts, background color, isHidden, even whether a view is installed in the view hierarchy.
**You can include or exclude any constraint based on size class**

* using size class information
```swift
let myHorizSizeClass: UIUserInterfaceSizeClass = traitCollection.horizontalSizeClass
```
return an enum either `.compact` or `.regular` or `.unspecified`

## do size class in the InterfaceBuilder



# Drag and Drop
## Interactions
A view "signs up" to participate in drag and/or drop using an interaction. It's kind of like a "gesture recognizer" for drag and drop.
```swift
let drag/dropInteraction = UIDrag/DropInteraction(delegate: theDelegate)
view.addInteraction(drag/dropInteraction)
```
 ## starting a drag
 ```swift
 func dragInteraction(_ interaction: UIDragInteraction, itemsForBeginning session: UIDragSession) -> [UIDragItem]
 ```

 * UIDragItem
 complete asynchronized
 ```swift
 let dragItem = UIDragItem(itemProvider: NSItemProvider(object: provider))
 ```
 Providers: NSAttributedString, NSString, UIImage, NSURL, UIColor, MKMapItem, CNContact.
Note that some of these types start with NS... You might have to use `as?` to cast them.

You can also provide an object that will be passed to drop targets inside your own app..
```swift
dragItem.localObject = someObject
```

## adding to a drag
Even in the middle of a drag, users can add more to their drag if you implement 
```swift
func dragInteraction(_ interaction: UIDragInteraction, itemsForAddingTo session: UIDragSession) -> [UIDragItem]
 ```

 ## Accepting a drop
 when a drag moves over a view with a UIDropInteraction, the delegate gets
 ```swift
 func dropInteraction(_ interaction: UIDropInteraction, canHandle session: UIDragSession) -> Bool
 ```
 at which point the delegate can refuse the drop before it even gets started.

 To figure that out, the delegate can ask what kind of objects can be provided
 ```swift
 let stringAvailable = session.canLoadObjects(ofClass: NSAttributedString.self)
 let imageAvailable = session. canLoadObject(ofClass: UIImage.self)
 ```
 can refuse the drop if it isn't your liking.

 If you don't refuse it in canHandle, then as the drag progresses, you'll start getting
 ```swift
 func dropInteraction(_ interaction: UIDropInteraction, sessionDidUpdate session: UIDragSession) -> UIDropProposal
 ```
to which you will respond with 
```swift
UIDropProposal(operation: .copy/.move/.cancel)
```
* `.cancel` means that drop would be refused
* `.copy` means drop would be accepted
* `.move` means drop would be accepted and would move the item (only for drags within an app)

If it matters, you can find out where the touch is with 
```swift
session.location(in: view)
```

## accepting a drop
If all that goes well and the user let's go of the drop, you get to fetch the data
```swift
func dropInteraction(_ interaction: UIDropInteraction, performDrop session: UIDropSession)
```
You will implement this method by calling `loadObjects(ofClass:)` on the session. This will go and fetch the data **asynchronously** from whomever the drag source is 
```swift
session.loadObject(ofClass: NSAttributedString.self) { theStrings in
// do something with the dropped NSAttributedStrings

}
```
The passed closure will be executed some time later on the **main thread**. You can call multiple `loadObjects(ofClass:)` for different classes. You don't usually do anything else in `dropInteraction(performDrop:)`

# Table and Collection View

## UITableView and UICollectionView
* Table View presents the information in a long (possibly sectioned) list.
* Collection View presents the information in a 2D format (usually "flowing" like text flows).
They are very similar in their API.

## UITableView
* It can show simple ancillary information.. 
* Or arbitrarily complex information.. 
* The rows can also be Grouped (usually when the information is fixed like setting)

## UICollectionView
It is configurable to show information in any 2D arrangement.

## where does the data come form
per MVC, "views are not allowed to own their data". So we can't just somehow set the data in some var.

Instead, we set a `var` called `dataSource`. The type of the dataSource var is a protocol with methods that supply the data. dataSource is exactly like a delegate in how it works.

They also have a `delegate`. Their delegate controls how they look, not what data they display (that's the dataSource)

## Setting the dataSource and delegate
In UITableView:
```swift
var dataSource: UITableViewDataSource
var delegate: UITableViewDelegate
```
In UICollectionView:
```swift
var dataSource: UICollectionViewDataSource
var delegate: UICollectionViewDelegate
```
These are automatically set for you if you use the prepackaged MVCs(the ones in yellow). 99.99% of the time, these vars will want to be set to the Controller of the MVC.

## The UITableView/CollectionViewDataSource protocol
the "data retrieving" protocol has many methods.
But these 3 are the core (UITV, UICV for abbreviation)
```swift
// UITableView
func numberOfSections(in tableView: UITV) -> Int
func tableView(_ tv: UITV, numberOfRowsInSection section: Int) -> Int
// UICollectionView
func numberOfSections(in collectionView: UICV) -> Int
func collectionView(_ cv: UICV, numberOfItemsInSection section: Int) -> Int
```
`IndexPath` specifies which row (in TV) or item (in CV) we are talking about.
* In both, you get the section the row or item is in from `indexPath.section`. 
* In TV, you get which row from `indexPath.row`;
* in CV you get which item from `indexPath.item`. CV might seem like rows and columns, but it's not. It's just items "flowing" like text.

## Loading up Cells
Let's focus on how we implement that last method.

In UITableView
```swift
func tableView(_ tv: UITV, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tv.dequeueReusableCell(withIdentifier: "MyCellId", for: indexPath)
}
```
This gets the UITableViewCell we are going to load up with our Model data and return. The UITableView will then use that UITableView Cell to draw the row at the given indexPath. 

### Cell Resuse
UITableView will reuse the cells. When a UITableViewCell scrolls off the screen, it gets put in a pool to be reused. The `dequeueReusableCell(withIdentifier:)` method grabs one out of that reuse poll.

If the reuse pool is empty, they get created by copying a prototype cell you configure in your storyboard.

**The fact that cells are reused has serious implications for multithreading.** By the time something returns from another thread, a cell might have been reused.

```swift
func tableView(_ tv: UITV, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let prototype = decision ? "FoodCell" : "CustomFoodCell"
    let cell = tv.dequeueReusableCell(withIdentifier: prototype, for: indexPath)
}
```

The `decision` can be made based on many factors. Usually it's based on the indexPath. But it might also be based on the data in our Model at that indexPath.

### Configure the cell
for UITableView, the default UITableView Cell has a few basic things
```swift
textLabel, detailTextLabel and imageView
```

but for UICollectionView and custom UITableViewCells, we have to implement it.

### UITableViewCell subclass
```swift
class MyTVC: UITableViewCell
{
    @IBOutlet var name: UILabel
    // .....
}

func tableView(_ tv: UITV, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let prototype = decision ? "FoodCell" : "CustomFoodCell"
    let cell = tv.dequeueReusableCell(withIdentifier: prototype, for: indexPath)
    it let myTVC = cell as? MyTVC {
        myTVC.name = food(at: indexPath)
        myTVC.emoji = emoji(at: indexPath)
    }
}
```
## Static Table View
* using table view purely for UI layout, like the iOS Settings app.
In this case, you do not need to do any of the UITableViewDataSource stuff. And you can connect outlets directly to your Controller.

To do this, just set your UITableView to have Static Cells instead of Dynamic Prototypes. Usually static table views are Style Grouped.

We can segue row from Accessory: *Detail Disclosure*. And set its identifier.

```swift
func prepare(for segue: UIStoryboardSegue, sender: Any?) {
    if let identifier = segue.identifier {
        switch identifier {
        case "XyzSegue": // handle it
        case "AbcSegue": // handle it
            if let cell = sender as? MyTableViewCell, let indexPath = tableView.indexPath(for: cell) 
            let seguedToMVC = segue.destination as? MyVC {
                seguedToMVC.publicAPI = data[indexPath.section][indexPath.row]
            }
        default: break
        }
    }
}
```
this sender can now be cast `as` a UITableViewCell or custom UITableViewCell.

## Collection View Segue
```swift
func collectionView(collectionView: UICV, didSelectItemAtIndexPath indexPath: IndexPath)
```
Use `performSegue(withIdentifier:)` from there. This strategy could also be used for UITableView.

## What if Model changes?
```swift
func reloadData()
```
Cause it to call `numberOfSectionsInTableView` and `numberOfRows/ItemsInSection` all over again and then `cellForRow/ItemAt` on each visible row or item.

Relatively heavyweight, but if your entire data structure changes, that's what you need. If only part of your Model changes, there are lighter-weight reloaders
```swift
func reloadRows(at indexPaths: [IndexPath], with animation: UITableViewRowAnimation)
```
among others and of course similar methods for Collection View.

## Controlling the height of rows in a Table View
* fixed row height `var rowHeight: CGFloat`
* autolayout `rowHeight = UITableViewAutomaticDimension`
    * if you do automatic, help the table view out by setting `estimatedRowHeight` to something
* UITableView's delegate can also control row heights
```swift
func tableView(UITableView, {estimated}heightForRowAt indexPath: IndexPath) -> CGFloat
```
Beware: the non-estimated version of this could get called **A LOT** if you have a big table.

## Controlling the size of cells in a Collection View
* Cell size can be fixed in the storyboard.
* You can also drive it from autolayout similar to table view.
* Or you can return the size from this delegate method ...
```swift
func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAt indexPath: IndexPath) -> CGSize
```
## Table View Headers
```swift
func tableView(_ tv: UITV, titleForHeaderInSection section: Int) -> String?
```
## Collection View Headers
Headers and footers are a bit more difficult in collection view. 

You can't just specify them as Strings.

1. you have to "turn then on" in the storyboard
2. they are reusable (like cells are), so you have to make a UICollectionReusableView subclass. You put your UILabel or whatever for your header, the hook up an outlet. Then you implement this dataSource method to dequeue and provide a header
    ```swift
    func collectionView(_ collectionView: UICollectionView, viewForSupplementaryElementOfKind kind: String, at indexPath: IndexPath) -> UICollectionReusableView
    ```
use `dequeueReusableSupplementaryView(ofKind: withReuseIdentifier: for:` in there. kind will be UICollectionElementKindSectionHeader or Footer.

## Other Methods
There are dozens of other methods in these classes.

# UITextField
* Keyboard appears when UITextField becomes "first responder". You can make it the first responder by sending it the `becomeFirstResponder` message. To make the keyboard go away, send `resignFirstResponder` to the UITextField.
### Delegate can get involved with Return key, etc.
```swift
func textFieldShouldReturn(sender: UITextField) -> Bool // when "Return" is pressed
```
* Oftentimes, you will `sender.resignFirstResponder()` in this method.
* Returns whether to do normal processing when Return key is pressed (e.g. target/action)

## Find out when editing has ended
```swift
func textFieldDidEndEditing(sender: UITextField)
```
sent when the text field resigns being first responder.

## UITextField is a UIControl
So you can also set up `target/action` to notify you when things change.

Just like with a button, there are different UIControlEvents which can kick off an action. Right-click on a UITextField in a storyboard to see the options available.

## Keyboard
* control the appearance of the keyboard. Set the properties defined in the `UITextInputTraits` protocol (`UITextField` implements).
```swift
var autocapitalizationType: UITextAutocapitalizationType // words, sentences,..
var autocorrectionType: UITextAutocorrectionType // .yes or .no
var returnKeyType: UIReturnKeyType 
var isSecureTextEntry: Bool // for passwords, for example
var keyboardType: UIKeyboardType // ASCII, URL, PhonePad, ...
```

* Other Keyboard functionality
keyboards can have accessory views that appear above the keyboard (custom toolbar, etc).
```swift
var inputAccessoryView: UIView // UITextField method
```

* the keyboard comes up over other views. So you may need to adjust your view positioning (especially to keep the text field itself visible). You do this by reacting to the `UIKeyboard{Will,Did}{Show,Hide}` Notifications sent by UIWindow.

We have not talked about what a `Notification` is yet, but it;s pretty simple.

You register a method to get called when a named "event" occurs like this
```swift
NotificationCenter.default.addObserver(
    self, 
    selector: #selector(theKeyboardAppeared(_:)),
    name: Notification.Name.UIKeyboardDidShow,
    object: view.window)
```
The event here is `Notification.Name.UIKeyboardDidShow`.

The object is the one who is causing the event to happen.
```swift
func theKeyboardAppeared(_ notification: Notification)
```
will get called when it happens. 

The `notification.userInfo` is a Dictionary that will have details about the appearance.

Almost always the reaction to the keyboard appearing over your text field is to scroll it visible. If the first responder is not in a scroll view, then position it so the keyboard never covers it. UITableViewController listens for this & scrolls table automatically if a row has a UITextField. 

## Other UITextField properties
```swift
var clearsOnBeginEditing: Bool
var adjustsFontSizeToFitWidth: Bool
var minimumFontSize: CGFloat    // always set this if you set adjustsFontSizeToFitWidth
var placeholder: String?
var background/disabledBackground: UIImage?
var defaultTextAttributes: [String: Any]
```

UITextFields have a "left" and "right" overlays. You can control in detail the layout of the text field (border, left/right view, clear button).

# Persistence
* UserDefaults
* Property List
* Archiving and Codable
* Filesystem
* Core Data
* Cloud Kit
* UIDocument
* UIDocumentBrowserViewController
## UserDefaults
a very lightweight and limited database. Great for things like "settings" and such. Do not use it for anything big!
### What can you store in UserDefaults
It only stores Property List data.

A *Property List* is any combo of `Array, Dictionary, String, Date, Data` or a number (`Int`, etc). This is an old Obj-C API with no type that represents all those, so this API uses Any. `CGFloat` is not a Property List.
### What does the API look like
It just stores and retrieves Property Lists by key
```swift
func set(Any?, forKey: String) // the Any has to be a Property List (or crash)
func object(forKey: String) -> Any? // the Any is guaranteed to be a Property List
```

### Reading and Writing
You don't usually create one of these databases with `UserDefaults()`

* Instead, you use the static `let` called standard
```swift
let defaults = UserDefaults.standard
```
* Setting a value in the database is easy since the `set` method takes an `Any?`
```swift
defaults.set(3.1415, forKey: "pi")
defaults.set([1,2,3,4,5], forKey: "My Array")
defaults.set(nil, forKey: "Some Setting")   // removes data at that key
```

* getting things
```swift
func double(forKey: String) -> Double // 0 if not Double
func array(forKey: String) -> [Any]? // nil if non-Array at that key
func dictionary(forKey: String) -> [String:Any]? // return String or Any
```
### saving 
Your changes will be occasionally autosaved.

But you can force them to be saved at any time with synchronize ..
```swift
if !defaults.synchronize(){
    // failed! but not clear what you can do about it
}
```
it is not "free" to synchronize, but it's not that expensive either
## Archiving
There are two mechanisms for making ANY object persistent.

A new mechanism in iOS  11 which is supported directly by the Swift language environment. 
### old way - `NSCoder` mechanism
* Requires all objects in an object graph to implement these two methods
```swift
func encode(with aCoder: NSCoder)
init(coder: NSCoder)
```
`init` -> `NSKeyedArchiver` -> `NSKeyedUnarchiver`

### new way - `Codable` mechanism
* Some of the standard types that are automatically Codable by Swift..

```swift
String, Bool, Int, Double, Float
Optional
Array, Dictionary, Set, Data
URL
Date, DateComponents, DateInterval, Calendar
CGFloat, AffineTransform, CGPoint, CGSize, CGRect, CGVector
IndexPath, IndexSet
NSRange
```
* If you have non-standard types, you can do something similar to the old method.
* convert Codable object graph to either JSON or a Property List
    ```swift
    let object: MyType = ...
    let jsonData: Data? = try? JSONEncoder().encode(object)
    ```
    * Note that this encode `throws`. You can `catch` and find errors easily.
    * btw, you can make a String object out of this `Data` like this 
        ```swift
        let jsonString = String(data: jsonData!, encoding: .utf8) //JSON is always utf8
        ```
* To get your object graph back from the JSON
```swift
if let myObjectL MyType = try? JSONDecoder().decode(MyType.self, from: jsonData!){
}
```
* JSON is not "strongly typed". So things like Date or URL are just strings. Swift handles all this automatically and is even configurable, for example
```swift
let decoder = JSONDecoder()
decoder.dataDecodingStrategy = .iso8601
```

* catching error during a decoding. The thing decode throws is an `enum` of type `DecodingError`. Note how we can get the *associated values* of a enum similar to how we do with switch.
```swift
do {
    let object = try JSONDecoder().decode(MyType.self, from: jsonData!)
} catch DecodingError.keyNotFound(let key, let context){
    print("Couldn't find key \(key) in JSON: `\(context.debugDescription)")
 }catch DecodingError.valueNotFound(let type, let context){

 }catch DecodingError.typeMismatch(let type, let context){

 }catch DecodingError.dataCorrupted(let context)
```

### `Codable` Example
```swift
struct MyType : Codable {
    var someDate: Date
    var someString: String
    var other: SomeOtherType // SomeOtherType has to be Codable too
}
```
If you vars are also Codable, then you're done.

* change keys for json storage
```swift
struct MyType : Codable {
    var someDate: Date
    var someString: String
    var other: SomeOtherType // SomeOtherType has to be Codable too
    
    private enum CodingKeys : String, CodingKey {
        case someDate = "some_date"
        // note that the someString var will now not be included in the JSON
        case other // this key is also called "other" in JSON
    }

    // participate directly in the decoding by implement the decoding initializer
    init(from decoder: Decoder) throws{
        let container = try decoder.container(keyedBy: CodingKeys.self)
        someDate = try container.decoder(Date.self, forKey: .someDate)

        let superDecoder = try container.superDecoder()
        try super.init(from: superDecoder) // only if MyType is class
    }
    // participate directly in the encoding
    func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        try container.encode(someDate, forKey: .someDate)
        // encode the rest of var
    }
}
```
* Don't call `super.init` with your own decoder (use your container's superDecoder)

## File System
Your application sees iOS file system like a normal Unix filesystem. It starts at `/`. In fact, you can only read and write in your application's *sandbox*.

### Why sandbox
* security
* privacy
* cleanup

### What's in sandbox
* Application directory - not writeable. Your executable, .storyboards, .jpgs, etc
* Documents directory - Permanent storage created by and always visible to the user.
* Application Support directory - Permanent storage not seen directly by the user.
* Caches directory - Store temporary files here. This is not backed by iTunes
* Other directories (see documentation)

### Getting a path to these sandbox directories
`FileManager` (along with `URL`) is what you use to find out about what;s in the file system.

You can, for example, find the URL to these special system directories like this
```swift
let url: URL = FileManager.fault.url(
    for directory: FIleManager.SearchPathDirectory.documentDirectory,
    in domainMask: .userDomainMask // always .userDomainMask on iOS
    appropriateFor: nil, // only meaningful for replace file operations
    create: true // whether to create the directory if it doesn't already exist.
)
```
* examples of `SearchPathDirectory` values
    * `.documentDirectory`
    * `.applicationSupportDirectory`
    * `.cachesDirectory`, ...

* Building on top of these system paths:
```swift
// URL methods
func appendingPathComponent(String) -> URL
func appendingPathExtension(String) -> URL // e.g. "jpg"
```
* find out about what's at the other end of a URL
    ```swift
    var isFileURL: Bool // is this a file URL (whether file exist or not) or something else like HTTP URL?
    func resourceValues(for keys: [URLResourceKey]) throws -> [URLResourceKey: Any]?
    ```
    * Example keys: `.creationDateKey`, `isDirectoryKey`, `.fileSizeKey`


### Reading & writing
* Data:
```swift
init(contentsOf: URL, options: Data.ReadingOptions) throws
```
* writing binary data to a URL
    ```swift
    func write(to url: URL, options: Data.WritingOptions) throws -> Bool
    ```
    * The options can be things like `.atomic` (write to tmp file, then swap) or `.withoutOverwriting`.
* FIleManager provides utility operations
```swift
fileExists(atPath: String) -> Bool
```
can create and enumerate directories; move, copy, delete files. Thread safe (as long as a given instance is only ever used in one thread). Also has a delegate with lots of "should" methods (to do an operation or proceed after an error).

## Core Data
Object-oriented database, sometimes, you need to store large amounts of data locally in a database. And you need to search through it in an efficient, sophisticated manner. sql back end. 2016-17's iTunesU.

### how to access
via an `NSManagedObjectContext`.
### create/update objects
```swift
let context: NSManagedObjectContext = ...
if let tweet = Tweet(context: context){
    tweet.text = "140 characters of pure joy"
    tweet.created = Date()
    let joe = TwitterUser(context: tweet.managedObjectContext)
    tweet.tweeter = joe
    tweet.tweeter.name = "Joe Schmo"
}
```
### delete objects
```swift
context.delete(tweet)
```
### save changes
You **must** explicitly `save` any changes to a context, but note that this `throws`.
```swift
do {
    try context.save()
} catch {
    // deal with error
}
```
However, we usually use a UIManagedDocument which autosaves for us.
### Query
searching for objects in the database.

1. let's say we want to query for all TwitterUsers
```swift
let request: NSFetchRequest<TwitterUser> = TwitterUser.fetchRequest()
```
2. who have created a tweet in the last 24 hours
```swift
let yesterday = Date(timeIntervalSinceNow: -24*60*60) as NSDate
request.predicate = NSPredicate(format: "any tweets.created > %@", yesterday)
```
3. sorted by the TwitterUser's name
```swift
request.sortDescriptors = [NSSortDescriptor(key: "name", ascending: true)]
```
4. let's start searching
    ```swift
    let context: NSManagedObjectContext = ...
    let recentTweeters = try? context.fetch(request)
    ```
    * returns an empty Array (not `nil`) if it succeeds and there are no matches in the database.
    * returns an Array of `NSManagedObject`s (or subclasses thereof) if there were any matches.
    * And obviously the `try` fails if the fetch fails.

### much more
* Very efficient
* Support for multithreading
* Close integration with UITableView
* Optimistic

## Cloud Kit - 2015-16's iTunesU
A database in the cloud. Simple to use, but with very basic "database" operation.

Since it's on the network, accessing the database could be slow or even impossible.

This requires some  thoughtful programming.

### Important Components
* Record Type - like a class or struct
* Fields - like vars in a class or struct
* Record - an "instance" of a Record Type
* Reference - a "pointer" to another Record
* Database - a place where Records are stored
* Zone - a sub-area of a Database
* Container - collection of Databases
* Query - an Database search
* Subscription - a "standing Query" which sends push notifications when changes occur

### Use
you must enable enable iCloud in your Project Settings.
        
    under Capabilities tab, turn on iCloud (On/Off switch).
    Then, choose CloudKit from the Services

Cloud Kit Dashboard

    a web-based UI to look at everything you are storing.

Dynamic Schema Creation

    But you don't have to create your schema in the Dashboard.
    You can create it "organically" by simply creating and storing things in the database.
    When you store a record with a new, never-before-seen Record Type, it will create that type.
    Or if you add a Field to a Record, it will automatically create a Field for it in the database.
    This only works during Development, not once you deploy to your users.

* create
```swift
let db = CKContainer.default.publicCloudDatabase
let tweet = CKRecord("Tweet")
tweet["text"] = "140 characters of pure joy"
let tweeter = CKRecord("TwitterUser")
tweet["tweeter"] = CKReference(record: tweeter, action: .deleteSelf)
db.save(tweet){ (savedRecord: CKRecord?, error: NSError?) -> Void in 
    if error == nil {

    }else if error?.errorCode = CKErrorCode.NotAuthenticated.rawValue {
        // logged in to iCloud
    } else{
        // 29 CKErrorCodes
    }
}
```
* query
```swift
let db = CKContainer.default.publicCloudDatabase
let predicate = NSPredicate(format: "text contains %@", searchString)
let query = CKQuery(recordType: "Tweet", predicate: predicate)
db.perform(query) { (records: [CKRecord]?, error: NSError?) in
    if error == nil{
        // no error
    }else if error?.errorCode = CKErrorCode.NotAuthenticated.rawValue {
        // logged in to iCloud
    } else{
        // 29 CKErrorCodes
    }
}
```
* Subscriptions
    
    One of the coolest features of Cloud Kit is its ability to send push notifications on changes. All you do is register an NSPredicate and whenever the database changes to match it, boom!

## UIDocument
* When to use UIDocument

    If your application stores user information in a way the user perceives as a "document".
    If you just want iOS to manage the primary storage of user information.

* what does UIDocument do

    Manages all interaction with storage devices (not just filesystem, but also iCloud, Box, etc.).
    Provides **asynchronous** opening, writing, reading and closing of files.
    Autosaves your document data.
    Makes integration with iOS 11's new Files application essentially free.

* What do you need to do to make UIDocument work

    Subclass UIDocument to add `var`s to hold the Model of your MVC that shows your "document".
    Then implement one method that writes the Model to a `Data` and one that reads it from a `Data`.
    Now you can use UIDocument's opening, saving and closing methods as needed.
    You can also use its "document has changed" method (or implement undo) to get autosaving


1. Subclass UIDocument

    ```swift
    class EmojiArtDocument: UIDocument {
        var emojiArt: EmojiArt?
    }
    ```
2. create a UIDocument

    ```swift
    // Figure out where to store the document in the filesystem
    var url = FileManager.urls(for: .documentDirectory, in: .userDomainMask).first!
    url = url.appendingPathComponent("Untitled.foo")
    // instantiate your subclass by passing that url to UIDocument's only initializer ...
    let myDocument = EmojiArtDocument(fileURL: url)
    // then (eventually) set your Model var(s) on your newly created UIDocument subclass
    myDocument.emojiArt = ...
    ```
3. Create a Data for your Model

    ```swift
    override func contents(forType typeName: String) throws -> Any {
        return emojiArt converted into a Data
    }
    ```
    Note that the return type is `Any`... That's because your file format can also be a `FileWrapper`. A `FileWrapper` represents a directory full of files that make up your document.
    The `forType` is a UTI (type identifier) calculated from your fileURL's file extension (e.g. `.jpg`).

4. Turning a Data into a Model

    ```swift
    override func load(fromContents contents: Any, ofType typeName: String?) throws {
        emojiArt = contents concerted into an EmojiArt
    }
    ```
    more likely to throw!
5. Ready to go

    ```swift
    // open
    myDocument.open { success in
        if success {
            // ready to use
        } else{

        }
    }
    // save
    myDocument.updateChangeCount(.done)
    // undoManager
    //..
    
    // force a save
    let url = myDocument.fileURL
    myDocument.save(to url: URL, for: UIDocumentSaveOperation) { success in 
        if success {

        }  else {

        } 
    }
    ```
    `UIDocumentSaveOperation` is either `.forCreating` or `.forOverwriting`.
6. close your document

    ```swift
    myDocument.close { success in
        if success {

        } else{

        }
    }
    ```

* Document State
    ```swift
    var documentState: UIDocumentSate
    ```
    * .normal - document is open and ready for use!
    * .closed - document is closed and must be opened to be used
    * .savingError - document couldn't be saved (override `handleError` if you want to know why)
    * .editingDisabled - the document cannot currently be edited (so don't let your UI do that)
    * .progressAvailable - how far a large document is in getting loaded ( check `progress` var)
    * .inConflict - someone edited this document somewhere else. To resolve conflicts, you access the conflicting version with ...
    `NSFileVersion.unresolvedConflictVersionsOfItem(at url: URL) -> [NSFileVersion]?`
    For the best UI, you could give your user the choice of which version to use. Or, if your document's contents are "mergeable", you could even do that.
    * documentState can be "observed" using the UIDocumentStateChanged notification (more later)

* Thumbnail

    You can specify a thumbnail image for your UIDocument (icon)
    It can make it much easier for users to find the document they want in Files, for example.
    Essentially you are going to override the UIDocument method which sets file attributes. 
    The attributes are returned as a dictionary.
    ```swift
    override func fileAttributesToWrite(to url: URL, for operation: UIDocumentSaveOperation) throws -> [AnyHashable: Any] {
        var attributes = try super.fileAttributesToWrite(to: url, for: saveOperation)
        if let thumbnail: UIImage = ... {
            attributes[URLResourceKey.thumbnailDictionaryKey] = 
            [URLThumbnailDictionaryItem.NSThumbnail1024x1024SizeKey: thumbnail]
        }
        return attributes
    }
    ```
    It does not have to be 1024x1024.

* Other

    ```swift
    var localizedName: String
    var hasUnsavedChanges: Bool
    var fileModificationDate: Date?
    var userActivity: NSUserActivity? // iCloud documents only
    ```


## UIDocumentBrowserViewController
* Managing user documents. Users can easily manage their documents in a document-based app.

* It has to be the root view controller in your storyboard.

* what document types can be opened?

    You have to register which types your application uses.
    You do this in the Project Settings in the Info tab with your Target selected.
    In the Document Types area, add the types you support.
    * The `Types` field is the UTI of the type you want to support (`public.json`, `public.image`)
    * The `CFBundleTypeRole` and `LSHandlerRank` say how you handle this kind of document. Are you the primary editor and owner of this type or is it just something you can open?
* Declaring your own document type

    you can add this under Exported UTIs in the same place in Project Settings.

* Xcode template

    Setting up a UIDocumentBrowserViewController-based application requires a bit of setup. Mostly an entry in your Info.plist, a little bit of AppDelegate code and some stubbed-out code. 

* What is in the template?

    * A stub for Document Types in Project Settings (supports `public.image` file types)
    * An Info.plist entry Supports Document Browser = YES
    * A bit of code in AppDelegate to allow other apps (like Files) to get your app to open a file
    * A stubbed out UIDocument subclass (with empty contents and load(fromContents) methods)
    * A stubbed out MVC to display a document (just calls UIDocument's `open` and `close` methods)
    * A subclass of UIDocumentBrowserViewController (with almost everything implemented)

* What you need to do to personalize this template

    1. Use your `UIDocument` subclass instead of the stubbed out one
    2. Use your document-viewing MVC (already using UIDocument) instead of stub
    3. Add code to UIDBVC subclass to
        * configure the UIDBVC (allow multiple selection? creation of new documents? etc.)
            ```swift
            override func viewDidLoad(){
                super.viewDidLoad()
                delegate = self
                allowsDocumentCreation = true
                allowsPickingMultipleItems = true
                browserUserinterfaceStyle = .dark
                view.tintColor = .white
            }
            ```
        * specify the url of a template document to copy to create new documents
            ```swift
            func documentBrowser(_ controller: UIDBVC, didRequestDocumentCreationWithHandler handler: @escaping (URL?, UIDBVC.ImportMode) -> Void ) {
                let url: URL? = ... // where your blank , temolate document can be found
                importHandler(url, .copy or .move)
            }
            ```
            Usually you would specify .copy, but you could create a new template ach time and .move.
            Likely you would have some code here that creates that blank template.
        * present your document-viewing MVC modally given the url of a document
            ```swift
            func presentDocument(at url: URL){
                let story = UIStoryboard(name: "main", bundle: nil)
                if let docvc = story.instantiateViewController(withIdentifier: "DocVC") as? DocVC {
                    docvc.document = MyDocument(fileURL: url)
                    present(docvc, animated: true)
                }
            }
            ```
            You can call this function anything you want. But the point is that it takes a URL to one of your documents and you show it.
        * Update the Document Types in Project Settings to be your types (instead of public.image)
* Aside: Presenting an MVC without segueing

    How to present MVCs in any other way except by segueing.
    You present a new MVC from an existing MVC using `present(animated:)`
    ```swift
    let newVC: UIViewController = ...
    existingVC.present(newVC, animated: true){
        // completion handler called when the presentation completes animating
        // can be left out entirely if you don't need to do anything upon completion
    }
    ```
    * The real trick is "where do I get newMVC from"

        * you get it from your storyboard using its identifier which you set in Identity Inspector
            ```swift
            let storyboard = UIStoryboard(name: "Main", bundle: nil)
            if let newVC = storyboard.instantiateViewController(withIdentifier: "foo") as? MyDocVC {
                // "prepare" newMVC and then present(animated:) it
            }
            ``` 


# Alert and Action Sheet
Two kinds of "pop up and ask the user something" mechanisms.

## Alert
Pop up in the middle of the screen.
* Usually it has only two answers. 
* It can be disruptive to your UI
* Often used for "asynchronous" problem 
* It can have a text field to get a quick answer (e.g. pwd)
```swift
```swift
var alert = UIAlertController(
    title: "Login Required",
    message: "Please enter your Cassini guidance system..."
    perferredStyle: .alert
)
alert.addAction(UIAlertAction(
    title: "Cancel",
    style: .cancel) 
    { (action: UIAlertAction) -> Void in
        // do nothing
    }
)

alert.addAction(UIAlertAction(
    title: "Login",
    style: .default)
    { (action: UIAlertAction) -> Void in


    }
)

alert.addTextField(configurationHandler: { textField in
    textFiled.placeholder = "Guidance System Password"
    textField.isSecureTextEntry = true
})

present(alert, animated: true, completion: nil)
```

## Action Sheets
Slides in from the bottom of the screen and in a popover on iPad.
* can be displayed from bar button item or from any rectangular area in a view
* generally asks questions that have more than two answers.
* think of action sheets as presenting "branching decisions" to the user. (i.e. what next?)

```swift
var alert = UIAlertController(
    title: "Redeploy Cassini",
    message: "Issue commands to Cassini's guidance system."
    perferredStyle: .actionSheet / .alert
)
alert.addAction(UIAlertAction(
    title: String,
    style: UIAlertActionStyle,
    handler: (action: UIAlertAction) -> Void
))
alert.addAction(UIAlertAction(
    title: "Orbit Saturn",
    style: .default,) 
    { (action: UIAlertAction) -> Void in
        if !self.loggedin { self.login( )
        // if loggedIn go to titan}
    }
)

alert.addAction(UIAlertAction(
    title: "Orbit Saturn",
    style: .destructive) 
    { (action: UIAlertAction) -> Void in
        if !self.loggedin { self.login( )
        // if loggedIn go to titan}
    }
)

alert.addAction(UIAlertAction(
    title: "Cancel",
    style: .cancel) 
    { (action: UIAlertAction) -> Void in
        // do nothing
    }
)

// iPad do not need to distinguish
alert.modalPresentationStyle = .popover
let ppc = alert.popoverPresentationController
ppc?.barButton = redeployBarButtonItem

// enable it
present(alert, animated: true, completion: nil)
```
note that on iPad, no Cancel action.
# Notifications
The way you register a closure to get call when someone broadcasts on a named radio station

```swift
var observer: NSObjectProtocol?
observer = NotificationCenter.default.addObserver(
    formName: Notification.Name, // not a String 
    object: Any?,               // the broadcaster (or nil for "anyone")
    queue: OpeartionQueue?      // the queue on which to dispatch the closure below (nil = same queue as that of the broadcaster)
) { (notification: Notification) -> Void in // closure executed when broadcasts occur
    let info: Any? = notification.userInfo
    // info is usually a dictionary of notification-specific information
}
```

## Notification.Name
static vars on Notification.Name

```swift
Notification.Name(String)
// better to create an extension
```

## example of changing font size using system setting
```swift
let center = NotificationCenter.default
var observer = center.addObserver(
    forName: Notification.Name.UIContentSizeCategoryDidChange,
    object: UIApplication.shared,//or nil
    queue: OperationQueue.main  // or nil
) { notification in 
    // re-set the fonts of objects using preferred fonts
    // or look at the size category and do something with it 
    let c = notification.userInfo?[UIContentSizeCategoryNewValueKey]
    // c might be UIContentSzieCategorySmall, for example
}
center.removeObserver(observer) // when you're done listening
```

* add in `viewWillAppear`
* remove in `viewWillDisappear`

## Posting a Notification
```swift
NotificationCenter.default.post(
    name: Notification.Name,    // name of the "radio station"
    object: Any?,               // who is sending this notification (usually self)
    userInfo: [AnyHashable: Any]? = nil // any info you want to pass to station listeners.
)
``` 
Any closures added with `addObserver` will be executed

# KVO
Watching the properties of NSObject subclasses. The basic idea of KVO is to register a closure to invoke when a property value changes.

There is some "mechanism" required to make this work. NSObject implements this mechanism. Thus objects that inherit from NSObject can participate.

## what's it good for?
Usually used by a Controller to observe either its Model or its View

Not every property works with KVO. 
* A property has to be Key Value Coding-compliant to work
    * `setValueForKey`, `getValueForKey`
For example, UIView's frame and center work with KVO. So does most of CALayer underneath UIView. And in database.

## How does it work?
```swift
var observation = observed.observe(keyPath: KeyPath) { (observed, change) in
    // code to execute when the property described by keyPath changes
}
```
* As long as the `observation` remains in the heap, the closure will stay active.
* The `change` argument to the closure is an `NSKeyValueObservedChange`, which has the old value and the new value in it.
* The syntax for a `keyPath` is `\Type.property` or even `\Type.prop1.prop2.prop3`. Swift can infer the Type.



# Application LifeCycle
Not running ->  [Foreground: Inactive <-> Active] 
      |                          |
 Suspeneded <-> [Background: Background]

 * Inactive: Running your code but no UI events, e.g. preparation
 * Active: Running your code, receving and processing UI events
 * Background: Running your code for a limited time, no UI events
 * Suspended: Your code not running. You could be killed at any time.

1. Launching: Not Running -> Inactive -> Active
2. Switch to another application: Active -> Inactive -> Background --> Suspended --> Killed -> Not running

## Not Running -> Inactive
Your AppDelegate will receive
```swift
func application(UIApplication, will/didFinishLaunchingWithOptions:
    [UIApplicationLaunchOPtionsKey:Any]? = nil)
```
and you can observe
```swift
UIApplicationDidFinishLaunching
```
The passed disctionary (also in `notification.userInfo`) tells you why your application was launched.

Some examples:
* Someone wants you to open a URL
* You entered a certain place in the world
* You are continuing an activity started on another device
* A notification arrived for you (push or local)
* Bluetooth attached device wants to interact with you


It used to be that you would build your UI here. But nowadays we use storyboards for all that. So often you do not implement this method at all.

## Inactive <-> Active
### Inactive -> Active
our AppDelegate will receive:
```swift
func applicationDidBecomeActive(UIApplication)
```
and you can observe
```swift
UIApplicationDidBecomeActive
```
If you have "paused" your UI previously, here's where you would reactivate things.
### Active -> Inactive
Your AppDelegate will receive:
```swift
func applicationWillResignActive(UIApplication)
```
and you can observe
```swift
UIApplicationWillResignActive
```
You will want to "pause" your UI here. This might happen because a phone call comes in. Or you might be on your way to the background.


## Inactive <-> Background
### Inactive -> Background
our AppDelegate will receive:
```swift
func applicationDidEnterBackground(UIApplication)
```
and you can observe
```swift
UIApplicationDidEnterBackground
```
Here you want to (quickly) batten down the hatches. You only get to run 30s or so. You can request more time, but don't abuse this (or the system will start killing you instead).

prepare yourself to be eventually killed here (it probably won't happen, but be ready anyway).

### Background -> Inactive
our AppDelegate will receive:
```swift
func applicationWillEnterForeground(UIApplication)
```
and you can observe
```swift
UIApplicationWillEnterForeground
```
Time to un-batten the hatches. Maybe undo what you did in DidEnterBackground. You will likely soon be made Active.

## Other AppDelegate items
* State Restoration
* Data Protection
* Open URL
* Background Fetching

## UIApplication
* shared instance

    There is a single `UIApplication` instance in your application
    ```swift
    let myApp = UIApplication.shared
    ```
    it manages all global behavior. You never need to subclass it. It delegates everthing you need to be involved in to its `UIApplicationDelegate`. However, it does have some useful functionality.

    * Opening a URL in another application
        ```swift
        func open(URL)
        func canOpenURL(URL) -> Bool
        ```
    * Registering to receiving Push Notifications   
        ```swift
        func (un)registerForRemoteNotifications()
        ```
        Notifications, both local and push, are handled by the `UNNotification` framework.
    * Setting the fetch interval for background fetching

        You must set this if you want background fetching to work
        ```swift
        func setMinimumBackgroundFetchInterval(TimeInterval)
        ```
        Usually you will set this to `UIApplicaitonBackgroundFetchIntervalMinimum`
    * Asking for more time when backgrounded
        ```swift
        func beginBackgroundTask(withExpiraritionHandler: (() -> Void)?) -? UIBackgroundTaskIdentifier
        ```
        DO NOT forget to call `endBackgroundTask(UIBackgroundTaskIdentifier)` when you're done!
    * Turning on the "network in use" spinner (status bar upper left)
        ```swift
        var isNetworkActivityIndicatorVisible: Bool
        ```
    * Finding out about things  
        ```swift
        var backgroundTimeRemaining: TimeInterval {get} // until you are  suspended
        var preferredContentSizeCategory: UIContentSizeCategory {get} // big fonts or small fonts
        var applicationState: UIApplicationState { get } // foreground, background, active
        ```

## Info.plist
Many of your application's settings are in `Info.plist`

Or you can even edit it as raw `XML`

### Capabilities
Some features require enabling, like iCloud, Game Center, etc.

Switch on in Capabilities tab inside your Project Settings.

# More on Segues
* Modal
* Popover
* Unwind

## Modal View Controllers
A way of segueing that takes over the screen, like Contacts application.

* How do we get a model segue up?
    * Just ctrl-drag from a button to another View Controller & pick segue type "Modal".
    * If you need to present a Modal VC not from a button, use a manual segue
    ```swift
    func performSegue(withIdentifier: String, sender: Any?)
    ```
    * or, if you have the view controlelr itself (e.g. Alerts or fomr instantiateViewController)
    ```swift
    func present(UIViewContrller, animated: Bool, completion: (()->Void)? = nil)
    ```
* Preparing for a Modal segue
    ```swift
    func prepare(forL UIStoryboardSegue, sender: Any?) {
        if segue.identifier = "GoToMyModalVC" {
            let vc = segue,destination as MyModalVC
            // set up the vc to run here
        }
    }
    ```
* Hearing back from a Modally segued-to View Controller
    
    When the Modal View Controller is "dont", if there's nothing to be said, just dismiss the segued-to MVC. To communicate results, generally you would use Unwind segue.

* How to dismiss a view controller
    ```swift
    func dismiss(animated: Bool, completion: (()-> Void)? = nil)
    ```
    send this message to the presenting view controller. You can get at your presenting view controller with this UIViewController property
    ```swift
    var presentingViewController: UIViewController?
    presentingViewController?.dismiss(...)
    ```
    *Unwind Segues* automatically dismiss
* Controlling the appearance of the presented view controller
    ```swift
    var modalPresentationStyle: UIModalPresentationStyle
    ```
    * In addition to the default, `.fullScreen`, there's `.overFullScreen` (presenter visible behind for example blured)

    * On iPad, there's `.formSheet` and `.pageSheet`
    * In horizontally compact environments, there will all adapt to always be full screen!

* How is the modal view controller animated onto the screen?
    ```swift
    var modalTransitionStyle: UIModalTransitionStyle
    .coverVertical
    .flipHorizontal
    .crossDissolve  
    .partialCurl    // only if presenting VC is full screen
    ```
    The presentation & transition can be set via Identifier Inspector

## Demo

## Popover
Popovers pop an entire MVC over the rest of the screen.

* A popover is almost exactly the same as a Modal segue
* Things to note when preparing for a popover segue
    * All segues are managed via a `UIPresentationController`. We are only going to talk about a popover's `UIPopoverPresentationControler`. It notes what caused the popover to appear (a bar button item or just a rectangle in a view). 
    * You can also control what direction the popover's arrow is allowed to points.
    * Or you can control how a popover adapts to different sizes classes (iPad / iPhone)

```swift
func prepare(for segue: UIStoryboardSegue, sender: Any?) {
    if let ideintifier = segue.identifier {
        switch identifier {
            case "Do Something in a Popover Segue":
                if let vc = segue.destination as? MyController {
                    if let ppc = vc.popoverPresentationController {
                        ppc.permittedArrowDirections = UIPopoverArrowDirection.any
                        // adapting
                        ppc.delegate = self
                    }
                    // more preparation
                }
            default: break
        }
    }
}
```
* Adapatation to different size classes
    * When a popover is presenting itself in a horizontally compact environment (e.g. iPhone), there might not be enough room to show a popover window comfortably, so by default it "adapts" and shows the MVC in full screen modal instead.
    * But the popover presentation controller's delegate can control this "adapation" behavior. Either by preventing it entirely
        ```swift
        func adaptivePresentationStyle(
            for controller: UIPresentationController,
            traitCollection: UITraitCollection
        ) -> UIModalPresentaionStyle {
            return UIModalPresentationStyle.none // don't adapt
            // the default in horizontally compact environments (iPhone) is .fullScreen
        }
        ```
    * You can control the view controller that is used to present in the adapted environment (iPhone)
        ```swift
        // example: wrapping a UINavigationController around the MVC that is presented
        func presentaionController(controller: UIPresentationController,
        viewControlelrForAdaptivePresentationStyle: UIModalPresentationStyle)
        -> UIViewController?{
            // return a UIViewController to use (e.g. wrap a Navigation Controller around your MVC)
        }
        ```
* Important Popover Issue: Size
    ```swift
    var preferredContentSize: CGSiz
    ```

## Unwind Segue

It is the only segue that does Not create a new MVC. It can only segue to other MVCs that (directly or indirectly) presented the current MVC.

* What is it good for?
    * Jumping up the stack of cards in a navigation controller (other cards are considered presenters).
    * Dismissing a Modally segued-to MVC while reporting information back to the presenter.

* How does it work?

    * ctrl-drag the "Exit" button in the same MVC. Then you can choose a special `@IBAction` method you've created in anther MVC.
    * This means "segue by exiting me and finding a presenter who implements that method". If no presenter implements that method, the segue will not happen.
    ```swift
    @IBAction func goBack(segue: UIStoryboardSegue){
        let mvcUnwoundFrom = segue.source as? MyPresentedMVC
        myVar = mvcUnwoundFrom?.publicAPI
    }
    ```
    you will be dismissed while goBack.

Implement 
```swift
@IBAction func aFunToUseByExitButton(bySegue: UIStoryboardSegue){
    // do anything you want
}
```
## Embed Segue
Put a VC's self.view in another VC's view hierarchy!

Drag out a `Container View` from the object palette into the scene you want to embed it in. Automatically sets up an "Embed Segue" from container VC to the contained VC.

```swift
prepare(for segue:. semder:)
```

* View Loading Timing
* Don't forget. the embedded VC's outlets are not set at the time `prepare(for segue:, sender:)` is called.


# Core Motion & Camera
API to access motion sensing hardware on your device

* primary inputs: Accelerometer, Gyro, Magnetometer
* Class used to get this input is `CMMotionManager`

## Usage
1. check to see what hardware is available
    ```swift
    var {accelerometer, gyro, magnetometer, deviceMotion}Available: Bool
    ```
2. start the sampling going and poll the motion manager for the lastest sample it has
    ```swift
    func start{accelerometer, gyro, magnetometer, deviceMotion}Updates()
    ```
* Is the hardware currently collecting data?
    ```swift
    var {accelerometer, gyro, magnetometer, deviceMotion}Active: Bool
    ```
* Stop the hardware collecting data!!!
    ```swift
    func stop{accelerometer, gyro, magnetometer, deviceMotion}Updates()
    ```
or 
1. check to see what hardware is available
2. set the rate at which you want data to be reported from the hardware
3. Register a closure (and a queue to run it on) to call each time a sample is taken

## Accelerometer
* Checking the data
    ```swift
    var accelerometerData: CMAccelerometerData?
    // it provides 
    var acceleration: CMAcceleration
    struct CMAcceleration {
        var x: Double // in g (9.8 m/s/s)
        var y: Double // in g
        var z: Double // in g
    }
    ```
    This raw data includes acceleration due to gravity. So if the device were laid flat, z would be 1.0, and x&y would be 0.0

## Gyro
* Checking the data
    ```swift
    var gyroData: CMGyroData?
    // it provides
    var rotationRate: CMRotationRate
    struct CMRotataionRate {
        var x: Double   // in radians/s
        var y: Double
        var z: Double
    }
    ```
    sign of the rotation data follows right hand rule. The data above will be biased

## magnetometer
* Checking the data
    ```swift
    var magnetometerData: CMMagnetometerData?
    // it provides
    var magneticField: CMMagneticField
    struct CMMagneticField {
        var x: Double   // in mircroteslas
        var y: Double
        var z: Double
    }
    ```
    The data above will be biased

## deviceMotion
* Checking the data
    ```swift
    var deviceMotion: CMDeviceMotion?
    // it provides
    // acceleration data in CMDeviceMotion
    var gravity: CMAcceleration
    var userAccele ration: CMAcceleration // gravity factored out using gyro

    var rotationRate: CMRotationRate // bias removed from raw data using accelerometer
    var attitude: CMAttitude    // device's attitude (orientation) in 3D space
    class CMAttitude: NSObject {
        var roll: Double
        var pitch: Double
        var yaw: Double
    }
    var heading: Double // in degrees, where 0 is north (true or magnetic depending on frame)
    ```

## Reference Frame

 * Magnetometer use in CMDeviceMotion can be controlled by setting its reference frame. Specify this when calling `startDeviceMotionUpdates`
    ```swift
    xArbitraruZvertical // the default, does not use magnetometer
    xArbitraryCorrectedZVertical // use magnetometer (if available) to correct yaw over time
    xMagnetic/TrueNorthZVertical // uses magnetometer for device position/heading in world
    ```
    * These last two may require the user to calibrate the magnetometer.
    * And for TrueNorth, location information (e.g. GPS/Wifi/Cellular) will also be required.
    * North frames are necessary for apps that use things like Augmented Reality.
    * To get `heading`, for example, you must use a `Magneticnorth` or `TrueNorth` reference frame.
    * always ckeck to make sure the reference frame you want is available on the device
    ```swift
    static func availableAttitudeReferenceFrames() -> CMAttitudeReferenceFrame
    ```
* Registering a block to receive Accelerometer data
    ```swift
    func startAcclerometerUpdatasToQueue(queue: OperationQueue,
                                    withHandler: CMAccelerometerHandler)
    typealias CMAccelerationHandler = (CMAccelerometerData?, Error?) -> Void
    ```
* Registering a block to receive Gyro data
    ```swift
    func startGyroUpdatasToQueue(queue: OperationQueue,
                            withHandler: CMAGyroHandler)
    typealias CMGyroHandler = (CMGyroData?, Error?) -> Void
    ```
* Registering a block to receive Magnetometer data
    ```swift
    func startMagnetometerUpdatasToQueue(queue: OperationQueue,
                                    withHandler: CMMagnetometerHandler)
    typealias CMMagnetometerHandler = (CMMagnetometerData?, Error?) -> Void
    ```
* Registering a block to receive DeviceMotion data
    ```swift
    func startDeviceMotionUpdatasToQueue(using: CMAttitudeReferenceFrame,
                                    queue: OperationQueue,
                                    withHandler: CMDeviceMotionHandler)
    typealias CMDeviceMotionHandler = (CMDeviceMotion?, Error?) -> Void
    ```
* queue can be an `OperationQueue()` you create or `Operation.main` (or currentQueue)
* Errors
    ```swift
    CMErrorDeviceRequiresMovement
    CMErrorTrueNorthNotAvailable
    CMErrorMotionActivityNotAvailable
    CMErrorMotionActivityNotAuthorized
    ```
## Rate of collecting data
```swift
var accelerometerUpdateInterval: TimeInterval
var gyroUpdateInterval: TimeInterval
var magnetometerUpdateInterval: TimeInterval
var deviceMotionUpdateInterval: TimeInterval
```

## Accelerometer Over Time
* Historical Accelerometer Data

    You just want to know what happened over a period of time in the past. For example, if you want an idea of the user's physical movement pattern. The class `CMSensorRecorder` can record (at 50hz) and then play back accelerometer data. Not all devices are capable of this.
    ```swift
    isAccelerometerRecordingAvailable() -> Bool
    ```
    * start recording data
    ```swift
    func recordAccelerometer(forDuration: TimeInterval) // keep this short for performance
    ```
    * retrieving the recorded data
    ```swift
    func accelerometerData(from: Date, to: Date) -> CMSensorDataList // 3 day max
    ```
    * You enumerate over the CMAccelerometerData objects in a CMSensorDataList with `for in`...
    ```swift
    for dataPoint: CMRecordedAccelerometerData in sensorDataList {...}
    ```
* Activity Monitoring

    rough estimate of what the user is doing: For example, `stationary, walking, running, automotive, cycling`. You track this with a `CMMotionAcitivityManager` (not a CMMotionManager!)
    ```swift
    func startActivityUpdates(to: OperationQueue, withHandler: (CMMotionActivity?) -> Void)
    ```
    CMMoitonActivity is one of the above activities.
    * you can also query historical activity with
    ```swift
    func queryActivityStarting(from: Date, to: Date, to: OperationQueue, withHandler: ..)
    ```

## Pedometer
the user's "step" informationß

* Create a `CMPedometer` and then send it the message
    ```swift
    func startUpdates(from: Date, withHandler: (CMPedometerData?, Error?)-> Void)
    ```
    * The `from` Date is allowed to be in the past (7days max)
    * your hadnler will be called periodically with the struct `CMPedometerData` which has 
        * `startDaate` and `endDate` of the data
        * `numberOfSteps,distance,averageActivePace` and `currentPace` during the time, also `floorsAscended` and `floorsDesceneded`

## Altimeter
Get relative altitude changes.
```swift
func startRelativeAltitudeUpdates(to: OperationQueue, withHandler: (CMAltitudeData?, Error?))
```
`CMAltitudeData` has both change in altitude in meters and raw atomospheric pressure data.

## Authorization
some information is considered "private" to the user.

Specifically CMPedometer, CMSensorRecorder, CMMotionActivityManager and CMAltimeter.

iOS will automatically ask the user (once) for permission to access this information. You can find out what the status is at tany time with this static func on each of these.

```swift
static func authorizationStatus() -> CMAuthorizationStatus
struct CMAuthorizationStatus {
    case notDetermined
    case restricted
    case denied
    case authorized
}
```
Lack of authorization may also show up as ana error when you request data.

# Camera
Modal view controler to get media from camera or photo library.
`UIImagePickerController`

You put it up with 

```swift
present(_: aniamted: completion:)
```
## Usage
1. Create it & set its delegate (it can't do anything without its delegate)
2. Configure it (source, kind of media, user edibility, etc.)
3. Present it 
4. Respond to delegate methods when user is done/cancels picking the mediaß

* What the user can do depends on the platform

    * almost all devices have cameras, but some can record video, some can not.
    * You can only offer canmera or photo library on oPad (not both together at the same time) 
    * As with all device-dependenet API, we want to start by check what's available
        ```swift
        static func isSourceTypeAvailable(sourceType: UIImagePickerControllerSource Type) -> Bool
        ```
        Source type is `.photoLibrary` or `.camera` or `.savedPhotosAlbum` camera roll

* But don't forget that not every source type can give video

    so you then want to check
    ```swift
    static func availableMediaTypes(for: UIImagePickControllerSourceType) -> [String]?
    ```
    will return one or more of these
    ```swift
    // declared in the MobileCoreServices framework.
    import MobileCoreServices
    // ...

    kUTTypeImage    // almost all sources provide this
    kUTTypleLivePhoto   // must also say kUTTypeImage for this one to work
    kUTTypeMovie    // audio and video together, only some sources provide this
    ```
* You can get even more specific about cameras (usually not necessary)
    ```swift
    static func isCameraDeviceAvailable(UIImagePickerControllerCameraDevice) -> Bool
    ```
    returns `.rear` or `.front`
    * There are other camera-specific interrogations too, for example
    ```swift
    static func isFlashAvailableForCameraDevice(UIImagePickerControllerCameraDevice) -> Bool
    ```

* set the source and media type you want in the picker

    Example setup of a picker for capturing video (kUTTypeMovie)
    ```swift
    let picker = UIImagePickerController()
    let mediaTypeMovie = kUTTypeMovie as String
    picker.delegate = self // self must implement UINavigationControllerDelegate too
    if UIIPC.isSourceTypeAvailable(.camera) {
        picker.sourceType = .camera
        if let availableTypes = UIIPC.availableMediaTypesForSrouceType(.camera) {
            if availableTypes.contains(mediaTypeMovie) {
                picker.mediaTypes = [mediaTypeMovie]
                // proceed to put the picker up
            }
        }
    }
    ```
* Editability
    ```swift
    var allowsEditing: Bool
    ```
    If true, then the user will have opportunity to edit the image/video inside the picker. When your delegate is notified that the user is dont, you'll get both raw and edited versions.
* Limiting Video Capture
    ```swift
    var videoQuality: UIImagePickerControllerQualityType
    .typeMedium // default
    .typeHigh
    .type640x480
    .typeLow
    .typeIFrame1280x720 // native on some devices
    .typeIFrame960x540  // native on some devices

    var videoMaximumDuration: TimeInterval  // default to 10 mins
    ```
* Present the picker
    ```swift
    present(picker, animated: true, completion: nil)
    ```
    * On iPad, if you are not offering Camera (just photo library), you mus present with popover.
    * If you are offering the camera on iPad, then full-screen is preferred.
    * Remember: on iPad, it's Camera OR Photo Library(not both at the same time)

## done?
* Delegate will be notified when user is done
    ```swift
    func imagePickerController(UIIPC, didFinishPickingMediaWithInfo info: [String: Any]) {
        // extract image/movie data/metadata here from info
        picker.presentingViewController?.dismissViewControllerAnimated(true) { }
        // dismiss in new SwiftLang
    }
    ```
* Also dismiss it when cancel happens
    ```swift
    func imagePickerControllerDidCancel(UIIPC){
        picker.presentingViewController?.dismiss(true)
    }
    ```
## what is in that info dictionary
```swift
UIImagePickerControllerMediaType    // kUTTypeImage, kUTTypeMovie
UIImagePickerControllerOriginalImage    // UIImage
UIImagePickerControllerEditedImage  // UIImage
UIImagePickerControllerImageURL     // URL (in a temp location, so move it to keep it)
UIImagePickerControllerCropRect     // CGRect (in an NSValue)
UIImagePickerControllerMediaMetadata    // Dict of info about the image
UIImagePickerControllerLivePhoto    // a PHLivePhoto
UIImagePickerControllerPHAsset      // a PHAsset (see PHPhotoLibrary)
UIImagePickerControllerMediaURL     // URL of the video if kUTTypeMovie
```

## save to photo library
```swift
func UIImageWriteToSavedPhotosAlbum(
    _ image: UIImage
    _ target: Any?, // the object to send selector to when finished writing
    _ selector: Selector?, // selector to send to target twhen finished writing
    _ context: UnsafeMutableRawPointer? // passed to the selector
)
```
This is a simple and conventient way if the user only occasionally would want to save an image. Otherwise, you'll want to integrate with the Photos application: checkout `PHPhotoLibrary` - highly efficient and super powerful API. 

## More sophosticated media capture
check out  `PHPhotoLibrary` and `AVCaptrueDevice`

## Overlay view
```swift
var cameraOverlayView: UIView
```
be sure to set this view's frame properly/

Camera is always full screen, so use `UIScreen.main`'s `bounds` property.

If you use the built-in controls at the bottom, you might want your view to be smaller.

* Hiding the normal camera controls(at the bottom)
```swift
var showsCameraControls: Bool
```
will leave a blank area at the bottom of the scrren (camera's aspect 4:3)

With no controls, you'll need an overlay view with a "take picture" (at least) button.

That button should send `takePicture()` or (`startVideoCapture()`) to the picker.

* Don't forget to `dismiss` the modal view controller when you are done taking pictures.

# Core Image and Vision
* Core Image is a power ful and efficient framework for applying filters to your images.

    * Has a couple of hundred filters to choose from (blur, depth, comparison, colors, smoothing, etc.).
    * Core Image also has some feature detection API
* Vision framework provides powerful feature detection in images (e.g. faces, barcodes, etc.).
