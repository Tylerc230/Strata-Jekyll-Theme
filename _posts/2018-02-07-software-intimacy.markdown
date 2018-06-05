---
layout: post
---
<img src="/images/fulls/IMG_0100.jpg" class="fit image">
_Enlightenment is intimacy with all things._ —Ehei Dogen (1200-1253)

What is intimacy in software? It’s a funny concept which I’ve wanted to write about for some time. Like interpersonal intimacy, it’s hard to define. So, instead I will illustrate it with a few examples.

The first time I lacked intimacy with a codebase was on a project in which I had been asked to do some backend work. The server engineers set up my PHP environment and, to my chagrin, they weren’t using any type of debugger. Their process was to make a change locally, push it to develop and then test the feature using curl. If it didn’t work as expected, they would add some print statements, ssh into the server and tail the logs. Ouch! I felt like I was observing the code through a straw.

With a setup like this, it is hard to be sure what the server is doing at any given moment. I could only observer the values for which I had had the foresight to print beforehand. Many debugging insights occur by chance, through observing values in the debugger. All the properties look normal, but “oh wait, this property over here looks off” which leads us in a new direction. These chance observations are less likely to happen if we are required to add print statements apriori. Limited debugging combined with the tedious, ‘edit -> upload -> test -> tail the logs’ cycle means that we are much less likely to find the root cause of an issue, and more likely to add band aid fixes.

Once we were able to debug a running process over a socket, and could watch the program as it ran, our feeling of intimacy with the project grew. We could see the details of the running program more clearly now. By watching a program run in a debugger we learned how the program worked more innately. This kind of intimacy is especially important when learning a new codebase.

The next change to my process that increased my intimacy with a codebase was when I added unit testing to my toolkit. Using the debugger allows us to view a process as it runs; similarly unit testing allow us to see how our code behaves under different conditions. I like to think of my test suite as a little laboratory where I can test my subjects under tightly controlled conditions. Not only do I know how my code behaves under normal circumstances, now I know how it behaves under error cases and other obscure scenarios. I also know that all the assumptions I’ve made about the code still hold true after a change. Knowing that things still behave properly after a big refactor is a huge confidence booster.

The most recent change I’ve made to my process, and one I’m still experimenting with, is doing UI development in Playgrounds. The ability to observe the effect of a code change immediately in the interface is very powerful. With a properly factored UI, it is possible to inject all sorts of data and see the effect in real time.

My normal flow when building UIs was to do layout in interface builder, compile, run, navigate to the screen I was building and observe my changes. If I wanted to test an edge case, I would hardcode some data and feed it to the view. Once I had verified that the UI behaved as expected, I would remove the mock data. Adding hardcoded values is tedious and not easily reproducible.

Now, I load my view in a playground, along with different sets of mock data, and can instantly see my changes as they’re made. This technique is a great way to observe how the UI will behave under different conditions. It also speeds up the UI development process since I don’t have to navigate to the screen I’m building. Playgrounds are still in their infancy but in the future, they will be a useful development tool in additions to being a great learning device.

It isn’t possible to observe what a computer is doing directly, so developers have created powerful tools to see how things behave at runtime. By using these tools we can become more intimate with our software. This allows us to easily debug issues as they arise and refactor aggressively to prevent them from appearing in the first place.

A lack of intimacy motivates developers to make smaller local changes in order to fix issues instead of making larger refactors which will make the codebase more coherent. Knowing intimately how code behaves gives developers the confidence to make larger changes that improve the codebase, making it more maintainable.

The take home message is to embrace tools that help you understand what your software is doing and integrate them into your daily process..
