### App lifecycle
* return quickly from `applicationDidFinishLaunching`
    * move long running work to background queues
* launch quickly again
    * shrink memory footprint when your app enters the background to keep your app in memory
* official advice = target two most recent major releases of iOS
* include version fallbacks
    * use `#available(iOS 9.0, *) {_}` syntax in Swift

### Views and view controllers
* Layout on modern devices
    * avoid hard-coded layout values

#### Size classes
* Stop thinking about orientation
* Some size thresholds trigger major change
    * e.g. single column to double column
* packaged in `UITraitCollection`

#### Making timing deterministic
* Leverage `UIViewControllerTransitionCoordinator`
    * animate alongside a transition
    * get accurate completion timing, cancellation

### Autolayout
#### Managing constraints efficiently
* Identify constraints that get changed/added/removed
* Unchanged constraints are optimized
* Avoid removing all constraints
* Use explicit references to constraints

#### Constraint specificity
* Duplicate constraints create extra work
* Create flexible constraints
    * Avoid hard-coded values
    * Describe constraints using bounds
* Fully specify constraints
    * Underspecification generates ambiguity
* Testing and debugging
    * `UIView hasAmbiguousLayout]` (useful on `UIWindow`)
    * call `_autolayoutTrace` to find ambiguous constraints
    * consider doing this in a unit test

### Table and collection views
* Use self-sizing cells (iOS8+)
    * Fully specify constraints in cell
* If this isn't working, try adding a height constraint to your content
* Animating height changes
    * `tableView.beginUpdates`
    * Update model
    * Update cell contents
        * don't need to call `reloadRow:atIndexPath`
    * `tableView.endUpdates`
* Fast collection view layout invalidation
    * floating header in photos app
    * invalidate on bounds change
    * build targeted **invalidation context** for header view
    * `UICollectionViewInvalidationContext`
