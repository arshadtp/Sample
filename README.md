# Drive_AI

![Pod Version](https://img.shields.io/cocoapods/v/SVProgressHUD.svg?style=flat)
![Pod Platform](https://img.shields.io/cocoapods/p/SVProgressHUD.svg?style=flat)
![Pod License](https://img.shields.io/cocoapods/l/SVProgressHUD.svg?style=flat)

`DriveAI` detects the driving behaviour of a user by analyasing device sensors using our AI engine.

## Installation

### From CocoaPods

[CocoaPods](http://cocoapods.org) is a dependency manager for Objective-C, which automates and simplifies the process of using 3rd-party libraries like `Drive_AI` in your projects. First, add the following line to your [Podfile](http://guides.cocoapods.org/using/using-cocoapods.html):

```ruby
pod 'Drive_AI'
```



Second, install `Drive_AI` into your project:

```ruby
pod install
```

## Usage

Configure API key in app's main plist.
```swift
DriveAIAPIKey = <YOUR-API-KEY>
```
#### Initialize Drive_AI
Import module
```swift
import Drive_AI
```
Configure Drive AI
```swift
DriveAI.shared.configure()
```
>>> This method will initialize sensor data capturing if the trip is already started. If trip is not started yet, then this method will start listening for auto start/ stop trip based on the flag ```enableAutoStartAndStop```. This flag is true by default. For this feature to work user should grand ```Always``` location access for the app.

At this point you should be able to build the code without errors. Running the code will result in an ```assertionFailure``` with error ```Please add 'location' in required background modes```.

Enable Location in Backround modes.
```Target -> Capabilities -> Background Modes -> Location Updates```

Add code below code in ```application(_ application:, didFinishLaunchingWithOptions launchOptions:)```

```swift
    if let optn = launchOptions, optn[UIApplicationLaunchOptionsKey.location] != nil {
    	DriveAI.shared.configure()
    }
```
>>> This is required to get the location updates when the system launches the app due to a location update.

Builds and run the code. You are all done.

##### Disable Auto Start/ Stop

Set ```enableAutoStartAndStop``` to false
```swift
DriveAI.shared.enableAutoStartAndStop = false
```
##### Manual Start/ Stop

###### Authorize for location Access if not authorized yet
Create An object for location `LocationAuthoriser`
```swift
    lazy var locationAuthoriser: LocationAuthoriser = {
		return LocationAuthoriser()
	}()
```
Request for location Access
```swift
		locationAuthoriser.authoriseAppToUseLocation(progress: { (status, error) in
				print("in progress")
		}) { (status, error) in
			if let error = error {
					print("error: \(error)")
			} else if status == .authorizedWhenInUse || status == .authorizedAlways  {
					// Call start trip
			} else if  status == .denied {
					// Promt user to enable service
					print("Error: Denied location access")
			}
		}
```
###### Start trip

Call startTrip function
```swift
        DriveAI.shared.startTrip { [weak self] (response, error) in
            if let error = error, error != DriveAIError.tripAlreadyStarted {
				print("error: \(error)")
            } else {
                // Do your configurations
           		print("Trip started")
            }
         }
```
>>> The function will call server API to start trip if the current trip state is `.tripNotStarted` or `.tripEnded`. ie, will start new trip only if a the previous trip is ended. Method with invoke API for recording Sensor details if the API fails or a trip is aleady in progress.

###### Stop trip
Call stopTrip function
```swift
        DriveAI.shared.stopTrip(completion: { (response, error) in
            if let error = error {
                print("error: \(error)")
            } else {                       	
                print("Success")
             }
         })
```
>>> Method will call server API to end trip. This API may take little longer to complete since it will wait for upload task which syncs all recorded sensor value to sever to finish.
