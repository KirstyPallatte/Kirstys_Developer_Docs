## Adding Firebase to IOS

### Prerequisites
Install the following:

Xcode 15.2 or later
Make sure that your project meets these requirements:

Your project must target these platform versions or later:
iOS 13
macOS 10.15
tvOS 13
watchOS 7

### Create firebase project in firebase console
* Navigate to firebase and add new project

### Firebase config files and objects
When you register an app with a Firebase project, the Firebase console provides a Firebase configuration file (Apple/Android apps) or a configuration object (web apps) that you add directly to your local app directory.

### Download Firebase config file or object
* Go to your the Settings icon Project settings in the Firebase console.
* In the Your apps card, select the bundle ID of the app for which you need a config file.
* Click  GoogleService-Info.plist.
* Move your config file into the root of your Xcode project. If prompted, select to add the config file to all targets.
* Make sure that you only have this most recent downloaded config file in your Xcode project
