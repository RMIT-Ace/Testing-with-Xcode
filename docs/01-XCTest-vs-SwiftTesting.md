# Old vs New: XCTest vs SwiftTesting

When Apple introduced Swift Testing at WWDC 2024, it didn't deprecate XCTest - it gave us a choice.

For over a decade, XCTest has been the only game in town. It's battle-tested, it supports UI automation via XCUITest, and it works with Objective-C codebases that many teams still maintain. It's not going anywhere.

Swift Testing, on the other hand, is a clean-slate redesign built for modern Swift. Macros instead of naming conventions. A single #expect instead of a family of XCTAssert* functions. First-class parameterized tests, native tagging, and actor isolation baked in. It's clearly where Apple is investing.

So the question isn't which one is better - it's knowing when to reach for each one, and understanding what you're trading off when you do.

The table below maps them side by side across the features that matter day-to-day. 

**TL;DR:** if you need UI testing or Objective-C support, you still need XCTest. For everything else, Swift Testing is the more expressive and forward-looking choice — and both can coexist in the same project, so there's no need to migrate all at once.

<table>
  <thead>
    <tr>
      <th>Feature</th>
      <th>XCTest</th>
      <th>Swift Testing</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Since</td>
      <td>Xcode 5+ (2013)</td>
      <td>Xcode 16+, Swift 6 (2024)</td>
    </tr>
    <tr>
      <td>UI Testing</td>
      <td>✅ XCUITest</td>
      <td>❌ Not supported</td>
    </tr>
    <tr>
      <td>Performance</td>
      <td>✅ measure {}</td>
      <td>❌ Not supported</td>
    </tr>
    <tr>
      <td>Parameterized</td>
      <td>❌ Manual code</td>
      <td>✅ Native arguments</td>
    </tr>
    <tr>
      <td>Tags</td>
      <td>❌ Not supported</td>
      <td>✅ .tags()</td>
    </tr>
    <tr>
      <td>Cross-platform</td>
      <td>🍏 Apple only</td>
      <td>✅ Linux / Windows</td>
    </tr>
    <tr>
      <td>Objective-C</td>
      <td>✅ Full support</td>
      <td>❌ Swift only</td>
    </tr>
    <tr>
      <td>Xcode Test Plan</td>
      <td>✅ Full support</td>
      <td>✅ Full (from Xcode 16+)</td>
    </tr>
    <tr>
      <td>SPM Support</td>
      <td>✅ Yes</td>
      <td>✅ Yes</td>
    </tr>
    <tr>
      <td>Coexistence</td>
      <td>✅ Yes</td>
      <td>✅ Yes</td>
    </tr>
    <tr>
      <td>Assertions</td>
      <td>
        <pre><code>XCTAssertEqual(a, b)
XCTAssertTrue(condition)
XCTAssertNil(value)
XCTAssertThrowsError(expr)
XCTAssertGreaterThan(a, b)</code></pre>
      </td>
      <td>
        <pre><code>#expect(condition)
// covers all cases above</code></pre>
      </td>
    </tr>
    <tr>
      <td>Test Declaration</td>
      <td>
        <pre><code>import XCTest
class SomeUnitTests: XCTestCase {
  func testSample() {
    XCTAssertEqual(1, 1)
  }
}</code></pre>
      </td>
      <td>
        <pre><code>import Testing
struct MathTests {
  @Test func someFeature() {
    #expect(2 + 2 == 4)
  }
}</code></pre>
      </td>
    </tr>
    <tr>
      <td>Setup &amp; Tear Down</td>
      <td>
        <pre><code>override func setUp() { ... }
override func tearDown() { ... }
override func setUpWithError() throws { ... }</code></pre>
      </td>
      <td>
        <pre><code>init() throws { /* setup */ }
deinit { /* teardown */ }
// Or actor isolation via:
// init() async throws</code></pre>
      </td>
    </tr>
    <tr>
      <td>Organisation &amp; Tags</td>
      <td>Grouped via class hierarchy.<br/>No native tagging.<br/>Use test plans or naming conventions.</td>
      <td>
        <pre><code>@Suite("Auth", .tags(.critical))
struct AuthTests { ... }
@Test(.tags(.slow, .network))
func fetchProfile() { ... }</code></pre>
</td>
</tr>
<tr>
<td>Concurrency</td>
<td>
<pre><code>func testAsync() async throws {
let v = await fetchValue()
XCTAssertEqual(v, 42)
}
// Requires @MainActor for UI</code></pre>
</td>
<td>
<pre><code>@Test func asyncTest() async throws {
let v = await fetchValue()
#expect(v == 42)
}
// Actor isolation built-in</code></pre>
</td>
</tr>
  </tbody>
</table>

"Two roads diverged in a wood, and I — I took the one less traveled by, and that has made all the difference." — Robert Frost

Ace