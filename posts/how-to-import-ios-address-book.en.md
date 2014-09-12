<!--
.. title: How To Import iOS Address Book
.. slug: how-to-import-ios-address-book
.. date: 2014-09-12 20:59:52 UTC+05:00
.. tags: Programming, iOS, AddressBook, Swift
.. link:
.. description:
.. type: text
-->

**TLDR;** [Go to GitHub Project](https://github.com/ishaq/ContactsImporter)

Importing address book contacts is a straight forward task, however most tutorials discuss using `AddressBookUI` framework to interactively let the user select a contact to import. [This GitHub project](https://github.com/ishaq/ContactsImporter) is a proof of concept (POC) to import contacts via direct access to address book using `AddressBook` framework.

Here's the model class I used to represent a single imported contact:

```swift
//  Contact.swift

import UIKit

class Contact: NSObject {
    var firstName : String
    var lastName : String
    var birthday: NSDate?
    var thumbnailImage: NSData?
    var originalImage: NSData?

    // these two contain emails and phones in <label> = <value> format
    var emailsArray: Array<Dictionary<String, String>>?
    var phonesArray: Array<Dictionary<String, String>>?

    override var description: String { get {
        return "\(firstName) \(lastName) \nBirthday: \(birthday) \nPhones: \(phonesArray) \nEmails: \(emailsArray)\n\n"}
    }

    init(firstName: String, lastName: String, birthday: NSDate?) {
        self.firstName = firstName
        self.lastName = lastName
        self.birthday = birthday
    }
}
```

here are the functions that do the job:

```swift
    // this is the button handler that triggers contacts import
    @IBAction func importContacts(sender: AnyObject) {
        self.requestAccessAndImportContacts()
    }
    func extractABAddressBookRef(abRef: Unmanaged<ABAddressBookRef>!) -> ABAddressBookRef? {
        if let ab = abRef {
            return Unmanaged<NSObject>.fromOpaque(ab.toOpaque()).takeUnretainedValue()
        }
        return nil
    }

    func requestAccessAndImportContacts() {
        if(ABAddressBookGetAuthorizationStatus() == ABAuthorizationStatus.Denied || ABAddressBookGetAuthorizationStatus() == ABAuthorizationStatus.Restricted) {
            let alert = UIAlertView(title: "Address Book Access Denied", message: "Please grant us access to your Address Book in Settings -> Privacy -> Contacts", delegate: nil, cancelButtonTitle: "OK")
            alert.show()
            return
        }
        else if (ABAddressBookGetAuthorizationStatus() == ABAuthorizationStatus.NotDetermined) {
            var errorRef: Unmanaged<CFError>? = nil
            var addressBook: ABAddressBookRef? = extractABAddressBookRef(ABAddressBookCreateWithOptions(nil, &errorRef))
            ABAddressBookRequestAccessWithCompletion(addressBook, { (accessGranted: Bool, error: CFError!) -> Void in
                if(accessGranted) {
                    let contacts = self.importContacts()
                    self.showContacts(contacts)
                }
            })
        }
        else if (ABAddressBookGetAuthorizationStatus() == ABAuthorizationStatus.Authorized) {
            let contacts = self.importContacts()
            self.showContacts(contacts)
        }
    }


    func importContacts() -> Array<Contact> {
        var errorRef: Unmanaged<CFError>? = nil
        var addressBook: ABAddressBookRef? = extractABAddressBookRef(ABAddressBookCreateWithOptions(nil, &errorRef))
        var contactsList: NSArray = ABAddressBookCopyArrayOfAllPeople(addressBook).takeRetainedValue()
        println("\(contactsList.count) records in the array")

        var importedContacts = Array<Contact>()

        for record:ABRecordRef in contactsList {
            var contactPerson: ABRecordRef = record
            var firstName: String = ABRecordCopyValue(contactPerson, kABPersonFirstNameProperty).takeRetainedValue() as NSString
            var lastName: String = ABRecordCopyValue(contactPerson, kABPersonLastNameProperty).takeRetainedValue() as NSString

            println("-------------------------------")
            println("\(firstName) \(lastName)")

            var phonesRef: ABMultiValueRef = ABRecordCopyValue(contactPerson, kABPersonPhoneProperty).takeRetainedValue() as ABMultiValueRef
            var phonesArray  = Array<Dictionary<String,String>>()
            for var i:Int = 0; i < ABMultiValueGetCount(phonesRef); i++ {
                var label: String = ABMultiValueCopyLabelAtIndex(phonesRef, i).takeRetainedValue() as NSString
                var value: String = ABMultiValueCopyValueAtIndex(phonesRef, i).takeRetainedValue() as NSString

                println("Phone: \(label) = \(value)")

                var phone = [label: value]
                phonesArray.append(phone)
            }

            println("All Phones: \(phonesArray)")

            var emailsRef: ABMultiValueRef = ABRecordCopyValue(contactPerson, kABPersonEmailProperty).takeRetainedValue() as ABMultiValueRef
            var emailsArray = Array<Dictionary<String, String>>()
            for var i:Int = 0; i < ABMultiValueGetCount(emailsRef); i++ {
                var label: String = ABMultiValueCopyLabelAtIndex(emailsRef, i).takeRetainedValue() as NSString
                var value: String = ABMultiValueCopyValueAtIndex(emailsRef, i).takeRetainedValue() as NSString

                println("Email: \(label) = \(value)")

                var email = [label: value]
                emailsArray.append(email)
            }

            println("All Emails: \(emailsArray)")

            var birthday: NSDate? = ABRecordCopyValue(contactPerson, kABPersonBirthdayProperty).takeRetainedValue() as? NSDate

            println ("Birthday: \(birthday)")

            var thumbnail: NSData? = nil
            var original: NSData? = nil
            if ABPersonHasImageData(contactPerson) {
                thumbnail = ABPersonCopyImageDataWithFormat(contactPerson, kABPersonImageFormatThumbnail).takeRetainedValue() as NSData
                original = ABPersonCopyImageDataWithFormat(contactPerson, kABPersonImageFormatOriginalSize).takeRetainedValue() as NSData
            }

            let currentContact = Contact(firstName: firstName, lastName: lastName, birthday: birthday)
            currentContact.phonesArray = phonesArray
            currentContact.emailsArray = emailsArray
            currentContact.thumbnailImage = thumbnail
            currentContact.originalImage = original

            importedContacts.append(currentContact)
        }

        return importedContacts
    }

    func showContacts(contacts: Array<Contact>) {
        let alertView = UIAlertView(title: "Success!", message: "\(contacts.count) contacts imported successfully", delegate: nil, cancelButtonTitle: "OK")
        alertView.show()
    }
```

Please note that this code is only a POC and is therefore simple. All it does is:

1. import Persons only (not Groups).
2. For each person, import First Name, Last Name, Birthday (if it exists), all emails and phone numbers and image (if exists).

In any case, it should probably save you a couple of hours of reading on apple documentation.
