# Old vs New: XCTest vs SwiftTesting

When Apple introduced Swift Testing at WWDC 2024, it didn't deprecate XCTest - it gave us a choice.

For over a decade, XCTest has been the only game in town. It's battle-tested, it supports UI automation via XCUITest, and it works with Objective-C codebases that many teams still maintain. It's not going anywhere.

Swift Testing, on the other hand, is a clean-slate redesign built for modern Swift. Macros instead of naming conventions. A single #expect instead of a family of XCTAssert* functions. First-class parameterized tests, native tagging, and actor isolation baked in. It's clearly where Apple is investing.

So the question isn't which one is better - it's knowing when to reach for each one, and understanding what you're trading off when you do.

The table below maps them side by side across the features that matter day-to-day. 

**TL;DR:** if you need UI testing or Objective-C support, you still need XCTest. For everything else, Swift Testing is the more expressive and forward-looking choice — and both can coexist in the same project, so there's no need to migrate all at once.

| Feature | XCTest | Swift Testing |
|---|---|---|
| Since | Xcode 5+ (2013)| Xcode 16+, Swift 6  Modern (2024) |
| UI Testing | ✅ XCUITest | ❌ Not supported |
| Performance | ✅ measure {} | ❌ Not supported |
| Parameterized | ❌ manual code | ✅ Native arguments |
| Tags | ❌ Not supported | ✅ .tags() |
| Cross-platform | 🍏 Apple only | ✅ Linux / Windows |
| Objective-C | ✅ Full support | ❌ Swift only |
| Xcode Test Plan | ✅ Full support | ✅ Full (from XCode 16+) |
| SPM Support | ✅ Yes | ✅ Yes |
| Coexistence | ✅ Yes | ✅ Yes |
| Assertions | <pre><code>XCTAssertEqual(a, b)<br/>XCTAssertTrue(condition)<br/>XCTAssertNil(value)<br/>XCTAssertThrowsError(expr)<br/>XCTAssertGreaterThan(a, b)</code></pre>| <pre><code>#expect(condition)</code></pre> |
| Test Declaration | <pre><code>import XCTest<br/>class SomeUnitTests: XCTestCase {<br>  func testSample(){<br/>    XCTAssertEqual(1, 1)<br/>  }<br/>}</code></pre>|<pre><code>import Testing<br/>struct MathTests {<br/>  @Test func someFeature() {<br/>     #expect(2 + 2 == 4)<br/>  }<br/>}|
| Setup &<br/> Tear Down |<pre><code>override func setUp() { ... }<br/>override func tearDown() { ... }<br/>override func setUpWithError() throws { ... }</code></pre> |<pre><code>init() throws { /* setup */ }<br/>deinit { /* teardown */ }<br/>// Or actor isolation via<br/>// init() async throws</code></pre>|
| Organisation &<br/>Tags | Grouped via class hierarchy.<br/>No native tagging.<br/>Use test plans or naming conventions.|<pre><code>@Suite("Auth", .tags(.critical))<br/>struct AuthTests { ... }<br/>@Test(.tags(.slow, .network))<br/>func fetchProfile() { ... }</code></pre> |
| Concurrency | <pre><code>func testAsync() async throws {<br/>  let v = await fetchValue()<br/>  XCTAssertEqual(v, 42)<br/>}<br/>// Requires @MainActor for UI</code></pre> | <pre><code>@Test func asyncTest() async throws {<br/>  let v = await fetchValue()<br/>  #expect(v == 42)<br/>}<br/>// Actor isolation built-in</code></pre> |

"Two roads diverged in a wood, and I — I took the one less traveled by, and that has made all the difference." — Robert Frost

Ace