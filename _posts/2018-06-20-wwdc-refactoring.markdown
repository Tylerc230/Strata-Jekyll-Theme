---
layout: post
title: WWDC Refactored
---
<img src="/images/fulls/IMG_0087.jpg" class="fit image">
Recently, I've been discussing ways to architect iOS applications to make them easier to test. Yesterday, I stumbled upon [this](https://developer.apple.com/videos/play/wwdc2017/414/) talk from WWDC '17. In this video, the presenter espouses a lot of the same ideas I've been advocating here. It's a great video and I highly recommend it.

There, the presenter refactors a method that might be seen in any application. The original method would be hard or impossible to unit tests due to its dependency on the `UIApplication` singleton. They refactor it using several techniques, including mocking a dependency, in order to make the logic testable. The result is a much more testable unit of code.
Here is the original method:
```swift
@IBAction func openTapped(_ sender: Any) {
  let mode: String
  switch segmentedControl.selectedSegmentIndex {
    case 0: mode = "view"
    case 1: mode = "edit"
    default: fatalError("Impossible case")
  }
  let url = URL(string: "myappscheme://open?id=\(document.identifier)&mode=\(mode)")!
  if UIApplication.shared.canOpenURL(url) {
    UIApplication.shared.open(url, options: [:], completionHandler: nil)
  } else {
    handleURLError()
  }
}
```

And here is the refactored version:
```swift
protocol URLOpening {
  func canOpenURL(_ url: URL) -> Bool
  func open(_ url: URL, options: [String: Any], completionHandler: ((Bool) -> Void)?)
}
extension UIApplication: URLOpening {
  // Nothing needed here!
}
class DocumentOpener {
  let urlOpener: URLOpening
  init(urlOpener: URLOpening = UIApplication.shared) {
    self.urlOpener = urlOpener
  }
  func open(_ document: Document, mode: OpenMode) {
    let modeString = mode.rawValue
    let url = URL(string: "myappscheme://open?id=\(document.identifier)&mode=\(modeString)")!
    if urlOpener.canOpenURL(url) {
      urlOpener.open(url, options: [:], completionHandler: nil)
    } else {
      handleURLError()
    }
  }
}
```
And this is how you would test it:
```swift
class MockURLOpener: URLOpening {
  var canOpen = false
  var openedURL: URL?
  func canOpenURL(_ url: URL) -> Bool {
    return canOpen
  }
  func open(_ url: URL,
      options: [String: Any],
      completionHandler: ((Bool) -> Void)?) {
      openedURL = url 
  }
}
func testDocumentOpenerWhenItCanOpen() {
  let urlOpener = MockURLOpener()
  urlOpener.canOpen = true
  let documentOpener = DocumentOpener(urlOpener: urlOpener)
  documentOpener.open(Document(identifier: "TheID"), mode: .edit)
  XCTAssertEqual(urlOpener.openedURL, URL(string: "myappscheme://open?id=TheID&mode=edit"))
}
```

This is a huge improvement over the original method. It allows us to test all the logic contained in `DocumentOpener`. It decouples the application singleton from the logic under test. My only objection is that the test is a little convoluted. As someone unfamiliar with the code, I need to examine the mock object to see how it works and I need to open up the `DocumentOpener` to understand how it interacts with the `MockURLOpener`. Additionally, the `URLOpening` protocol makes the production code harder to reason about. Was the protocol added to facilitate testing or did the writer truly intend for consumers of `DocumentOpener` to implement multiple `URLOpening` classes? What if we refactored this using some tips I've outlined in previous articles?
```swift
class DocumentOpener {
  enum Result {
    case openURL(URL), invalidURLError
  }
  let isURLValid: (URL) -> Bool
  init(isURLValid: (URL) -> Bool) {
    self.isURLValid = isURLValid
  }
  func open(_ document: Document, mode: OpenMode) -> DocumentOpener.Result {
    let modeString = mode.rawValue
    let url = URL(string: "myappscheme://open?id=\(document.identifier)&mode=\(modeString)")!
    if isURLValid(url) {
      return .openURL(url)
    } else {
      return .invalidURLError
    }
  }
}
```
Arguably this is a little nicer; we've removed a few lines of code and a type declaration. The real value comes when attempting to test this unit.
```swift
func validURL(_ url: URL) -> Bool {
  return true
}
func testDocumentOpenerWhenItCanOpen() {
  let documentOpener = DocumentOpener(isURLValid: validURL)
  let result = documentOpener.open(Document(identifier: "TheID"), mode: .edit)
  XCTAssertEqual(result, .openURL(URL(string: "myappscheme://open?id=TheID&mode=edit")))
}

```
As you can see, we are injecting a single method instead of a conforming object. Additionally, we are returning a value representing the action we'd like to perform. The advantage is that our test is reduced from 18 lines to 8, there is almost no arrange code, and the return value is easily verified.

This isn't always the best approach. If `DocumentOpener` had a lot of interaction with the system via `URLOpening`, we'd end up injecting a lot of methods into `DocumentOpener`. At this point it might make sense to inject a `URLOpening` object instead.

In conclusion, by avoiding mocks and focusing on value types, it is possible to write tests that are shorter and easier to understand down the road.
