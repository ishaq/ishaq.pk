<!--
.. title: Released on the App Store: PILNAR Pilgrim Diary
.. slug: released-on-the-app-store-pilnar-pilgrim-diary
.. date: 2014-07-17 11:17:45 UTC+05:00
.. tags: App Store, iOS, Releases
.. link:
.. description:
.. type: text
-->

**TLDR;** [App Store Link](https://itunes.apple.com/us/app/pilnar-pilgrim-diary/id897587238?mt=8)

[PILNAR Pilgrim Diary](https://itunes.apple.com/us/app/pilnar-pilgrim-diary/id897587238?mt=8) has been released to the App Store. It's an application for Christian Pilgrims to replace paper diaries during their caminos. The application has been developed for a researcher from [Tilburg University](http://www.tilburguniversity.edu/) and is part of their PILNAR project.

Server for the application is written in Django. It uses Django REST Framework to provide the API layer. Client is native iOS application. From a developer standpoint, it has the following standout features:

- Offline Sync
- Multi-lingual (English and Dutch)
- Custom reports for Admins (on the server)


I am proud of offline sync. It feels good to be able to use the app with or without an internet connection and notice the changes being synced back to the server whenever you get a connection. User can also force sync content if he wants the changes to immediately be reflected on the server. I have been meaning to write notes about offline sync for some time. Basically it boils down to:

1. Pull the modified/new lookup data
2. Push the modified/new content to the server
3. Pull the modified/new content from the server.

Notice that any information is pulled/pushed only if it needs to be, both client/server keep track of timestamps, so we are wasting very little bandwidth.

One downside of offline sync is that if the user has not had internet connectivity for a long time, User may be looking at stale data e.g. user may be reading a story which has been heavily modified by its author.

I have already written about [localization](localization-notes.html). That post was written at an interesting time. It dealt with Xcode 5, Apple released Xcode 6 days later which has much improved support for Localization than its predecessor. The general good practices still apply.

Custom reports weren't so difficult, all I had to do was to modify admin templates to put links to the new views (report views). There's the nice [django-adminplus](https://github.com/jsocol/django-adminplus) app which can make it easier but I couldn't use it because I was on django 1.7 and it's not compatible with it.

It's a useful application , If you are going on a camino, do give it a try. Buen Camino!
