<!--
.. title: List of all Country Codes Sorted by Their Localized Names
.. slug: list-of-all-country-codes-sorted-by-their-localized-names
.. date: 05/29/14 09:36:35 UTC+05:00
.. tags: Objective C, iOS, Programming
.. link:
.. description:
.. type: text
-->

If you need to display a list of countries in an iOS app, for example on a registration form. The following code creates a sorted array of country codes:

```objectivec
NSLocale *locale = [NSLocale currentLocale];
NSArray *countryArray = [NSLocale ISOCountryCodes];

NSMutableDictionary *countriesDictionary = [[NSMutableDictionary alloc] initWithCapacity:countryArray.count];
for (NSString *countryCode in countryArray) {
    NSString *localizedCountryName = [locale displayNameForKey:NSLocaleCountryCode value:countryCode];
    countriesDictionary[countryCode] = localizedCountryName;
}

NSArray *sortedCodes = [countriesDictionary keysSortedByValueUsingComparator:^(id obj1, id obj2) {
     NSString *country1 = obj1;
     NSString *country2 = obj2;
     return [country1 localizedCompare:country2];
 }];
```

I am using country codes (instead of country names) here because it makes sense to story standardized codes e.g. PK in the database. And besides, if you have country code, you can always get its localized name with:

```objectivec
NSString *localizedCountryName = [locale displayNameForKey:NSLocaleCountryCode value:countryCode];
```

The above code is also [available as gist](https://gist.github.com/ishaq/45ed5e0201be380b359c).
