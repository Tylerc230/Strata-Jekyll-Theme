---
layout: post
---
<img src="/images/fulls/DSC00695.jpg" class="fit image">
Developers, repeat after me, “My app is not a web page”. Again! “My app is not a wep page”. It seems that most developers are writing their mobile apps like they did web pages in the 1990’s. They makes requests to the server, put the app into a waiting state, and display the response in the UI. If they are offline, an error is reported to the user and the app is unusable. This experience is fine for a webapp in the browser with little or no local storage and a ubiquitous network connection, but falls short in the mobile world.

Mobile is an entirely different beast. Users have come to expect fast, responsive user interfaces and to be able to use their apps wherever they are. Additionally, these devices have vast amounts of local storage, first class databases and lots of processing power, allowing a richer user experience. Unfortunately they also have slow, flaky or absent network connections, making data synchronization to the server more difficult.

There are a few rules I like to follow when writing an app that interacts with a server.

1.  Any functionality that does not specifically rely on data from a server response should be available when offline. Why should my photo sharing app become unusable on a camping trip?

2.  UI should update immediately for any interaction that doesn’t specifically require a response from the server, even if it triggers a server request; users should not have to wait to upload a photo.

Fulfilling these requirements takes more engineering work but leads to a better user experience.

Following a ‘thick client’ model such as this is not easy. Developers need to worry about keeping data in sync with the server and how to handle network errors. The upside is that the experience feels more fluid; users won’t feel constantly ‘blocked’ by the ‘waiting…’ UI. Ideally the user is unaware that they are connected to a server at all. I’ve written a small client and server library to fulfill these requirements.

I’d like to introduce a iOS/RoR library I’ve written which allows users to create, update and collaborate on content while offline and sync it to a server when the network becomes available. It supports editing of items by several users and conflict resolution when 2 users edit something simultaneously. Content can be text or binary data.

The benefit of this library is that users can create and edit collaborative blog or picture posts while offline and sync changes to the server at a later date. The only time a user is aware of the server is when they receive a conflict during a content sync.

## How it works

### The model

There is a one to one correlation between ActiveRecord server models and Coredata managed object models (the sync_status property is the only exception). Each object in their respective databases has the following fields:

1.  sync_status – This field is only found in the Coredata model, not on the server. It has 3 states; Synced, NeedsSync and Conflicted. If it is in the Synced state, all of its modifications have been pushed to the server. The server may have newer modification which the client will receive during the next sync, but the client object has no local modification that the server doesn’t know about.  
    If the object is in the NeedsSync state, it means that the client has made local modifications to the object which have not been pushed to the server yet (or that the object is new and has never been synced to the server). Once the client syncs with the server the status will be updated to either Synced or Conflicted.  
    During a sync, if the server discovers that an object being synced has been modified by another client, it will send down the conflicting version of the object to the client which the client will mark as being Conflicted. Once the Conflict is resolved by the client, the resolution will be pushed to the server and the object will be marked as Synced.

2.  is_deleted – This flag is found on both the client and the server models. Since the server does not keep track of which clients have synced and has no knowledge of the number of clients it is supporting, it must keep a record of which objects have been deleted indefinitely. For this reason, objects are never removed from the database, even if a user ‘deletes’ the content. It is the client’s responsibility to never display objects with an is_deleted flag set.

3.  last_modified – This field is used by the server to determine if an object was modified since the last time a client pulled the data down. This field is only ever updated on the server. If the client attempts to push modifications to an object, and that object has a modification date which is earlier than the server’s version, the server rejects the client’s version and sends back the conflicting version which the client must then resolve.

4.  guid – When the client creates a new object, it is assigned a globally unique identifier. This number is used as the primary key to differentiate between 2 objects. GUIDs are essentially long random hex numbers. These numbers are so large and sufficiently random that the likelihood of 2 being identical is astronomically low; even if they are generated on 2 different devices. This is beneficial because it means 2 different devices can create objects in the database and they can be differentiated from each other.

### The sync process

When the client becomes connected to the network again, or when the user presses the ‘sync’ button, the modification date of the most recently modified object in its local database is passed to the server, along with any objects which are in the NeedsSync state (ie. has local modifications). The server looks at all the newly modified objects, if the last_modified timestamps are different, it knows that its version has changes which the client is unaware of. If the dates are the same, it writes the object to the database, overwriting what was there. It then looks at the timestamp passed by the client. It searches the database for any objects with modification timestamps greater than the one passed up and sends those objects down to the client. It also sends any conflicted objects in a separate array.

The client then writes the new objects to its own database; using guids to update existing objects. If the client discovers any conflicts, it must present the conflict to the user and ask them to resolve the conflict. It does this by showing the user the two version of the object and asks them to manually merge them. Once the merge is complete the client pushes those updates to the server during the next sync operation.

As always, you can find the latest version of this lib at [https://github.com/Tylerc230/offline_sync_blog_3](https://github.com/Tylerc230/offline_sync_blog_3). Be sure to look at the example folder as this is where the server portion resides. Remember to set the base url of SyncStorageManager to your sever url. Please report any bugs you find to me.

The inspiration for this lib comes from this SO post: [http://stackoverflow.com/questions/5035132/how-to-sync-iphone-core-data-with-web-server-and-then-push-to-other-devices/5052208#5052208](http://stackoverflow.com/questions/5035132/how-to-sync-iphone-core-data-with-web-server-and-then-push-to-other-devices/5052208#5052208)
