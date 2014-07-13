<!--
.. title: Project Structure Notes
.. slug: project-structure-notes
.. date: 06/10/14 10:17:15 UTC+05:00
.. tags: Programming, iOS, Xcode
.. link:
.. description:
.. type: text
-->


A good project structure makes maintenance easy. And if we follow the same project structure across projects, a new programmer; who is already familiar with the structure, will quickly become productive. This post is summary of the project structure I have used for sometime. Like any other technical article, the guidelines presented should be improved upon as required and/or new tools become available.

## TLDR; ##
1.     Use [CocoaPods](http://cocoapods.org/), the de-facto package manager for Cocoa projects, for dependency management.

2.     Use [xcode-git-version.sh](https://gist.github.com/ishaq/4003642#file-xcode-git-version-sh) in a *Run Script* build phase to automatically set build version using git repository information.

3.     Put [ArchiveHousekeeper](https://gist.github.com/ishaq/167ef4940f8615f5ac19) script in a target (say *ArchiveHousekeeper*) and update target scheme's Build section to add *ArchiveHousekeeper* as dependency for Archive operation. It would stop the build when archiving, hence making sure that releases (archives) are created from a clean git tree.

4.     Use [update_localization](localization-notes.html) script to automatically update strings files. Follow the practices listed in that post.


## Step By Step ##

Create/Clone a git repository. Create an appropriate *.gitignore* file in it. A typical .gitignore file for an iOS project might look like:

```
# OS X
.DS_Store

# Xcode
build/
*.pbxuser
!default.pbxuser
*.mode1v3
!default.mode1v3
*.mode2v3
!default.mode2v3
*.perspectivev3
!default.perspectivev3
xcuserdata
*.xccheckout
profile
*.moved-aside
DerivedData
*.hmap
*.xccheckout

# CocoaPods
Pods
```

Please note that this is only an example, [gitignore](https://github.com/github/gitignore) repository on GitHub has a comprehensive collection of .gitignore templates, mix them as appropriate for your project.

From here on, I would assume that the project exists in a git repository.

### Install CocoaPods ###

1.    Create a new project in Xcode as usual. I'll call it  *TestProject* for this post. Open terminal and `cd` to project's root directory.

2.     If [CocoaPods](http://cocoapods.org/) is not already installed, run `sudo gem install cocoapods` to install it. Be sure to read at least the [Getting Started](http://guides.cocoapods.org/using/getting-started.html) guide on [CocoaPods Guides](http://guides.cocoapods.org/) if this is your first time using CocoaPods.

3.     run `pod init` to create *Podfile*. Note that this file can just as well be created by hand. `pod init` only provides a template to serve as a starting point. Edit the Podfile to add pods. At the very least you should add *TestFlightSDK*. Your *Podfile* should roughly look like this:

        platform :ios, "7.0"

        target "TestProject" do
        pod "TestFlightSDK", "~> 3.0"
        end

        target "TestProjectTests" do

        end

    Note: *3.0* is the latest version of *TestFlightSDK* pod at the time of this writing. You should use latest version available.

4.    Run `pod install` to install the pods. It does two things, a) creates a workspace if needed and b) installs any needed pods. Note that you should run it even if you didn't add any pods to the Podfile because we want the workspace to be created.

5.    `pod install` would have told you to use workspace (*.xcworkspace*) file instead of project file (*.xcodeproj*) from now on. CocoaPods installs pods in a subproject in the workspace. If you use the project file instead of workspace, it would not be able to find the pods and fail to compile.

6.    If you added *TestFlightSDK* pod (you absolutely should have), go to the [TestFlight](http://testflightapp.com/) website and follow the instructions to integrate it.

7.    commit the changes. If the repository doesn't have any tags yet, create a tag now. I'll call it `0.0.0` because I use [semantic versioning](http://semver.org/).


### Add Git Version Script ###

1.    Open *Build Phases* of the target, *TestProject* in this case, and add *New Run Script Build Phase*.

    ![New Run Script Build Phase](http://i.imgur.com/gHy6a4X.png "New Run Script Build Phase")

2.    Paste contents of [xcode-git-version.sh](https://gist.github.com/ishaq/4003642#file-xcode-git-version-sh) into the textarea for the script. Double click the title of the phase (it would say *Run Script*) and change it to *Set Version*. Build phase titles appear in the build log and when you have more than one *Run Script* build phase, a good title helps identify the phase in the build log.

    ![Set Version Build Phase](http://i.imgur.com/RP6tyXC.png "Set Version Build Phase")

3.  Build the project to make sure the script is working. Your build log should have an output similar to the screenshot below.

    ![Set Version output](http://i.imgur.com/KwVGIAF.png "Set Version output")

    Note that *Set Version* script won't work if your repository doesn't have any tags. I usually create a tag called *0.0.0* at the very first commit.

    In addition to the standard `CFBundleVersion` and `CFBundleShortVersionString`, this script adds a new key called `FullVersion` which includes short SHA hash of the commit the build is created from. This string may be displayed on About page of the application. The testers may specify it in the bug reports. However it's not needed if you use [semantic versioning](http://semver.org/) and ArchiveHousekeeper script. Semantic versioning would require you to set meaningful version labels and ArchiveHousekeeper will not let you create releases (archives) from a dirty repository. More on ArchiveHousekeeper below.

### Add ArchiveHousekeeper Script ###

1.    Create a new directory called *tools* inside the project root. Create a directory called *ArchiveHousekeeper* inside it. If you are in the terminal, run the following commands inside project root.

        mkdir tools
        cd tools
        mkdir ArchiveHousekeeper


2.    Download [ArchiveHousekeeper.sh](https://gist.github.com/ishaq/167ef4940f8615f5ac19#file-archivehousekeeper-sh) and place it inside *ArchiveHousekeeper* directory. If the file is not executable, mark it as such with:

        chmod +x ArchiveHousekeeper.sh


3.    Create a new *External Build System* target to your project. Set *Product Name* to *ArchiveHousekeeper* and *Build Tool* to *./tools/ArchiveHousekeper/ArchiveHousekeeper.sh*. Click *Finish*.

    ![External Build System Step 1](http://i.imgur.com/Z00lubm.png "External Build System Step 1")

    ![External Build System Step 2](http://i.imgur.com/7wn49Bs.png "External Build System Step 2")

4.  Open *TestProject* target's scheme (Tap on the target name in the top bar and choose *Edit Scheme*). Open *Build* section and click the *+* button in bottom left corner of *Targets* pane to add a new target. Add *ArchiveHousekeeper*.

    ![Build section of target's scheme](http://i.imgur.com/SyqVIPi.png "Build section of target's scheme")

5.  Uncheck ArchiveHousekeeper for all build operations except *Archive*. Click *OK* to dismiss the popup.

    ![ArchiveHousekeeper is a dependency for Archive operation](http://i.imgur.com/5ugTe2H.png "ArchiveHousekeeper is a dependency for Archive operation")

6.  Try archiving the project, it should fail since your project has uncommitted changes. (Note: If *Archive* menu item in the *Product* menu is disabled/gray, you probably have a simulator selected as target platform, selecting *iOS Device* would enable it. )

    ![Archive operation on a dirty tree would fail](http://i.imgur.com/FVUPkKv.png "Archive operation on a dirty tree would fail")

7.  Commit all pending changes to make working directory clean and try archiving again. This time it should succeed.

    ![Archive operation on a clean tree would succeed](http://i.imgur.com/pOnYPyQ.png "Archive operation on a clean tree would succeed")


### Add update_localization Script ###

Localization requires planning, See my [previous post](localization-notes.html) on this. I am intentionally not presenting a summary here. Go read that post and come back.


### Project Directory Structure ###

Here's the rough directory structure I follow:

```
TestProject
 |-- TestProject
 |     |-- LocalizableResources // all localizable files (*.storyboard, *.xib or *.strings) are here inside `<lang>.lproj` folders
 |     |    |-- Base.lproj
 |     |    |-- en.lroj
 |     |    |-- (any other <lang>.lproj folders)
 |     |    +-- nl.lproj
 |     |-- Source
 |     |    |-- Controllers // Controllers and Views *.h/*.m go into their respective dirs here, their *.xib or *.storyboard files
 |     |    |    |          // would go into the appropriate .lproj folders in the LocalizableResources folder above
 |     |    |    +-- Views
 |     |    |-- Models // data models (may have a  CoreData subdirectory if project uses CoreData)
 |     |    |-- Utils // contains utility code that doesn't fit anywhere else
 |     |    +-- ...
 |     +-- ...
 |-- TestProjectTests
 |-- Pods // Cocoa Pods directory
 |-- TestProject.xcodeproj
 |-- TestProject.xcworkspace // workspace file, always open this instead of the project file
 |-- tools
 |    |-- update_localization
 |    |    +-- update_localizatin.py   // update_localization.py script
 |    +-- ArchiveHousekeeper
 |         +-- ArchiveHouseKeeper.sh // ArchiveHousekeeper.sh script
 +-- ...
```

### Other Tips ###

*     Use `DLog` and `ALog` instead of `NSLog`. Additionally, `LogError` is a shortcut to conditionally log `NSError` objects. Paste the following code inside your precompiled header (*TestProject-Prefix.pch* in this case).

```objectivec
#ifdef DEBUG
#define DLog(__FORMAT__, ...) NSLog((@"%s [Line %d] " __FORMAT__), __PRETTY_FUNCTION__, __LINE__, ##__VA_ARGS__)
#else
#define DLog(...) do {} while (0)
#endif

// ALog always displays output regardless of the DEBUG setting
#define ALog(__FORMAT__, ...) NSLog((@"%s [Line %d] " __FORMAT__), __PRETTY_FUNCTION__, __LINE__, ##__VA_ARGS__)

#define LogError(error) if((error)) ALog(@"%@", (error));
```

*     Treat warnings as errors.

*     *Analyze* your code frequently, at least before creating an archive/release.

*    Use Image catalogs. This is important to keep images organised now that there are a gazillion resolutions of iOS devices we need to care about.

*    Use [GitFlow](http://nvie.com/posts/a-successful-git-branching-model/). It would make your life easier. In short, all development should occur on *develop* and feature branches. *master* branch should only have releases. Server should only have *master* and *develop* branches unless otherwise required. Don't push feature or release branches to the server.

*     [SourceTree](http://www.sourcetreeapp.com/) is a good git/hg GUI client. I like command line as much as any developer, but there are things which are better suited for a GUI applications, managing repositories is one of them. SourceTree has a [handy button to use GitFlow](http://blog.sourcetreeapp.com/2012/08/01/smart-branching-with-sourcetree-and-git-flow/).

*    Use [semantic versioning](http://semver.org/).

*     I am ambivalent about CoreData. On one hand it makes sense to use it since more developers are becoming familiar with it. Many 3rd party libraries support CoreData out of the box e.g. [RestKit](http://restkit.org/). Eventually it would make maintenance easier. On the other hand however, It's a pain to implement. Need an attribute to be unique? No can do (unless you write code to ensure uniqueness). Need to delete all records/entities in a table in one go? Sorry. CoreData is an over-hyped, under-cooked framework which causes more pain than it is worth. The reason it has become so popular is the same reason objective-c became popular; Apple shoved it down our throats. So IMO if you are competent enough to figure out how to use CoreData properly, you'll be able to write your own code for SQLite with much less effort. Like I said, I am ambivalent when it comes to CoreData, use your own judgement.

* Comment your class interfaces properly. It helps you create re-usable components. Make sure you have [VVDocumenter-Xcode](https://github.com/onevcat/VVDocumenter-Xcode) installed, it helps you write javadoc style comments.


### Useful Pods ###

If someone has already built something you need, you should use it instead of reinventing the wheel. Always check [CocoaPods](http://cocoapods.org/) and [CocoaControls](https://www.cocoacontrols.com/) before implementing something from scratch. Chances are, someone already solved the problem face.

The following is a list of most useful pods I have used.

#### TestFlightSDK ####

TestFlightSDK is a must have for any iOS project. It provides over-the-air beta testing and crash reporting.

##### Homepage: #####
[http://www.testflightapp.com/](http://www.testflightapp.com/)
##### CocoaPods: #####
[http://cocoapods.org/?q=testflightsdk](http://cocoapods.org/?q=testflightsdk)

#### MRProgress ####

Inspired from [MBProgressHUD](https://github.com/jdg/MBProgressHUD), MRProgress is a modern looking ProgressHUD/activity-indicator/loading-wheel that follows iOS7 design.

##### GitHub: #####
[https://github.com/mrackwitz/MRProgress](https://github.com/mrackwitz/MRProgress)

#### AFNetworking ####

The de-facto network library for iOS.

##### GitHub: #####
[https://github.com/AFNetworking/AFNetworking](https://github.com/AFNetworking/AFNetworking)

#### MEAlertView and MEActionSheet ####

Simple block based *UIAlertView* and *UIActionSheet* replacements.

##### MEAlertView: #####
[https://github.com/enriquez/MEAlertView](https://github.com/enriquez/MEAlertView)
##### MEActionSheet: #####
[https://github.com/enriquez/MEActionSheet](https://github.com/enriquez/MEActionSheet)

#### RBStoryboardLink ####

Storyboards are a powerful tool. A good practice is to break storyboards so that they encapsulates different sub-flows inside the main application e.g. *Login and Signup* flow, *Edit Profile* flow, etc. However, apple doesn't provide an elegant way to create links between storyboards. That's where RBStoryboardLink comes in. Also, Rob Brown's blog post on [storyboards best practices](http://robsprogramknowledge.blogspot.com/2012/01/uistoryboard-best-practices.html) is good read.

##### GitHub: #####
[https://github.com/rob-brown/RBStoryboardLink](https://github.com/rob-brown/RBStoryboardLink)

#### Lockbox ####

An easy to use wrapper for key chain. Very handy when you need to store data securely e.g. user's access token, etc.

##### GitHub: #####
[https://github.com/granoff/Lockbox](https://github.com/granoff/Lockbox)

#### FXForms ####

Probably the best table based forms library on the iOS. Its strength is the amount of things it can guess from your existing models without any extra code. Even when you have to customize the forms, the amount of code is minimal.

##### GitHub: #####
[https://github.com/nicklockwood/FXForms](https://github.com/nicklockwood/FXForms)


#### DateTools ####

If your project does heavy date/time and time period calculations, this library can make your life easier.

##### GitHub: #####
[https://github.com/MatthewYork/DateTools](https://github.com/MatthewYork/DateTools)

#### RHManagedObject ####

This is the CoreData API that apple should have provided. If you are using CoreData, Using RHManagedObject can save you from tons of biolerplate code.

##### GitHub:#####
[https://github.com/chriscdn/RHManagedObject](https://github.com/chriscdn/RHManagedObject)


#### INTULocationManager ####

For one-off location updates i.e. when you just need to find out where user currently is, or need to auto-fill an address.

##### GitHub: #####
[https://github.com/intuit/LocationManager](https://github.com/intuit/LocationManager)

#### IDMPhotoBrowser ####

For when you need to display a bunch of photos.

##### GitHub: #####
[https://github.com/ideaismobile/IDMPhotoBrowser](https://github.com/ideaismobile/IDMPhotoBrowser)
