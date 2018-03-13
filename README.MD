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

