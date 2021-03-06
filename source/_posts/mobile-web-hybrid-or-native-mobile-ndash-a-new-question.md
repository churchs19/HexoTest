title: Mobile Web, Hybrid, or Native Mobile – A New Question
date: 2012/4/23 14:17:57
alias: blog/471/
tags:
- Development
- Software
- Mobile
---
Continuing on my recent thread about [decision making for choosing a mobile web, hybrid, or native mobile application architecture](/Blog/Entry/470), I’ve added another question to consider for the decision tree:

* Does your application need to take advantage of push notifications?

After a discussion with a client last week, I ended up adding this question to my list. The client had already implemented a hybrid mobile application based on ASP.NET for Android and iPhone and wanted to have all of the registration take place through a web form, but I quickly noticed that the registration processes are very different between Android, iOS, and Windows Phone. Not only are the processes different, but the central application server needs to know what type of device it is to successfully send a push notification. Once the device is registered for push notifications, the basic mechanics of sending and receiving notifications are similar on each of the devices.

That being said, my first dive into the world of push notifications has suggested that the best approach for building push notifications would be to build all of the registration and message handling functionality as native mobile code, even within the context of a hybrid app.  Push notifications are definitely a disqualifier for a pure mobile web solution, but don’t necessarily push all the way into a fully native mobile solution.

More reading:

[Windows Phone 7 Push Notifications](http://msdn.microsoft.com/en-us/library/ff402558/)

[Android Cloud to Device Messaging](https://developers.google.com/android/c2dm/)

[iOS Push Notifications](https://developer.apple.com/appstore/push-notifications/index.html)