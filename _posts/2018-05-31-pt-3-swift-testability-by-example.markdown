---
layout: post
title: Pt. 3 Swift Testability By Example
---
<img src="/images/fulls/DSC01650.jpg" class="fit image">
In [my last article](/2018/05/08/pt-2-testable-swift/), I discuss the easiest path to testable Swift. In that article I list qualities that make tests valuable. Additionally, I show that business and application logic should be decoupled from volatile or asynchronous dependencies. Now I’d like to focus on the “State” object that houses all of this logic and illustrate some design decisions that will make it easier to test.  
An experienced tester knows that certain features are trivial to test while others must be mangled and distorted before they yield to testing. Tests that are easy to write are ones that require little or no arrangement code, don’t require new classes to enable testing (ie. mocks or stubs) and produce easily verifiable output. The easiest thing in the world to test is a pure function:

``` swift
let sum = addInts(3, 4)
XCTestAssertEqual(sum, 7)
```

We dream of tests that look like this. A pure function is one who’s return value is solely determined by its input parameters. Its internals don’t reference any global mutable data which can affect the return value. For a given input, it will always return the same output. Its output is another value, not a write to a database or network request. The output is a value that can be easily verified using the `==` operator.  
The next easiest thing to test is a mutable object that has internal state but doesn’t reference any volatile dependencies. To test these objects we must first construct the object, then mutate it until it is in the state we’d like to verify and finally check that certain conditions are true. An example of this would be:

```swift
var calculator = Calculator(startingValue:0)
calculator.press(key: .three)
calculator.press(key: .plus)
calculator.press(key: .four)
calculator.press(key: .equals)
let finalValue = calculator.currentValue
XCTestAssertEqual(finalValue, 7)
```

Again, this class (or struct) doesn’t make any network calls and doesn’t reference any global data; calling these methods, in this order, with these parameters will always result in the same final value. It’s not quite as simple as the previous example but it is still obvious what this test does and why.  
We only want to write tests that resemble these two forms. Unfortunately, most unit tests I’ve seen in Swift follow a different pattern.

```swift 
class MockAPIClient: APIClient {
    var error: Error?
    func make(request: APIRequest, onComplete complete:(Error?) -> ()) {
        complete(error)
    }
    //Add all other methods required by APIClient,
    //possibly stubbing return values to make
    //MockAPIClient behave like the real APIClient
}
class MockLoginDelegate: LoginViewModelDelegate {
    var showErrorBannerCalled: Bool = false
    func showErrorBanner(withMessage: String) {
        showErrorBannerCalled = true
    }
    //Stub all other LoginViewModelDelegate methods
}
let mockDelegate = MockLoginDelegate()
let apiClient = MockAPIClient()
apiClient.error = //some error
let viewModel = LoginViewModel(apiClient: apiClient)
viewModel.delegate = mockDelegate

viewModel.userTappedLogin(username: "steve_jobs", password: "i<3apple")
XCTestAssertTrue(mockDelegate.showErrorBannerCalled)
```


I avoid this style because it violates a number of rules outlined in my [previous article](http://www.sfsoftwareist.com/2018/05/08/testable-swift/).

*   First, it is long and it would be a lot longer if I had implemented every method in `LoginViewDelegate` and `APIClient`. Long tests force the reader to figure out what code is required by the test versus what is there just to keep the compiler happy. They are also brittle; every line must be maintained and can break if an interface changes. Besides, who wants to type that much.
*   In order for `LoginViewModel` to function properly, `MockAPIClient` must return the proper values from all its public methods. For instance, if `hasNetworkConnection: Bool` is declared in `APIClient`, we need to grok the implementation details of `LoginViewModel` in order to know that this method must return `true` for the tests to behave properly. Any time a test is coupled to implementation details like this, it’s sure to cause trouble.
*   In order to inject the `MockAPIClient`, we are forced to add the `APIClient` protocol which wouldn’t otherwise be there. This protocol needs to be updated whenever `RealAPIClient`‘s public interface changes. This forces all of the tests to be updated as well. Protocols add a level of abstraction that can make code harder to reason about; it’s unclear to the reader that `APIClient` will always be a `RealAPIClient` at runtime.

Converting the above to a more simplified form gives us the following:

``` swift 
let loginState = LoginState()
let error = /*some error*/
let commands: [LoginState.Command] = loginState.handleLogin(response: nil, error: error)
XCTestAssertEqual(commands, [.showErrorBanner("Failed to log in")])
```

Four lines of code, no mocks or stubs and no protocols added. Value types are passed in and value types are returned. The production code might look like the following:

``` swift
class LoginViewModel {
    var state = LoginViewState()
    let apiClient = APIClient()
    func loginTapped() {
        let commands = state.loginTapped()
        self.handle(commands: commands)
    }

    func handle(commands: [LoginViewState.Command]) {
        for command in commands {
            switch command {
            case .sendLoginRequest(let request):
                apiClient.send(request: request) { response, error in
                    let commands = self.state.handleLogin(response, error)
                    self.handle(commands: commands)
                }
            case .showErrorBanner(let message):
                self.ui.showErrorBanner(message)
            case .showLandingScreen:
                self.delegate.transitionToLandingPage()
            }
        }
    }    
}
```
_disclaimer: I have no idea if any of this compiles_  

I’ve left out some details for brevity but you can see how the logic of _what_ to do is embedded in the state object while _how_ to do it is found elsewhere. Of course, this is a toy example but you can imagine the state object will become more complex as requirements are added (eg. validation logic, error cases). This will be contained and kept separate from the IO represented by the `Command` objects. You can find more examples of these patterns [here](https://github.com/Tylerc230/CleanArchitectureSample).

In this article I’ve outlined some specific examples of code that is easily testable. I’ve also given some pointers on how to restructure code to be tested painlessly. Thanks for reading!
