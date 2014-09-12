<!--
.. title: Number of Days Till Next Birthday
.. slug: number-of-days-till-next-birthday
.. date: 2014-09-12 20:02:09 UTC+05:00
.. tags: Swift, Programming, Cocoa
.. link:
.. description:
.. type: text
-->

Just wrote this function as part of a routine that sends reminders for upcoming birthdays of your friends, it may come in handy in future:

```swift
class func daysToNextBirthday(birthdate: NSDate?) -> Int {
        if (birthdate != nil) {
        }
        else {
            return Int.max
        }

        let calender = NSCalendar.currentCalendar()
        let now = NSDate()
        let nowComponents = calender.components(NSCalendarUnit.MonthCalendarUnit | NSCalendarUnit.DayCalendarUnit | NSCalendarUnit.YearCalendarUnit, fromDate: now)
        let birthdayComponents = calender.components(NSCalendarUnit.MonthCalendarUnit | NSCalendarUnit.DayCalendarUnit | NSCalendarUnit.YearCalendarUnit, fromDate: birthdate!)

        // birthday is today or later this year
        if(birthdayComponents.month > nowComponents.month || (birthdayComponents.month == nowComponents.month && birthdayComponents.day >= nowComponents.day)) {
            birthdayComponents.year = nowComponents.year
        }
        else { // birthday passed, next birthday would be next year
            birthdayComponents.year = nowComponents.year + 1
        }

        let daysRemaining = calender.components(NSCalendarUnit.DayCalendarUnit, fromDateComponents: nowComponents, toDateComponents: birthdayComponents, options: nil)
        let nextBirthday = calender.dateFromComponents(birthdayComponents)
        println("birthdate: \(birthdate!) - nextBirthday: \(nextBirthday) - daysRemaining: \(daysRemaining.day)")
        return  daysRemaining.day
    }
```

Also available as [gist](https://gist.github.com/e5bbeb0595123d3ca0e7.git)
