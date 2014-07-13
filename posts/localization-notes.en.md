<!--
.. title: Localization Notes
.. slug: localization-notes
.. date: 06/02/14 17:25:31 UTC+05:00
.. tags: Localization, genstrings, Objective C, iOS, Programming
.. link:
.. description:
.. type: text
-->

**UPDATE** Xcode 6 provides a vastly improved workflow for localizations. While *NSLocalizedString* macros guidelines presented here are still relevant. You don't need to rely on 3rd party scripts anymore to keep localization files updated. Watch [session 412 on wwdc](https://developer.apple.com/videos/wwdc/2014/?id=412) to learn localization workflow in Xcode 6.


I sometimes wonder if it is worth it to write blog posts. The things I write here are often compiled from other sources (docs, books, StackOverflow, other blogs, etc). But then my rationale for writing these posts is to write notes for my future self so that I don't spend time looking for the same things again.

Localizing iOS apps is one such problem, there's a lot of information available, but since I didn't know better I approached the problem in worst possible way. So here, dear reader, I am going to write lessons learnt and practices to follow for future projects.

Localising iOS apps is an interesting problem. Apple's documentation is pretty good, their tools are pretty useless. On top of useless tools, a common pitfall for novices is using localization macros improperly, it's tempting to write them like:

```objectivec
NSLocalizedString(@"Please note that syncing data on WWAN may incur data usage charges by Telecomm operators", nil);
```

but as we'll discuss below, it is a nightmare for localization. It should be avoided.

## Getting Started ##

Apple provides four [localization macros](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Miscellaneous/Foundation_Functions/Reference/reference.html):

```objectivec
NSString *NSLocalizedString(NSString *key, NSString *comment)
```
```objectivec
NSString *NSLocalizedStringFromTable(NSString *key,
   NSString *tableName,
   NSString *comment)
```

```objectivec
NSString *NSLocalizedStringFromTableInBundle(NSString *key,
   NSString *tableName,
   NSBundle *bundle,
   NSString *comment)
```

```objectivec
NSString *NSLocalizedStringWithDefaultValue(NSString *key,
   NSString *tableName,
   NSBundle *bundle,
   NSString *value,
   NSString *comment)
```

most of the time, you would use *NSLocalizedString* and *NSLocalizedStringFromTable*. The former generates the default *Localizable.strings* file when using *genstrings* (or other tools that use *genstrings* internally). The later generates *&lt;tableName&gt;.strings*. It is a good idea to use separate tables for strings that belong to different groups e.g. *StandardGUIStrings*, *NetworkErrorStrings*, *InAppPurchasesStrings*, *ApplicationSpecificStrings*, etc.

### Use Localization Macros Properly ###

**TLDR;** Use short, distinct keys like *EditProfile.DiscardChangesAlertTitle*

Like I said above, it's very tempting to start writing localization macros like this:

```objectivec
NSLocalizedString(@"Are you sure?", nil);
```

This is *bad* for many reasons:

* The key is vulnerable to be misused. One can use slightly different keys in different places for the exact same message. This key is tightly tied to the message, So in future if the message was to changed/reworded, this key would be changed i.e.:

```objectivec
// this:
NSLocalizedString(@"Are You Sure?", nil);
// might become this:
NSLocalizedString(@"Are You Sure to Discard Changes?", nil);
```

The keys should be tied to their context/purpose, What purpose does this key serve? Is it *title for discard changes alert*? If so, a better key name would be *DiscardChangesAlertTitle*.

* It provides no auto-completion, a lesser nuisance, but an issue nonetheless.
* It provides no comment for the translators about the context of the string.

To remedy the above problems, I have started to use short keys that convey the purpose they are used for. To save on typing and to take advantage of autocompletion, I have started defining macros for *NSLocalizedString* family macros. Typically my strings macro header file looks like this:

```objectivec
// StandardGUIStrings.h
// `MI` is project prefix
#define kMILSDiscardChangesAlertTitle NSLocalizedStringFromTable(@"DiscardChangesAlertTitle", @"StandardGUIStrings", "Are you sure you want to discard changes? -  generic discard changes alert title")
// ... other macros ...
```

I try to match the header file's name with the string table's i.e. macros for *StandardGUIStrings.string* should be inside *StandardGUIStrings.h*

Also, it is a good idea to name the keys and macros in a way that conveys the module they are used it. When required, I postfix the macro name with `_<modue_name>` and prefix the key with `<module_name>.` For example, I may have strings macros like:

```objectivec
// StandardGUIStrings.h
// `MI` is project prefix
#define kMILSDiscardChangesAlertTitle NSLocalizedStringFromTable(@"DiscardChangesAlertTitle", @"StandardGUIStrings", "Are you sure you want to discard changes? -  generic discard changes alert title")
// ... other macros ...
```

and

```objectivec
// ApplicationSpecificStrings.h
// `MI` is project prefix
#define kMILSDiscardChangesAlertTitle_EditProfile NSLocalizedString(@"EditProfile.DiscardChangesAlertTitle", "Are you sure you want to discard changes to your profile? -  discard changes alert title for edit profile screen")
// ... other macros ...
```

Note that:

1. The above is a hypothetical example but you should get the idea.
2. The reason I like to *postfix* macro names is to have similar strings grouped together in autocompletion popup.
3. I didn't use a key-prefix/macro-postfix for *StandardGUIStrings*, but you can if you want to. These examples are merely guidelines, use your best judgement.

### Use String Tables Effectively ###
**TLDR; ** You *must* use localization tables if you are writing a reusable library. Even if you are not, you *should* use them so that you have *.strings* files that you can carry from project to project.

This is specially true if you are writing a library, use *NSLocalizedStringFromTable* instead of *NSLocalizedString*. It would make your code more portable by avoiding clashes with the other strings in the project.

Even if you are not using a library, you should use string tables to group related strings together. For example, *StandardGUIStrings* may have commonly used UI strings e.g. *OK*, *Cancel*, *Yes*, *No*, etc. *IAPStrings* may have strings for InApp Purchases. *NetworkMessages* (see note below) may have network messages e.g. *Login Failed*, *Username Already In Use*, etc. And an *ApplicationSpecificStrings* may have the strings that are only used within that particular app. This allows you to carry generic strings/headers from one project to another and reduces the effort/cost of translation because only a subset of strings may need to be translated.

**NOTE**: Network error messages means messages specific to your applications. The built-in error messages e.g. *Network Timeout* are localized by apple. `localizedDescription` method of `NSError` would return a localized string suitable for displaying to the user.

As mentioned in the [A Localization Workflow that works well](http://innovaptor.com/blog/2013/02/07/a-localization-workflow-that-works-well-in-practice.html), it is tempting to define custom macros like *NSLocalisedStringStandardGUI* to use *StandardGUIStrings* table instead of typing *NSLocalizedStringFromTable*. It is not worth it because *genstrings* won't find it.

### Enable Base Localization and Generate Initial set of .strings ###
**TLDR; ** Enable *Base* localization on your project and add any localization languages. It is also a good idea to use Interface Builder to generate initial set of *.strings* for your storyboards/xibs for each localized language because it gives you a base structure.

Open your project settings in Xcode and check *Use Base Internationalization* on the *Info* tab. You should also add all localization languages that you plan to support. Note that if you just started your project and there are no Interface Builder files in it yet, you will not be able to enable *Base* localization. Add a storyboard first and then try enabling it.

![Check "Use Base Internationalization"](http://i.imgur.com/X3V6vLK.png "Use Base Internationalization should be checked")

*Check "Use Base Internationalization"*



![Use +/- to add remove supported Localizations](http://i.imgur.com/qZvwy9M.png "Use +/- to add remove supported Localizations")

*Use +/- to add remove supported Localizations*



To localize a storyboard or xib, select it in the Project Navigator (`Cmd+1`). In the File Inspector (`Option+Cmd+1`), there is *Localize* button in the *Localization* section, . Click it to generate *Base* localization. Afterwards it would give you the option to either generate *Localizable Strings* OR *Interface Builder Cocoa Touch Storyboard* for each language.  



![Localize button is shown if interface file is not already localized.](http://i.imgur.com/W17C2ba.png "Localize button is shown if interface file is not already localized.")

*Localize button is shown if interface file is not already localized*



*Localizable Strings* is a new localization option for interface files. it generates a *.strings* file much like the one generated by `genstrings` tool. Auto Layout will take care of different string lengths and text direction.



![Choose either Localizable Strings or Inteface Builder file](http://i.imgur.com/oDFsAbs.png "Choose either Localizable Strings or Inteface Builder file")

*You can choose either Localizable Strings or Inteface Builder file*


**UPDATE:** As mentioned at the top of this post, if you are using Xcode 6, you don't need to rely on 3rd party tools. You should skip rest of the post.

### The update_localization Script ###

**TLDR; **Add *update_localization.py* as *Run Script* phase to your target's build phases. Your *.strings* files will be updated with new items everytime you build.

**GitHub Link:** [update_localization.py](https://github.com/iv-mexx/update_localization)

The most famous tool for localiation is *genstrings*. Unfortunately it's not as useful. It scans your code files looking for localization macros and generates *.strings* files based on what it finds. Everytime it runs, it overwrites the generated *.strings* files, so you have to manually re-insert any translations that were already done. Also it doens't extract strings from interface (storyboard/xib) files.

There's another Apple tool called *AppleGlot*. Registered Apple developers can download it from [Developer Download Center](https://developer.apple.com/downloads/). I suggest to not waste time on its current version (4.0 at the time of this writing), it is very difficult to integrate into an automated workflow and requires too much house keeping.

[update_localization.py](https://github.com/iv-mexx/update_localization) is a small python script that can automatically extract strings from the code and interface files. It keeps track of already translated keys and will not overwrite them. This is a very good tool, the following text elaborates on integrating it into your build process.

First, you need to get *update_localization.py*, So:

1. [Download](https://github.com/iv-mexx/update_localization) the script (you only need *update_localization.py* to extract/update *.strings* files)
2. If it is not already an executable, mark it as such with: `chmod +x update_localization.py`

You can test the script in *Terminal* to make sure it works the way you want it to. At the time of this writing, It takes the following arguments (all optional):

```
-i, --input  : input path, this is the path that will be scanned to update *.strings files
-o, --output : output path, this is the path where *.strings files would be updated
--ignore     : these path patterns would be ignored, for example you may want to ignore `Pods` or other 3rd party library directories
--interface  : scan interface files too, in absence of this flag, only code files will be scanned
--extension  : file extensions that should be scanned
--verbose    : makes it verbose
-h, -help    : prints help message
```

You should check the available options yourself. Either run `./update_localization --help` OR look at the source code.

### Project Structure to Support update_localization Script ###
**TLDR; ** Place all your localizable resources (storyboards, xibs, .strings) in single *&lt;lang&gt;.lproj* folders.

*update_localization* script generates all its output to a single folder. So all of your localizable resources (storyboards, xibs, .strings) should be in one *&lt;lang&gt;.lproj* folder. For example, a directory structure like this won't work so well

```
// This directory structure for localization folders won't work well with update_localization

<project root>
 |-- <project name> // .lproj folders here have storyboards and .strings
 |     |-- Base.lproj
 |     |-- en.lroj
 |     |-- nl.lproj
 |     +-- Source
 |     |    |-- Views  // .lproj folders under `Views` have xibs of custom views
 |     |    |    |-- Base.lproj
 |     |    |    |-- en.lproj
 |     |    |    +-- nl.lproj
 |     |    |-- Controllers // .lproj folders under `Controllers` have xibs for controllers
 |     |    |    |-- Base.lproj
 |     |    |    |-- en.lproj
 |     |    |    |-- nl.lproj
 |     |    |-- Models
 |     |    +-- ...
 |     +-- ...
 |-- <project name>Tests
 |-- Pods // Cocoa Pods directory
 |-- <project name>.xcodeproj
 |-- <project name>.xcworkspace
 +-- ...
```

You should not have multiple folders for same localization e.g. in the above directory tree, *en.lproj* appears three times 1) at &lt;project name&gt;, 2) under *Views* and 3) and *Controllers*. Put all localizable resources in single *.lproj* folders like this:

```
// This directory structure for localization folders works better

<project root>;
 |-- <project name> // all localizable files (.storyboard, .xib or .strings) are here inside <lang>.lproj folders
 |     |-- Base.lproj
 |     |-- en.lroj
 |     |-- nl.lproj
 |     |-- Source
 |     |    |-- Views // xibs of views are in the .lproj folders above
 |     |    |-- Controllers // xibs of controllers are in the .lproj folders above
 |     |    |-- Models
 |     |    |-- ...
 |     +-- ...
 |-- <project name>Tests
 |-- Pods // Cocoa Pods directory
 |-- <project name>.xcodeproj
 |-- <project name>.xcworkspace
 +-- ...
```

With a directory structure like above, all *.strings* files for, say english, will be copied to *en.lproj*

I like to put all the localization folders under their own parent called *LocalizableResources*. Also, I have a *tools* folder at *&lt;project root&gt;* which contains a folder called *update_localization*. Inside it, I put the *update_localization.py* script. My project directory structure looks like this:

```
// This is what my directory structure looks like

<project root>
 |-- <project name>
 |     |-- LocalizableResources // all localizable files (.storyboard, .xib or .strings) are here inside `<lang>.lproj` folders
 |     |    |-- Base.lproj
 |     |    |-- en.lroj
 |     |    |-- (any other <lang>.lproj folders)
 |     |    +-- nl.lproj
 |     |-- Source
 |     |    |-- Views // xibs of views are in the .lproj folders above
 |     |    |-- Controllers // xibs of controllers are in the .lproj folders above
 |     |    |-- Models
 |     |    |-- ...
 |     +-- ...
 |-- <project name>Tests
 |-- Pods // Cocoa Pods directory
 |-- <project name>.xcodeproj
 |-- <project name>.xcworkspace
 |-- tools
 |    +-- update_localization
 |         +-- update_localizatin.py   // the update_localization.py script
 +-- ...
```

The localization commands in the next section assume this directory structure, If yours is different, please modify the paths accordingly.

### Add update_localization Script into your build phases ###
Open your target's build settings in Xcode. On the *Build Phases* tab, add *New Run Script Build Phase*.

![New Run Script Build Phase](http://i.imgur.com/nvzgjKS.png "New Run Script Build Phase")

*New Run Script Build Phase*


![Paste the commands in the textarea under Shell field](http://i.imgur.com/xHqWwOk.png "Paste the commands in the textarea under Shell field")

*Paste the commands in the textarea under Shell field*


In the textarea for script, paste the following commands:

```bash
./tools/update_localization/update_localization.py --input pilgrimapp/ --output pilgrimapp/LocalizedResources/en.lproj/ --interface --extension h --extension c --extension m --extension mm
./tools/update_localization/update_localization.py --input pilgrimapp/ --output pilgrimapp/LocalizedResources/nl.lproj/ --interface --extension h --extension c --extension m --extension mm
```



![The above commands pasted in the textarea](http://i.imgur.com/tdrfQvW.png "The above commands pasted in the textarea")

*Run Script phase with update_localization commands pasted. (NOTE: You can double click the build phase title "Run Script" and rename it to something more helpful. In the above figure, I have renamed mine to "update_localization -- Update Strings Files")*

Note that

* *pilgrimapp* is my project's name, replace it with yours.
* You'll have to change the paths to match yours if your directory structure is not the same as described previous section.
* I am doing two localizations. English i.e. *en.lproj* and Dutch i.e. *nl.lproj*. The *update_localization.py* script has to be run for both. In future if we decide to add localization for another language, say Arabic i.e. *ar.lproj*, we'll have to add command for that too.
* If you are not using a macro header file, you can leave the `--extension` parameters. *update_localization* will search `.m`, `.mm` and `.c` files by default.

Now everytime you build your project, any new items will be automatically added to *.strings* files while keeping the existing translations intact.

That is it, localizations are not as hard if you follow a good workflow and most of the time they are worth it. Localized apps get more attention since a user is more inclined to download an app in his native language. Even if you are not planning to localize your app, It's a good idea to keep user facing strings separate from the rest of the code.

### TODO ###
* There should be a way for *update_localization* to automatically detect the languages the project is supposed to be localized in, and it should run for each of them. Explicity adding a command for every language is a minor nuisance though, and that too, only when we decide to add a new language to the project.

### Related Articles ###
* [A Localization Workflow that works well](http://innovaptor.com/blog/2013/02/07/a-localization-workflow-that-works-well-in-practice.html): This is the *update_localization* script I use. The article itself is a good read. This is where I got the idea to use multiple string tables.
- [Best Practice using NSLocalizedString](https://stackoverflow.com/questions/9895621/best-practice-using-nslocalizedstring/10513294#10513294): a Stack Overflow thread. [Answer by Pascal](http://stackoverflow.com/a/10513294/83950) has a handy macro. if a key has not been translated, it uses comment as value.
- [Automatic Strings Extraction From Storyboards for Localization](http://oleb.net/blog/2013/02/automating-strings-extraction-from-storyboards-for-localization/): another good article on automatically extracting strings from Storyboards (or nibs). The script that comes with it works quite well with interface files, as in, it doesn't need a one *&lt;lang&gt;.lproj* like *update_localization* script does. It detects multiple *lproj* folders for the same language and works well with them. I did not use it because I wanted a single tool to handle both *.strings* and interface files. At the time of this writing, this article suggests checking Apple's *AppleGlot* out, but like I said above, I would suggest to not waste time on *AppleGlot* (at least not on the current version 4.0).

### UPDATE ###
[Lin-Xcode5](https://github.com/questbeat/Lin-Xcode5) looks like a very promising tool (ironic though that the demo screenshot of a localization tool shows *NSLocalizedString* without a comment). I cannot test it because the *update_localization* script outputs files in UTF16 format and *Lin-Xcode5* supports UTF8 files only. Note that according to [Apple Docs](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/LoadingResources/Strings/Strings.html), UTF16 is the preferred file format for localization files. I guess I'll modify *Lin-Xcode5* and see. Thanks to **Daniel Hotop** for pointing me to it.

>Note: It is recommended that you save strings files using the UTF-16 encoding, which is the default encoding for standard strings files. It is possible to create strings files using other property-list formats, including binary property-list formats and XML formats that use the UTF-8 encoding, but doing so is not recommended. For more information about the standard strings file format, see “Creating Strings Resource Files.” For more information about Unicode and its text encodings, go to http://www.unicode.org/ or http://en.wikipedia.org/wiki/Unicode.

If you would like to change *update_localization* to produce UTF8 files to use with Lin, change the default encoding from `utf16` to `utf8` for `write_file` and `strings_to_file` functions.


### UPDATE 2014-06-10 ###
After configuring *update_localization*, your builds may start failing because when Xcode tries to run this script. If you look at the build log, it would say something alongs the lines of *ibtool* not being able to open a storyboard (or xib) file. This is because ibtool demon sometimes doesn't get terminated after interface files have been processed (bug?). If you see this, run:

```
ps aux | grep ibtoold
```

and run `kill -9` on any hung *ibtoold* process you see.
