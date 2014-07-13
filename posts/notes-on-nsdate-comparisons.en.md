<!--
.. title: Notes on NSDate Comparisons
.. slug: notes-on-nsdate-comparisons
.. date: 2014/05/07 18:30:05
.. tags: Objective C, iOS, Programming
.. link:
.. description:
.. type: text
-->

Recently I had to write some code to sync data between an iOS app and a server. From the outset it didn't look that hard. At high level, it meant:

1. Pull a record from the server, lets call it `remoteRecord`
2. Search local database using `remoteRecord`'s primary key and get the matching `localRecord`
3. compare `modified` timestamps
    1. if `remoteRecord.modified > localRecord.modified`, update `localRecord`
    2. else if `remoteRecord.modified < localRecord.modified`, update `remoteRecord`
    3. else both records are identical. skip and go to next pair of records


Translating it to code was another story; but that's probably another post. This post is about date time nuisances. Although the title mentions `NSDate`, these issues apply to any datetime data type that includes fractional seconds in it.

### Comparing using Binary Comparison Operators i.e. ==, !=, >, <, >=, <= ###

This one is straight forward, but occasionally I end up writing

```objectivec
if(remoteRecord.modified > localRecord.modified) {
    // update local record
}
else if(remoteRecord.modified < localRecord.modified) {
    // update remote record
}
else { // 2nd else
    // skip, go to next pair of records
}
```

which is wrong and should actually be

```objectivec
if([remoteRecord.modified compare:localRecord.modified] ==  NSOrderedDescending) {
    // update local record
}
else if([remoteRecord.modified compare:localRecord.modified] == NSOrderedAscending) {
    // update remote record
}
else { // 2nd else
    // skip, go to next pair of records
}
```

This is one of those mistakes where you get zero help from compiler and analyzer (no warnings or hints, and yes I know compiler thinks I am doing pointer comparison). It is only after a few confused test runs that you notice the mistake. And it is too easy to fall into this trap if you are also programming in a language that supports binary comparison operators for dates. Python and Ruby are two such languages. Both are used by many iOS developers to write servers. So it would have been great if dev tools could drop a hint when they see this.

### Comparing Dates without considering fractional seconds ###

The above comparison still didn't quite work. See I don't want to process records that have same timestamp to save on network traffic. However, I noticed that the code almost never entered the second `else` clause, In fact sometimes it would keep pulling/pushing same data over and over, wasting bandwidth and battery. So I took a closer look:

```
remoteRecord.modified: May 7, 2014, 6:35:10 PM
localRecord.modified: May 7, 2014, 6:35:10 PM
```

Both the records seem to have same timstamp but the code wasn't hitting the second `else`. Upon closer inspection:

```
remoteRecord.modified:
    May 7, 2014, 6:35:10 PM
    2014-05-07T18:35:10.393+0500 (ECMA 262 format)
    421162510.393000 (timeIntervalSinceReferenceDate)
localRecord.modified:  
    May 7, 2014, 6:35:10 PM
    2014-05-07T18:35:10.393+0500 (ECMA 262 format)
    421162510.393426 (timeIntervalSinceReferenceDate)
```
See the small (but very real) difference in `timeIntervalSinceReferenceDate` representation (third representation)? Here's what happened:

1. `localRecord` was created on the client, its `modified` timestamp set to `421162510.393426`.
2. On first sync, I pushed this record to the server, however when `421162510.393426` was converted to ECMA 262 string, it dropped the microseconds and sent `421162510.393`. Now server has a `modified` timestamp that is slightly behind the client's.
3. On next sync operation, this same record would be pushed again, microseconds would be dropped again. All subsequent sync operations would keep pushing it again and again because the client thinks it has slightly newer/updated record.

This is one scenario, there may be others. Also you don't necessarily have to be sending dates over the network to face this issue. Say you wanted to display records to the user, but you wanted to sort them so that records with the same `modified` date would be alphabetically sorted by `title`. If you don't ensure you are dropping the same fractional components (millis, micros, seconds, minutes) in all the places where it matters, results would be unexpected. One such example is discussed here: [NSDate and Fractional Seconds](http://woolybeastsoftware.com/woolyblog/2012/07/18/nsdates-and-fractional-seconds/).

#### The Fix ####

In my case, I only needed dates to be accurate to seconds. So I started to drop all fractional seconds. Here's the small category I wrote to ease it:

**NSDate+dateByRemovingFractionalSeconds.h**
```objectivec
#import <Foundation/Foundation.h>

@interface NSDate (dateByRemovingFractionalSeconds)
+ (NSDate *)dateWithoutFractionalSeconds;
- (NSDate *)dateByRemovingFractionalSeconds;
@end
```

**NSDate+dateByRemovingFractionalSeconds.m**
```objectivec
#import "NSDate+dateByRemovingFractionalSeconds.h"

@implementation NSDate (dateByRemovingFractionalSeconds)
+ (NSDate *)dateWithoutFractionalSeconds {
    NSDate *now = [NSDate date];
    NSTimeInterval seconds = trunc([now timeIntervalSinceReferenceDate]);
    NSDate *date = [[self class] dateWithTimeIntervalSinceReferenceDate:seconds];
    return date;
}
- (NSDate *)dateByRemovingFractionalSeconds {
    NSTimeInterval seconds = trunc([self timeIntervalSinceReferenceDate]);
    NSDate *date = [[self class] dateWithTimeIntervalSinceReferenceDate:seconds];
    return date;
}
@end
```

Problem solved. The above category is [available as gist](https://gist.github.com/ishaq/1eef9f62757611b47ec4) too.


**NOTE:** I use [ECMA 262 date time string specification](http://ecma-international.org/ecma-262/5.1/#sec-15.9.1.15)  because the server (built on [Django REST Framework](http://www.django-rest-framework.org)) uses it by default. ECMA 262 is a subset of ISO 8601 which uses millisecond precision, and includes the 'Z' suffix for the UTC timezone, for example: `2013-01-29T12:34:56.123Z`.

**PS:** On a side note, [DateTools](https://github.com/MatthewYork/DateTools) is a good library for Date/Time handling.
