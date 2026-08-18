# TDD UI Test (Continue)


## Repository

Source code project:
* https://github.com/RMIT-Ace/FoodTastingJournal

Branch:
* Tutorial/06-TDD-iteration-4

# More Map & Location Testing

Having established the basic map view in the previous installment, it is time to push our test coverage further. A map that simply renders is not particularly useful on its own - what makes it genuinely valuable is the ability to show the user where they currently are. Knowing your position on the map is the foundation of any location-aware feature: it orients the user, helps them discover nearby places, and sets the stage for distance calculations and search filtering down the road.

Following the TDD discipline, we write the test first before a single line of implementation exists. This keeps us honest about what the feature actually needs to do, and gives us a clear, automated signal for when we have got it right.

We can express the requirement in XCTest language as follows:

```swift
@MainActor
    func testNearbyMapShowsCurrentLocationPin() throws {
        let app = XCUIApplication()
        app.launch()

        // The pin must be inside the map.
        let mapMarker = app.descendants(matching: .any)["NearbyMap"].firstMatch
        XCTAssertTrue(mapMarker.waitForExistence(timeout: 5), "Expected to find the map in NearbyView before checking for the pin.")

        // Look for the current-location pin by its accessibility identifier.
        let currentLocationPin = mapMarker.descendants(matching: .any)["CurrentLocationPin"].firstMatch

        // Allow a short wait for the pin to appear on the map.
        let exists = currentLocationPin.waitForExistence(timeout: 10)
        XCTAssertTrue(exists, "Expected to find a pin for the current location on the map, but it was not present. Make sure the current-location annotation has accessibilityIdentifier 'CurrentLocationPin'.")
    }
```

The test first confirms the map itself is present - there is no point looking for a pin inside something that does not exist. It then searches for a descendant element whose `accessibilityIdentifier` is `"CurrentLocationPin"`, giving it up to ten seconds to appear. The generous timeout accounts for the fact that Core Location can take a moment to produce its first fix, especially on a freshly launched simulator. Notice how the error messages are phrased to guide a developer who encounters a failure: they explain not just *what* went wrong but *what to do about it*.

Run the test now, and it will fail - exactly as expected. The red result is useful information: it confirms the test is genuinely checking something that does not yet exist in the app. Now we can write the code to make it green.

# Implementing Changes to Satisfy the Test

## Location Manager

The first thing the app needs is a dedicated object responsible for obtaining and broadcasting the device's position. It is good practice to isolate location logic in its own class rather than scattering it across views, because location permissions and updates have their own lifecycle that is best managed in a single place.

We tap into `CLLocationUpdate.liveUpdates()`, an async sequence introduced in iOS 17 that continuously emits fresh location fixes. Using a `for try await` loop, we process each update as it arrives: if a valid location is present we store it, and if the user has denied location permission we log a warning and exit the loop gracefully rather than leaving it running in the background.

```swift
// File: LocationManager.swift

func startUpdatingCurrentLocation() async {
        do {
            // Start continuous updating of location.
            for try await update in CLLocationUpdate.liveUpdates() {
                if let location = update.location {
                    currentLocation = location
                } else if update.authorizationDenied {
                    print(">> WARNING: Location access denied")
                    break
                }
            }
        } catch {
            print(">> WARNING: Location updates failed: \(error.localizedDescription)")
        }
    }
```

By catching errors at this level, we ensure that a failure in the location stream does not propagate up and crash the app. In a production setting you would surface this to the user more gracefully - for example, by setting a published error state - but a console warning is sufficient for our testing purposes here.

## Create a LocationManager Instance

With the `LocationManager` class in place, we need to create a single shared instance and inject it into the SwiftUI environment so that any view in the hierarchy can access it without needing to pass it down manually through every layer.

The right place to do this is at the very top of the app - in the `@main` entry point. By creating the instance in the `App` struct's `init()` and injecting it via `.environment()`, we make it available as an `@Environment` value to every view the app presents.

```swift
import SwiftUI

@main
struct FoodJournalApp: App {
    
    let locationManager: LocationManager
    
    init() {
        locationManager = LocationManager()
    }
    
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .environment(locationManager)
    }
    
}
```

This pattern - creating a dependency once at the top and distributing it through the environment - is clean and testable. In UI tests you could substitute a mock `LocationManager` that returns a fixed coordinate, giving you full control over what the pin displays without relying on actual device GPS.

## Update NearbyView

Now that `LocationManager` is available in the environment, `NearbyView` needs to pull it in and use it to both centre the map and render the current-location annotation.

First, we declare the environment dependency and a piece of state for the camera position:

```swift
struct NearbyView: View {
    @Environment(LocationManager.self) private var locationManager
    @State private var cameraPosition: MapCameraPosition = .automatic
    ...
}
```

Starting the camera at `.automatic` is a sensible default - MapKit will pick a reasonable initial framing, and we will move it ourselves as soon as we have a real location.

With MapKit's `Annotation`, we can mark the user's current position on the map. The annotation wraps a system image styled in red, and - crucially for our test - its accessibility identifier is set to `"CurrentLocationPin"`, which is exactly what the test is looking for.

```swift
            Map(position: $cameraPosition) {
                if let currentLocation = locationManager.currentLocation {
                    Annotation("Current Location", coordinate: currentLocation.coordinate) {
                        Image(systemName: "mappin.circle.fill")
                            .font(.title)
                            .foregroundStyle(.red)
                            .accessibilityIdentifier("CurrentLocationPin")
                    }
                }
            }
            .accessibilityIdentifier("NearbyMap")
            .navigationTitle("Nearby")
```

The conditional unwrap (`if let currentLocation`) means the annotation is only added when a location has actually been received - the map renders cleanly before the first GPS fix arrives, rather than crashing or displaying a stale placeholder.

Next, we kick off the location update stream when the view appears, using `.task` to tie the async work to the view's lifetime. When the view disappears, the task is automatically cancelled, stopping the stream cleanly.

```swift
            Map(position: $cameraPosition) {
                if let currentLocation = locationManager.currentLocation { ...  }
            }
            .accessibilityIdentifier("NearbyMap")
            .navigationTitle("Nearby")
            .task {
                await locationManager.startUpdatingCurrentLocation()
            }
```

Finally, we watch for changes to `currentLocation` and pan the camera to follow it. The `initial: true` flag means the camera move also fires immediately when the view first appears, so if a location is already available (for example, because another view already started updates) the map snaps to it straight away rather than waiting for the next update.

```swift
            Map(position: $cameraPosition) { ...  }
            .accessibilityIdentifier("NearbyMap")
            .navigationTitle("Nearby")
            .task { ...  }
            .onChange(of: locationManager.currentLocation, initial: true) {
                guard let currentLocation = locationManager.currentLocation else { return }
                setCameraPosition(to: currentLocation)
            }
```

Together, these three modifiers - `.task`, `.onChange`, and the `Annotation` inside the `Map` - form a tight loop: the view starts the stream, listens for updates, and immediately reflects each new fix both in the camera position and in the pin's coordinate.

## Preparing the Simulator for Map & Location Testing

One thing that cannot be done in a unit test - but is straightforward with the iOS Simulator - is feeding the app a realistic GPS fix. Physical location data is obviously not available inside a simulator, so Xcode lets you simulate it.

Open the Simulator app, then navigate to **Features > Location > Custom Location…** and enter the latitude and longitude of the location you want to test with.

![](res/map-location.png)

For this project, choosing a location near a food district is a nice touch - it makes the "Nearby" concept feel tangible when you see the pin drop in a realistic place. Once you set a custom location, the simulator will feed that coordinate to Core Location just as a real device would, and `CLLocationUpdate.liveUpdates()` will emit it to your app.

It is worth noting that custom locations persist across app launches within a simulator session, so you do not need to set it again every time you run the tests. If you switch simulators, however, you will need to configure the location again on the new device.

# Run All Your Test Cases

With all the implementation in place, run the complete test suite. Every test case should now pass - including the new `testNearbyMapShowsCurrentLocationPin` test that was failing just a short while ago.

This is the payoff of the TDD cycle: a suite of tests that each document a concrete behaviour, and production code that exists precisely because those tests demanded it. The next time a change is made to `NearbyView` or `LocationManager`, these tests will immediately flag any regression, giving the whole team confidence to refactor freely.