# Adding Firebase to iOS

## Prerequisites

Ensure the following are installed or set up:

- [Firebase Setup Documentation](https://firebase.google.com/docs/ios/setup?authuser=0&hl=en)
- **Xcode 15.2 or later**

Make sure your project meets these platform version requirements:

- iOS 13 or later  
- macOS 10.15 or later  
- tvOS 13 or later  
- watchOS 7 or later  

---

## Step 1: Create a Firebase Project and Register Your App

1. Go to the [Firebase Console](https://console.firebase.google.com/).
2. Create a new project.
3. Navigate to **Project Settings** (gear icon) > **Your Apps**.
4. Select **iOS**, then add your app by providing the **Bundle ID**.

---

## Step 2: Download Firebase Config File

1. In **Project Settings** in the Firebase Console:
   - Select the app's **Bundle ID** under "Your Apps."
   - Download the `GoogleService-Info.plist` file.
2. Move the config file into the **root directory** of your Xcode project.
3. Ensure the config file is added to **all targets** during the setup.
4. Confirm that only the latest config file is in your project.

---

## Step 3: Add Firebase SDK

1. In Xcode, open your project and navigate to **File > Add Packages**.
2. Enter the Firebase iOS SDK repository URL:  
   ```text
   https://github.com/firebase/firebase-ios-sdk

## Step 4: Add Initialization Code

// SwiftUI

```swift
import SwiftUI
import FirebaseCore

class AppDelegate: NSObject, UIApplicationDelegate {
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]? = nil) -> Bool {
        FirebaseApp.configure()
        return true
    }
}

@main
struct YourApp: App {
    @UIApplicationDelegateAdaptor(AppDelegate.self) var delegate

    var body: some Scene {
        WindowGroup {
            NavigationView {
                ContentView()
            }
        }
    }
}

// UIKit
import UIKit
import FirebaseCore

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?

    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        FirebaseApp.configure()
        return true
    }
}

#### Once you have the dsym in the project and have configured firebase add a script

* Click the Build Phases tab, then complete the following steps so that Xcode can process your dSYMs and upload the files.
* Click add > New Run Script Phase.
* Make sure this new Run Script phase is your project's last build phase; otherwise, Crashlytics can't properly process dSYMs.
* Expand the new Run Script section.
` "${BUILD_DIR%/Build/*}/SourcePackages/checkouts/firebase-ios-sdk/Crashlytics/run"
* In the Input Files section, add the paths for the locations of the following files:
`${DWARF_DSYM_FOLDER_PATH}/${DWARF_DSYM_FILE_NAME}
` ${DWARF_DSYM_FOLDER_PATH}/${DWARF_DSYM_FILE_NAME}/Contents/Resources/DWARF/${PRODUCT_NAME}
`${DWARF_DSYM_FOLDER_PATH}/${DWARF_DSYM_FILE_NAME}/Contents/Info.plist
`$(TARGET_BUILD_DIR)/$(UNLOCALIZED_RESOURCES_FOLDER_PATH)/GoogleService-Info.plist
`$(TARGET_BUILD_DIR)/$(EXECUTABLE_PATH)
* If you have ENABLE_USER_SCRIPT_SANDBOXING=YES and ENABLE_DEBUG_DYLIB=YES in your project build settings, then include the following:
`${DWARF_DSYM_FOLDER_PATH}/${DWARF_DSYM_FILE_NAME}/Contents/Resources/DWARF/${PRODUCT_NAME}.debug.dylib
