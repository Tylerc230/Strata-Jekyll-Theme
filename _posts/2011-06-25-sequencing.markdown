---
layout: post
title: Sleep Easy with Sequencing and Dependency Resolution
---
<img src="/images/fulls/IMG_0003.jpg" class="fit image">
Writing games is hard. Writing games for iOS is even harder. Typically, one of the most fragile and error prone parts of a game is the loading and initialization process. For obvious reasons, it is also one of the most critical pieces of your app. By using a sequencer, developers can define dependencies for steps in their loading process thereby removing some of the brittleness and instability from their game.

What does “game loading and initialization” mean exactly? I’m referring to everything that happens from the point when the user taps your game’s icon to the point when he is actually playing the game. The steps involved in loading a game can include any of the following:

*   Downloading new art and other resources.
*   Downloading user data.
*   Parsing user data.
*   Requiring user login.
*   Playing a loading video.
*   Loading textures into memory.
*   Loading user game data into memory.

Generally, some of these steps can be started in parallel such as downloading new art and playing a loading video. Many of these steps can take an arbitrary amount of time to complete depending on network speeds, amount of data being loaded etc. Further complicating the issue is that some of these events are dependent on other events happening first. For instance, loading textures into memory is dependent on having the new art downloaded from the server. Parsing user data is dependent on downloading that user data which, in turn, is dependent on user login. And of course actually starting the game is dependent on all of these steps being complete.

Ordinarily, a developer would attempt to resolve these dependencies by placing the dependent code into the “didFinish*” callback of its dependencies. This is inflexible since it’s difficult to add more dependencies to a step, difficult for new developers on the team to see the dependency, and hard to account for various dependencies being resolved in different orders (ie. sometimes the new art will download before the video has completed, sometimes it will take longer).

By using a sequencer, a developer is able to explicitly state, in one place, all the dependencies a certain step requires. If a step requires an additional dependency, all the developer has to do is add another enum to his list of dependencies and ensure that it gets resolved at some point in his code. Once all a game’s loading steps are entered into the sequencer, the developer can rearrange method calls, move steps to a secondary thread or change the location of a step in the code without fear of breaking the the fragile initialization order.

“Well that’s great” you say, “but where can I get such a wondrous piece of software?” Well fear not, dear reader. Your thoughtful author has decided to include this very package. The primary class, and the only one you’ll be interacting with directly is called Sequencer.m. The Sequencer class allows a developer to associate an action (ie. a method call), with a set of dependencies like so:

```objc
sequencer_ = [[Sequencer alloc] init];
[sequencer_ addTarget:self action:@selector(parseUserData) dependencies:LSLoggedIn, LSUserDataDownloaded, LSEnd];
[sequencer_ addTarget:self action:@selector(startGame) dependencies:LSLoggedIn, LSAssetsLoaded, LSUserDataDownloaded, LSEnd];
```


Once a dependent piece of code has been executed, we notify the sequencer that we have completed a step. The sequencer will then iterate over all of its entries and see if any of them have all of their dependencies resolved. If so it will execute that entry’s action method and remove it from its list of entries.

```objc
//at the end of the didLogin callback method
[sequencer_ resolveDependency:LSLoggedIn];
...
//in your connectionDidFinishLoading:  method after downloading your user data
[sequencer_ resolveDependency:LSUserDataDownloaded];
```

Once these 2 lines above have been executed, parseUserData will be called and the data will be parsed.

As you can see from the above example, all my dependencies are clearly defined in one place, the order in which the dependencies are completed doesn’t matter, and I can change the order that code executes in without breaking the dependency. With the Sequencer class in your programmer’s tool belt, you will sleep easy knowing that that one last change you made before pushing to production didn’t break your game’s setup process.

You can get the latest bits at [https://github.com/Tylerc230/sequencer_blog_1](https://github.com/Tylerc230/sequencer_blog_1 "Github")

If you’re looking for an iOS developer for your next project you, can find out more about my work at [www.casselmanconsulting.com](http://www.casselmanconsulting.com).
