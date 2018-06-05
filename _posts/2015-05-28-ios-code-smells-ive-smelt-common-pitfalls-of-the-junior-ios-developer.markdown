---
layout: post
title: iOS Code Smells I’ve Smelt — Common Pitfalls of a Junior iOS Developer
---
<img src="/images/fulls/IMG_0066.jpg" class="fit image">
Being a freelance iOS developer, I get the opportunity to work on a variety of projects, written by developers with varying degrees of experience. This has given me the opportunity to catalog the most common iOS anti patterns or [‘Code smells’](http://en.wikipedia.org/wiki/Code_smell "'Code Smell'") I’ve smelt. I will enumerate the most common smells and some possible solutions to guide you on your path to mobile app nirvana.

**MVC**

The most common error that junior developers make is constructing the ‘Massive View Controller’ and its cousin, the ‘Massive App Delegate’. People commonly place bits of logic in these two classes because they don’t know where else it should go. If this pattern isn’t curtailed early on, view controllers can become 400 (4000?) line behemoths. This is a sign of an unhealthy code base because the code isn’t easily reused, leading to code duplication and unnecessary dependencies. The other, more insidious problem is that, inevitably, your view controller will implement multiple systems in the same class, their logic will become hopelessly entangled and difficult to reason about. Attempting to <span class="Apple-converted-space"></span> modify one system can easily introduce bugs in another. <span class="Apple-converted-space"></span> Any given class should do only one thing and have a clean interface exposing its functionality.

To keep view controllers slim, remember they should only be responsible for transferring changes from the model layer to the views, or passing user input back to the model layer. View controllers should be the most boring classes in your project. Similarly the app delegate should simply relay system messages to the data layer. When it comes down to it, the reason these two classes become so bloated is that they are the only classes that Xcode generates automatically for you. The solution is to simply pull out a piece of paper and draw some boxes representing your classes. Then write down a list of all the public methods that each class exposes. After that, draw some arrows showing their dependencies to each other. Do this before writing a single line of code and your project will be more modular and flexible.

**APIs define explicit relationships between objects**

The second stinkiest smell I’ve smelt has many different manifestations but all share a common root. I’m going to call this one the ‘Lack of Well Defined APIs’ smell. People think an API is something you slap on top of a library right before you ship it. On the contrary, any object you create should have a carefully considered interface before work begins on the implementation. If your project has even two classes, each should have a well defined list of public methods describing the functionality of the class. For bonus points it should even have all of its dependencies exposed in the constructor (see [Dependency Injection](https://sites.google.com/site/unclebobconsultingllc/blogs-by-robert-martin/dependency-injection-inversion "Dependency Injection")). The approach new developers take is to dive right into the implementation and not consider how classes relate to one another.

There are many different forms the ‘ill defined API’ code smell can take. One example is a custom table view cell which exposes all of its subviews to the world. Users of this custom cell are expected to access the labels and buttons of this cell directly. Instead, all access to the subviews should be through an API. This makes the code easier to read, explains exactly what the cell does, and allows the developer to change the implementation of the cell without changing any other class. In the example below, I don’t need to change any code outside of the cell when I later decide to use use the ‘attributedText’ property instead of the ‘text’ property in the cell’s highScoreLabel.

```objc
//This is a leaky cell API
cell.highScoreLabel.text = [NSString stringWithFormat:@“%@ %d”, user.name, user.highScore];

//This method abstracts away the implementation of the cell, leading code that is easier to modify
[cell setName:user.name highScore:user.highScore];
```

Another common example of a poor API is when embedding a button in a cell, often developers will set the target/action pair on the button in the data source. My approach is to expose a block in the cell’s interface which gets executed when the button is tapped. This allows me to capture the data object that the cell represents in cellForRowAtIndexPath:.

```objc
//This is a leaky abstraction
[cell.favoriteButton addTarget:self forAction:@selector(toggleMarkFavorite:) forControlEven:UIControlEventTouchUpInside]
```

```objc
//A better alternative
cell.favoriteButtonTapped = ^(BOOL isFavorite) {
if (isFavorite) {
    [cellObject markAsFavorite];
} else {
    [cellObject unmarkAsFavorite];
}
```

In the same vain, modifying the UI by walking the view hierarchy, is a design anti pattern. Code that either iterates over subviews calling “`isKindOf:“` searching for a particular view, or searches up the hierarchy calling superview.superview, is a code smell. Making assumptions about the structure of a view hierarchy is extremely brittle and will easily break with simple changes to the view structure. I once worked on a project where a previous developer had referenced table view cells by calling superview.superview from an embedded subview. Once iOS 7 rolled out, apple changed the internal structure of the cell and ALL THE CODE BROKE. The solution is to add direct references (properties) to any views which need to be modified at run time through Interface Builder.

Another symptom of poorly considered object relationships is excessive use of NSNotifications. Notifications are, in and of themselves, not a bad thing and do have many legitimate uses. However using them to facilitate communication between two objects instead of a direct dependency is a code smell. At first glance, Notifications make writing code easier; changes to one object are easily propagated to other objects without defining an explicit relationship between them. The down side to this approach is that the relationship between these two objects isn’t obvious by looking at their public APIs. It’s better to make a direct relationship between two objects, either through a ‘has a’ relationship, the delegate pattern or a more specific observer pattern (blocks also facilitate interaction between objects but see my next point).

**Too much of a good thing**

When blocks were first introduced, I thought they were the best thing since the ‘for’ loop. I quickly discovered that, while blocks make asynchronous code easier to read, they became difficult to follow and debug as they become more deeply nested. The beauty of blocks is that you can capture the state of your program at any point and do operations on it later. The problem is that it can be difficult to determine the order that operations occur if too much of your code is executed in blocks. Don’t get me wrong, blocks are a good thing and are absolutely appropriate for many circumstances. However, whenever creating a block nested inside another block, ask yourself if there is a way to combine them. Alternatives to blocks are: the delegate pattern, an observer pattern, NSOperations or a Promises library.

**Polymorphic views**

Xib files make interfaces easy to assemble but can make reusing components difficult. This gives rise to the ‘overloaded view’ code smell. This happens when you define a custom view, such as a table view cell, in IB. Then elsewhere in the UI you see a similar view, not quite the same but similar, so you decide to reuse the same xib file, possibly passing in a ‘type’ enum to the cell to differentiate between the two. The view shows and hides various subviews depending on it’s ‘type’. These view ‘types’ seem to grow in number and are only superficially related to one another. Code paths which are unrelated accumulate in one place and become hopelessly entangled. The result is one class representing many views in your app which might share some minor commonalities but, in reality, have very little to do with one another. You know there is a problem when the view is littered with switch statements; adding and hiding views depending on some state. This is a problem because code written for one ‘type’ <span class="Apple-converted-space"></span> can inadvertently get executed for a completely separate type. It also makes the code harder to reason about and debug.

A better approach is to factor out the common components into a separate .xib file. The xib then gets added as a subview where needed.

**Just make it an object already**

A smell I see quite often, especially when working with RESTful APIs, is the ‘Dictionaries as Objects’ smell. JSON objects, received from the server, are parsed intro dictionaries and used directly throughout the app. The problem is that dictionaries are stringly typed; as in you have to use strings to access the values. This makes the code verbose and can lead to bugs if keys are misspelled. It is also impossible to attach functionality to a dictionary to process the data like you would with methods on classes. Additionally the kinds of values you can store in a dictionary are restricted (no primitive types). It is preferable to define classes for all JSON objects and convert the dictionary responses to objects as soon as you receive them. The advantage is that you can access properties using getters instead of keys, you can use primitive types as values and you can you can add functionality to the data if need be.

**Ya ain’t gonna need it**

Poor grammar but good advice. A lot of code I come across is littered with commented out code or worse, functions and classes that aren’t used anywhere but were left in the code base, collecting digital dust. The habit of developers to save code because ‘they might need it later’ is fairly pervasive. The problem with stale, unused code is that it still needs to be mentally parsed and understood when reading the code and still needs to be maintained when refactors occur. It is difficult to remove later because a method might appear to be used, but in reality might actually be called from an unused branch of code. If some functionality has been removed, just delete all the code associated with it. It will exist forever in version control <span class="Apple-converted-space"></span> in case it is needed in the future. Furthermore, code is cheap to write and expensive to maintain. No matter how brilliant you think your snippet of code is, just delete it, it could probably be recreated later if necessary.

**Caching considered harmful**

The final code smell I would like to discuss is the dangers of caching. Most developers have a good idea of what caching is; they could probably give an example involving local storage of downloaded images. It’s important to recognize the subtler forms of caching and how they can be detrimental. Some examples of subtle caching are: 1) Performing a calculation at startup and saving the result, 2) creating an object once and using it for the lifetime of the app, 3) fetching objects from Core Data and keeping them in memory, 4) pulling objects from the server and storing them in Core Data. The main problem with these sorts of caching is that now you have two places where the data is stored and they can become inconsistent. The complexity in determining when a cache is stale and needs to be refreshed can introduce bugs to a code base. Before deciding to cache or ‘memoize’ <span class="Apple-converted-space"></span> a value, be certain that you need the performance boost, otherwise you’re optimizing prematurely. This anti pattern crops up a lot with Core Data, or when constructing singletons (unless you truly have some state that needs to be maintained for the entire life of your app, it’s better to create new *Manager objects for every view controller that needs access to them). Also, consider keeping data pulled from the server in memory and refresh it when the app launches.

**Conclusion**

I’ve outlined the seven stinkiest code smells found in iOS projects. Taking the time up front to structure your code properly will always pay dividends down the road.Refactor old code, design clean APIs, create new classes when needed, avoid premature optimization and generally keep your project clean and lean, and your codebase will remain flexible and less error prone. New features will be added easily and old ones will be modified with little friction. The source of and solution to bugs found in your code will be apparent and easy to fix. If you can avoid these code smells, you’ll be miles ahead of your peers. Happy coding!
