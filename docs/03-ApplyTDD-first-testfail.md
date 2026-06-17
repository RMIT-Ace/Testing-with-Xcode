# Applying TDD - First "Test Fails"

## Introducing Food Tasting Journal

To put TDD into practice, we'll build a sample project: a Food Tasting Journal app. It logs visited restaurants with name, location, rating, and a short review, and uses the user's current location to surface nearby spots they've already tried.

## Sample Repository
Source code project:
* https://github.com/RMIT-Ace/FoodTastingJournal

Branch:
* Tutorial/01-first-testfail

## Writing First Fail Test Cases

The TDD approach forces us to think carefully about which features are most crucial to test first. We'll start by identifying what truly matters for the app:

* First, we need to get the user's current location.
* Then, list all the nearby restaurants the user has visited

```swift
import Testing
import Foundation

@testable import FoodTastingJournal

struct FoodJournalTests {
    
    @Test("Location manager provides current location")
    func getCurrentLocation() async throws {
        // TODO: to be implemented
    }
    
    @Test("Nearby restaurants are within the nearby distance.")
    func nearByRestaurantsWithinDistance() throws {
        // TODO: to be implemented
    }
}
```

These two initial test cases have opened Pandora's box. We now have to think about the app's internal implementation: how to retrieve and present a location, how to represent a restaurant, how to maintain a list of them, and so on.

A bit overwhelming?

Good news - with TDD, at this stage we only need to make these tests fail. Then we tackle satisfying them, one by one.

So, how do we make them fail?

With SwiftTesting, there are number of ways to force test failure.

| Method | Stops execution? | Use case |
| --- | --- | --- |
| Issue.record() | No | Manually flag failure, then keep gathering more info |
| #expect(false) | No | Force a failure but continue running assertions |
| #require(false) | Yes (throws) | Force a failure and immediately stopping tests |

So, now we can rewrite our test cases as follow.

```swift
import Testing
import Foundation
@testable import FoodTastingJournal

struct FoodJournalTests {
    
    static let alwaysFailed: Bool = 1 == 2
    
    @Test("Location manager provides current location")
    func getCurrentLocation() async throws {
        Issue.record("No test implemented")
    }
    
    @Test("Nearby restaurants are within the nearby distance.")
    func nearByRestaurantsWithinDistance() throws {
        #expect(Self.alwaysFailed, "To be implemented")
    }
}
```

Rerun the test cases. All tests fail successfully!

Excellent! We've just completed the first stage of TDD's Red, Green, Refactor cycle (see:  [Test Driven Development - TDD](02-TestDrivenDevelopment.md)). This is a great start — we now know exactly where to focus our development work.
In the next post, we'll move on to the next stage: making these test cases pass.

"To err is huamn" -- Alexander Pope,

Ace