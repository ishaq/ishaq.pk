<!--
.. title: Experience with Realm Cocoa
.. slug: experience-with-realm-cocoa
.. date: 2014-12-28 17:11:27 UTC+05:00
.. tags: iOS, Programming
.. link:
.. description:
.. type: text
-->

Realm Cocoa has created some buzz over the last few months. Some of the reasons one might use it are:

- It gives you a ready to use model layer. You don't have to write middle layer code to map objects to db records.
- It allows querying using Predicates.
- It supports Swift i.e. if you are using Swift
- It's fast.

If you want to read more on why you might want to use Realm, you should check their [excellant docs](http://realm.io/docs/cocoa/)

However, what's not mentioned in their documentation is that:

- Realm has [no support for NULL values](https://github.com/realm/realm-cocoa/issues/628)
- No support for integrity constraints
- There is no concept of relationships through an intermediate (may be weak) object
- Model layer is barebones.

For me, no support for NULL values and integrity constraints is a deal breaker. It is fun working with it but I would stay with SQLite in future.

Note that Realm may be a pretty good database and It may be just me struggling to get rid of RDBMS habits. Use your best judgement.
