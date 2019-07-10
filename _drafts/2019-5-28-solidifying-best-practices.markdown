With so many iOS architectures out there these days, I was sure that one would
speak to me. I've tried Viper, MVVM, ReSwift and others. They each had their
strong points but ultimately their weaknesses limited their usefulness. So what
did I do?? Invent one more architecture! And what is the worst name I could come
up with for it? Glad you asked! "Yet Another iOS Archtecture" or YAIA
(pronounced 'Yay-yeah!'). An architecture should have goals, otherwise its just
a collection of arbitrary rules which is no fun. Below I'll describe some of the
values which motivated YAIA.

1) I want writing unit tests to be painless. This is the main goal of YAIA and
everything else falls out of that. I'd like to write test without mocks, fakes
or do any kind of injecting of anything. I'd like to avoid having to test async
code and I certainly don't want to do any IO such as reading files or hitting a
database in my tests. I'd like my test to run as quickly as possible. 
2) It should be flexible and light weight. It doesn't require implementers to
create classes X, Y and Z _and_ protocols for each of those classes (I'm looking
at you, Viper!). 
3) Changes should be easy to make. This is related to points one and two. I
don't want to discourage users from making improvements because they'll have to
make changes in a lot of places or might break something in the process. 
4) Simple. No unnecessary abstractions or classes. Abstractions come at
a cost and if they don't pay for themselves, they're out.

To achieve these goals I cherry-picked the features from a few different
architectures, that had shown demonstrable value. From ReSwift,
an implementation of the redux pattern, I borrowed the emphasis on value types
and keeping data and logic in one place. In YAIA, the models that represent the business
and app logic are all value types (as opposed to reference types). The inputs
and outputs to this values-only system are also values making results easy to
verify. A nice side effect of a values only system is that its impossible for it
to contain any async code paths (a struct or other value type can't be captured
by reference in a block).

While there are many things to dislike about Viper, one of its tenants is that
dependencies should be hidden behind interfaces. Types from those dependencies
shouldn't creep into the business logic. What this means for YAIA, is that
NSManagedObjects from Core data, for instance, wont show up the in values-only
core. Instead, we copy data out of core data and into plain old swift structs
and pass those value types into our system. This allows us to test our system
without instantiating core data objects. It also means that we could
theoretically switch databases without too much pain since the core data
dependencies are contained; our business logic layer should be unaffected by a
database change.

I've described this architecture in previous posts but I'll go over the
highlights here. For a given screen in a mock, a login screen for example, the
class that ties the different elements together is called a `Scene` (naming
things is hard), in this example a `LoginScene`. It is similar to the
`ViewModel` in MVVM. In YAIA, it has a weak reference to the view controller via
a protocol This is one of the few places that a protocol is used in YAIA. The
view controller has a strong reference to the Scene so that the Scene's
lifecycle is tied to the view controller's. 

The `Scene` object contains a `State` object which is some sort of value type;
either a struct or enum generally. I will discuss the `State` object further but
for now just understand that it is where the bulk of the logic resides. The
`Scene` object may also have network gateways or database interfaces injected
into it. Routing is handled by a `FlowCoordinator`. It is the `Scene`'s
responsibility to take input from the UI, a button press for instance, and pass
it to the `State` object for processing. At this point, the `State` object might
update its internal state and, if the processing results in some sort of output,
such as saving an object to the database or making a network call, it will
return an enum indicating the IO to be executed by the appropriate interface. If
the `State` object determines a view transition is required, it will return an
enum representing that transition. To update the view, the `State` will return a
ViewModel instance. In YAIA, a ViewModel is a simple data transfer object that
contains all the info the view controller needs to update itself.

So how does YAIA achieve the goals listed above? Since the main
goal is to make unit tests easy, I had to figure out what configuration of code
is easiest to test. It turns out, if you separate the logic and state of the app
from the IO, the state/logic portion of the app becomes much more testable. The
easiest thing in the world to test is a function that takes values in and
returns a value. I wouldn't hesitate to write a test for `func add(a: Int, b:
Int)->Int`. It takes values as inputs and returns a value. It doesn't
have any internal state, doesn't read from a database and doesn't have any
dependencies that make the tests complicated. A test based on this method would
be easy to read and debug. 

The opposite end of the spectrum is a class which handles all of its IO
internally and has its logic and IO coupled together. Additionally, this class
will also be dependent on other large classes that are difficult to recreate in
a test environment or must be mocked. When testing these types of classes, the
code to set up the test end up being many times longer than the actual verify
step. All this setup means they are brittle and hard to read. By making the
`State` object a value type that doesn't do any IO we side step most of the
problems outlined above. These are the types of classes you end up with if you
don't start with testing in mind.

To achieve flexibility, I recommend using your best judgment about when to use
all the pieces above. If you have a screen with some text and a single button,
just put all the logic in the view controller and move on. When it comes to
architecture, one size does not fit all.

view models
