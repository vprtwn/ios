(WWDC15 – 226)
Dave DeLong – frameworks engineer, primary engineer on WWDC app

### Core Concepts
#### NSOperationQueue
* High-level `dispatch_queue_t`
* Cancellation
    * somewhat tricky to cancel dispatch blocks
* `maxConcurrentOperationCount`
* Long running task – can be longer than dispatch blocks

#### Lifecycle of an NSOperation
* Pending -> Ready -> Executing -> Finished
    * At any point can -> Cancelled

#### Cancellation
* Cancelling just flips the `cancelled` property
* Subclasses dictate meaning of cancellation
* Susceptible to race conditions

#### Readiness
* First operation to enter become ready = first to execute
* An operation becomes ready when all of its dependencies become ready

#### Dependencies
* Beware of operation deadlock when setting up dependencies
* In WWDC app
    * Settings -> version -> other info -> save MOC
    * Dependencies can guarantee correct ordering
* Network communication is hidden behind operations
    * Migrating backend to CloudKit was easy

### Beyond the Basics
#### UI Operations
* Authentication -> NSOperation
* Watching video -> Watch video operation
* all modal UI encapsulated in NSOperation

#### Block Operations
* Dependencies for blocks
* Seems like Apple is set on operations as asynchrony primitive (over promises)

#### Generating Dependencies
* Save Favorite generates dependencies:
    * user info
    * CloudKit access

#### Extending readiness concept
* Network unavailable = not ready

#### Composing Operations
* Downloading and parsing
    * Import Data op
        * Download op -> Parse op

#### Mutual Exclusion
* How to guarantee:
    * One alert at a time
    * One video at a time
    * Only load database once
* Use cross-queue dependencies 

### Sample Code
* Operation base ckass
* Other classes:
    * NSURLSessionTask -> operation wrapper class
    * Alert operation

#### OperationCondition
* Generates dependencies
* Defines mutual exclusivity
* Checks for satisfied conditions
* Examples
    * MutuallyExclusive<T>
    * Reachability
    * Permissions

#### OperationObserver
* Notified about events:
    * start
    * end
    * operation generation
* Examples:
    * timeouts
    * background tasks
        * watches state of UIApplication, automatically begins background task when app enters background
    * network activity indicator

#### Summary
* Read Threading Programming Guide


