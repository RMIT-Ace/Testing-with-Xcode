# TDD UI Test (Continue)


## Repository

Source code project:
* https://github.com/RMIT-Ace/FoodTastingJournal

Branch:
* Tutorial/04-third-testfail
* Tutorial/05-third-testpass

# Testing UI: Adding a Map

Continuing from the previous post, the next feature we want to implement for our Food Journal app is a map view - something to help users discover food spots nearby. It's a natural fit for a tasting journal: not only do you want to record what you ate, but also where you ate it.

As always, we follow the Test Driven Development approach: write the test first, watch it fail, then write just enough code to make it pass.

Here's the test case for what we want:

```swift
@MainActor
    func testNearbyViewShowsMap() throws {
        let app = XCUIApplication()
        app.launch()

        let mapMarker = app.descendants(matching: .any)["NearbyMap"].firstMatch

        // Allow a short wait for the map to appear.
        let exists = mapMarker.waitForExistence(timeout: 5)
        XCTAssertTrue(exists, "Expected to find a map in NearbyView, but it was not present. Make sure the Map in NearbyView has accessibilityIdentifier 'NearbyMap'.")
    }
```

A few things worth noting here. We're using waitForExistence(timeout: 5) rather than checking the element's existence immediately - maps can take a moment to render, and this gives the UI a short grace period before failing. We're also using an accessibilityIdentifier ("NearbyMap") to locate the map view. This is a common and reliable pattern in UI testing: rather than searching for an element by type or label, we tag it with a known identifier and query for that directly.

Run the test, and of course, it will fail - the NearbyView doesn't contain a map yet, so there's nothing for the test to find.

# Making the Test Pass

Now it's time to implement the feature. Since this is an iOS app, we'll use MapKit - Apple's built-in framework for embedding maps. SwiftUI makes this refreshingly simple with the Map view, which handles rendering and basic interaction out of the box.

Here's our implementation:

```swift
import SwiftUI
import MapKit

struct NearbyView: View {
    
    var body: some View {
        NavigationStack {
            Map()
                .accessibilityIdentifier("NearbyMap")
                .navigationTitle("Nearby")
        }
    }
}
```

The key detail here is .accessibilityIdentifier("NearbyMap") - this is what ties our implementation to our test. Without it, the UI test would have no way to locate the map, even if it were rendering perfectly. It's a good habit to set accessibility identifiers on any significant UI element you plan to test.

Re-run the test, and it should now pass. Then re-run the entire test suite to make sure nothing was broken in the process. This last step is just as important as the first - passing one test while silently breaking another is a common pitfall.


# Getting the Hang of TDD

Starting to see the pattern? Write a failing test, implement just enough to pass it, then verify nothing else broke. Repeat.

It might feel like extra overhead at first, but TDD has a compounding benefit: every feature you add comes with a built-in safety net. The more tests you accumulate, the more confidently you can refactor or extend the app without worrying about unexpected regressions.

In the next post, we'll continue building out the Nearby view - there's a lot more we can do once the map is on screen.
