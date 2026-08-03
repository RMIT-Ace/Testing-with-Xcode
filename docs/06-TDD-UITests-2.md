# TDD UI Test (Continue)


## Repository

Source code project:
* https://github.com/RMIT-Ace/FoodTastingJournal

Branch:
* Tutorial/04-third-testfail
* Tutorial/05-third-testpass

# Testing UI Map

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


