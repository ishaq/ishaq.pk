<!--
.. title: Realm Cocoa: Inserting invalid object for RLMPropertyTypeAny property
.. slug: realm-cocoa-inserting-invalid-object-for-rlmpropertytypeany-property
.. date: 2015-01-21 13:51:40 UTC+05:00
.. tags: iOS, Programming, Realm
.. link:
.. description:
.. type: text
-->

I started getting this exception when one of my models had an array property defined. For example:

```swift
class Post : RLMObject {
  dynamic var content: String = ""
  dynamic var authors: Array<String> = "" // This property is an Array i.e. not supported by Realm
}

let post = Post()
post.content = "Hello World"
post.authors = ["ishfaq", "shoaib"]

let realm = RLMRealm.defaultRealm()
realm.beginWriteTransaction()
realm.addObject(post) // This line would throw the exception
realm.commitWriteTransaction()
```

 Realm docs on [property types](http://realm.io/docs/cocoa/0.89.2/#property-types) don't include NSArray. This is an easy mistake to make given we can have an array (`RLMArray`) of `RLMObject` instances. And it can be hard to debug since the exception throw does not include name of the property. I have created [Ticket 1369](https://github.com/realm/realm-cocoa/issues/1369) for it.
